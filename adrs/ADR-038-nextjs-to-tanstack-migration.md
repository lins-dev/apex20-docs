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
| `next/font` | `@fontsource-variable/*` referenciados em `head().links` via `?url` |
| `middleware.ts` (i18n + auth guard) | `beforeLoad` hooks nas rotas do TanStack Router |
| `generateMetadata()` | `head()` no objeto de configuração da rota |
| `next.config.ts` rewrites (proxy) | `app.config.ts` → Nitro `routeRules` |
| `NEXT_PUBLIC_*` env vars | `VITE_*` env vars (Vite padrão) |
| Server Components (RSC) | SSR Loaders + Client Components |
| `headers()` server API | Context de request nos Loaders |
| `router.refresh()` | `router.invalidate()` |
| `next start` | `vinxi start` (TanStack Start) |
| `next build` | `vinxi build` (TanStack Start) |

### Padrões de implementação TanStack Start v1

#### Entry points

O TanStack Start expõe dois entry points explícitos (diferente do Next.js onde são transparentes):

```
src/client.tsx    → hidrataçãoo no cliente (hydrateRoot)
src/ssr.tsx       → handler SSR do servidor (createStartHandler)
```

O arquivo `src/router.ts` exporta a função `getRouter()` — uma factory que cria uma **nova instância** por request (crítico para SSR sem vazamento de estado entre requests).

#### Estrutura do `__root.tsx`

O `__root.tsx` usa `createRootRouteWithContext<RouterContext>()` para injetar o contexto global (auth, etc.) com type-safety em toda a árvore de rotas. CSS global é importado com o sufixo `?url` e declarado em `head().links` — **nunca como import direto no componente** (o Vite trata diferente; import direto não gera `<link>` no HTML SSR):

```tsx
import globalsCss from "@/styles/globals.css?url";

export const Route = createRootRouteWithContext<RouterContext>()({
  head: () => ({
    links: [{ rel: "stylesheet", href: globalsCss }],
  }),
  // ...
});
```

Scripts inline que devem executar **antes** da hidratação do React (ex: detecção de tema) usam `<ScriptOnce>` de `@tanstack/react-router` — **nunca `dangerouslySetInnerHTML` no JSX**:

```tsx
import { ScriptOnce } from "@tanstack/react-router";

// A função é serializada para string — não é uma string literal
const themeScript = `(function(){var t=localStorage.getItem('apex20-theme');if(t)document.documentElement.setAttribute('data-theme',t);})();`;

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    <html suppressHydrationWarning>
      <head><HeadContent /></head>
      <body>
        <ScriptOnce children={themeScript} />
        {children}
        <Scripts />
      </body>
    </html>
  );
}
```

`HeadContent` vem de `@tanstack/react-router` (não de `@tanstack/start`). Renderiza tudo declarado nos objetos `head()` ao longo da árvore de rotas.

#### Autenticação via RouterContext (Padrão 2)

O estado de auth é um **Zustand store global** (não `useState` local). O store é injetado no `RouterProvider` via `context`, tornando-o disponível com type-safety em qualquer `beforeLoad` via `context.auth`:

```tsx
// src/router.ts
interface RouterContext {
  auth: AuthState; // do Zustand store
}

export function getRouter() {
  return createRouter({
    routeTree,
    context: { auth: undefined! },
  });
}

// src/client.tsx
function App() {
  const auth = useAuth(); // Zustand store — global e reativo
  return <RouterProvider router={router} context={{ auth }} />;
}

// src/routes/_protected.tsx
export const Route = createFileRoute("/_protected")({
  beforeLoad: ({ context }) => {
    if (!context.auth.isAuthenticated) {
      throw redirect({ to: "/login" });
    }
  },
});
```

**Por que Zustand e não `useState`?** `useState` cria estado local por componente — se `SignInForm` chama `useAuth().signIn(token)` e o `App` também chama `useAuth()`, são instâncias separadas. Com Zustand, todos os consumidores de `useAuth()` compartilham o mesmo estado global. Quando `signIn` é chamado em qualquer componente, o `App` (que passa `auth` para o `RouterProvider`) re-renderiza com o novo estado, e a próxima navegação receberá `context.auth.isAuthenticated === true`.

#### Proxy ConnectRPC

O proxy para o backend é configurado via Nitro `routeRules` no `app.config.ts`, não via `vite.server.proxy`. O `vite.server.proxy` funciona apenas no servidor de desenvolvimento Vite; `routeRules` funciona tanto em dev quanto em produção (`vinxi start`):

```ts
// app.config.ts
import { defineConfig } from "@tanstack/start/config";

export default defineConfig({
  server: {
    routeRules: {
      "/connect/**": {
        proxy: { to: `${process.env.VITE_API_URL ?? "http://localhost:8787"}/**` },
      },
    },
  },
});
```

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
