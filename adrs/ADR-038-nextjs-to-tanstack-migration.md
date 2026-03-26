# ADR-038: Migração do Next.js para React + TanStack 🔄

**Data:** 2026-03-25
**Status:** Aceito

## Contexto

Durante a Sprint 2, com o projeto ainda em fase inicial e a base de código enxuta, foi realizada uma análise estratégica aprofundada sobre a viabilidade de hospedar o `apex20-web` em VPS (Hostgator) em vez da Vercel, e avaliadas as alternativas ao Next.js disponíveis no ecossistema React.

### Problemas identificados com Next.js em self-hosting (VPS)

1. **`NEXT_PUBLIC_*` inlined no build:** Variáveis públicas são hardcoded no bundle em tempo de build. Staging e produção exigem builds separados — inviabiliza o modelo "build once, deploy anywhere".
2. **Image Optimization (sharp) em Linux:** Uso excessivo de memória com glibc sem configuração adicional de memory allocator (jemalloc/tcmalloc).
3. **Cache multi-instância:** ISR e o Router Cache exigem um custom cache handler (Redis) para sincronizar múltiplas instâncias. Sem isso, usuários recebem dados stale de pods diferentes.
4. **Custom server vs. `output: 'standalone'`:** São mutuamente exclusivos. WebSocket integrado ao servidor Next.js (necessário em versões futuras) seria incompatível com o modo portátil para Docker.
5. **RAM em produção:** App Router em idle consome 300–500 MB; sob carga pode ultrapassar 1 GB em VPS entry-level.
6. **Nginx:** Requer `X-Accel-Buffering: no` explícito para não quebrar o streaming SSR.
7. **Breaking changes de cache (Next.js 15):** GET Route Handlers e pages deixaram de ser cacheados por padrão — migrações sem atenção causam aumento de carga.
8. **Middleware Edge Runtime:** APIs Node.js completas no Middleware são experimentais (`next@canary` apenas), impedindo uso de bibliotecas nativas na camada de edge.

### Alternativas avaliadas

#### viNext (vinext.io)
Projeto da Cloudflare (Steve Faulkner) que reimplementa a API do Next.js sobre Vite. Lançado em fevereiro de 2026, construído com IA em menos de uma semana.
- **Versão:** 0.1.0 — declarado explicitamente como experimental e não battle-tested.
- **Positivos:** ~94% da API do Next.js coberta, builds 4.4x mais rápidos, bundle 57% menor, integração nativa com R2/KV/Durable Objects.
- **Negativo crítico:** Menos de 2 meses de vida. Sem battle-test em produção real em escala. Risco de abandono ou breaking changes severos. Código gerado por IA sem revisão humana profunda. `generateStaticParams()` não implementado.
- **Decisão:** Descartado por imaturidade. Reavaliar quando atingir v1.0 com 6+ meses de produção real.

#### React + TanStack Start
Meta-framework full-stack construído sobre TanStack Router + Vinxi/Vite.
- **Paradigma:** SSR + Loaders + Server Functions (sem RSC). Similar ao Remix, mas com type safety radical end-to-end.
- **Maturidade:** TanStack Router v1 estável (~2024), TanStack Start v1 estável (~2025). Ecossistema ativo e crescente.
- **Self-hosting:** Output Node.js padrão sem complexidade de `output: 'standalone'`. Variáveis de ambiente são runtime por padrão. Vite como base — builds rápidos, DX superior.
- **Type safety:** Routes, URL params, search params e loaders são 100% tipados end-to-end — superior ao Next.js.
- **Para o VTT:** O core do Apex20 (grid, WebSocket, canvas, combat tracker) é 100% client-heavy — RSC não agrega valor real. A ausência de RSC no TanStack Start não é uma limitação prática para este produto.

## Decisão

**Migrar o `apex20-web` de Next.js 16 (App Router) para React 19 + TanStack (Router + Start + Query).**

Esta migração é executada enquanto o projeto é pequeno e o custo de refatoração é mínimo. Postergar a decisão aumentaria o custo exponencialmente à medida que mais features Next.js-específicas fossem adicionadas.

### Stack frontend resultante

```
React 19                     → View layer (sem mudança)
TanStack Router v1           → Roteamento file-based, 100% type-safe
TanStack Start v1            → SSR, Loaders, Server Functions, bundling
TanStack Query v5            → Server state, cache, data fetching
Vite / Vinxi                 → Bundler base (substitui webpack/turbopack do Next.js)
Zustand v5                   → Estado global client-side (sem mudança)
XState v5                    → Máquinas de estado (sem mudança)
```

### Mapeamento de substituições

| Next.js (removido) | TanStack / Equivalente (adotado) |
|---|---|
| `next` package | `@tanstack/start`, `@tanstack/react-router` |
| App Router (`src/app/`) | TanStack Router (`src/routes/`) |
| `next/link` | `<Link>` do `@tanstack/react-router` |
| `next/navigation` (`useRouter`) | `useNavigate` do `@tanstack/react-router` |
| `next/font` | `fontsource` + CSS variables |
| `middleware.ts` (i18n + auth guard) | `beforeLoad` hooks nas rotas do TanStack Router |
| `generateMetadata()` | Meta options do TanStack Router |
| `next.config.ts` rewrites (proxy) | `vite.config.ts` → `server.proxy` |
| `NEXT_PUBLIC_*` env vars | `VITE_*` env vars (Vite padrão) |
| Server Components (RSC) | SSR Loaders + Client Components |
| `headers()` server API | Context de request nos Loaders |
| `next start` | `vinxi start` (TanStack Start) |
| `next build` | `vinxi build` (TanStack Start) |

### Critério de aceite da migração

A branch `refactor/nextjs-to-tanstack` só pode ser mergeada para `dev` quando:
1. Zero imports de `next/*` em qualquer arquivo `.ts`/`.tsx`.
2. Todos os testes unitários passando (50+ testes Vitest).
3. Todos os testes visuais passando (Playwright).
4. `npm run build` executando sem erros.
5. `npm run typecheck` sem erros TypeScript.
6. Rota `/`, `/login` e `/signup` funcionando com i18n e auth guard.
7. ConnectRPC client funcionando (proxy `/connect/*` → backend).

## Justificativa

- **Momento certo:** O projeto tem baixa dependência real do Next.js (sem ISR, sem Server Actions, sem Route Handlers). Migrar agora custa uma sprint; migrar após a Sprint 4 custaria meses.
- **Self-hosting:** TanStack Start gera um servidor Node.js standard — sem as complexidades de cache, sharp, e NEXT_PUBLIC_* inlining que tornam o self-hosting em VPS um exercício de gerenciamento de infraestrutura.
- **Coerência de ecossistema:** O Apex20 já usa Zustand (Daishi Kato) e adotará TanStack Query. TanStack Router + Start completam um ecossistema coeso com a mesma filosofia: explícito, type-safe, sem magia.
- **DX:** Vite como base de build é mais rápido e integra nativamente com o Storybook já configurado (`@storybook/react-vite`). Elimina a dualidade webpack/turbopack do Next.js.
- **Sem RSC é aceitável:** O VTT é fundamentalmente uma aplicação client-heavy (canvas, WebSocket, real-time). RSC beneficia principalmente páginas de marketing com muito conteúdo estático — não é o core do produto.

## Consequências

**Positivas:**
- Self-hosting trivial em qualquer VPS ou Cloudflare Workers.
- Variáveis de ambiente runtime por padrão (sem rebuild por ambiente).
- Build mais rápido (Vite nativo).
- Type safety end-to-end nas rotas.
- WebSocket server integrável sem conflito com o servidor.
- Redução de dependências de infraestrutura (sem Redis para ISR, sem sharp).

**Negativas:**
- Sprint dedicada de refatoração (custo imediato).
- Ecossistema menor que Next.js — menos exemplos/plugins disponíveis.
- Sem RSC — bundle da landing page ligeiramente maior (mitigado pelo tree-shaking do Vite).
- TanStack Start mais novo que Next.js — menos soluções documentadas para edge cases.

## Referências

- **ADR-001:** Escolha da Stack Tecnológica (atualizado).
- **ADR-036:** Arquitetura Frontend Modular (atualizado).
- **ADR-025:** Gestão de Estados de Jogo (Zustand/XState — sem mudança).
- **ADR-010:** Estratégia de Testes (critério de aceite da migração).
- **ADR-037:** Estratégia de Branching (branch `refactor/nextjs-to-tanstack`).
- **Pesquisa:** [nextjs.org/docs/app/guides/self-hosting](https://nextjs.org/docs/app/guides/self-hosting)
- **Pesquisa:** [github.com/cloudflare/vinext](https://github.com/cloudflare/vinext)
- **Pesquisa:** [tanstack.com/start/latest](https://tanstack.com/start/latest)
