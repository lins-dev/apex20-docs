# Detalhamento Técnico: Sprint M — Migração Next.js → TanStack ⚔️🔄

**Objetivo:** Remover completamente o Next.js do `apex20-web` e substituir por React + TanStack (Router + Start + Query), mantendo 100% da funcionalidade existente e todos os testes passando.
**Status Atual:** 🔴 Não Iniciado
**Prioridade:** 🔥 Máxima — todas as demais sprints estão suspensas até conclusão.
**Branch:** `refactor/nextjs-to-tanstack` (criada a partir de `dev`)
**ADR:** [ADR-038](../adrs/ADR-038-nextjs-to-tanstack-migration.md)

---

## Critério de Aceite (Definition of Done)

Antes de abrir o PR para `dev`, **todos** os itens abaixo devem estar verdes:

- [ ] Zero imports de `next/*` em qualquer `.ts`/`.tsx`
- [ ] `npm run typecheck` sem erros
- [ ] `npm run lint` sem warnings
- [ ] `npm run test` — todos os 50+ testes Vitest passando
- [ ] `npm run test:visual` — todos os testes Playwright passando
- [ ] `npm run build` sem erros
- [ ] Rota `/` — landing page renderizando com i18n correto
- [ ] Rota `/login` e `/signup` — formulários funcionando
- [ ] Rota `/design-system` — página carregando
- [ ] Auth guard — redirecionando rotas protegidas sem token
- [ ] i18n — detecção de locale via cookie e Accept-Language funcionando
- [ ] ConnectRPC proxy `/connect/*` → backend funcionando

---

## Fase 1 — Setup da Nova Stack

### 1.1 Remover Next.js e dependências

- [ ] Remover `next` do `package.json` (dependencies)
- [ ] Remover `@types/node` se exclusivo do Next.js (verificar se ainda é necessário)
- [ ] Remover scripts do `package.json` que referenciam `next` (`dev`, `build`, `start`)
- [ ] Remover `next.config.ts`
- [ ] Remover `postcss.config.mjs` (Tailwind v4 tem integração direta com Vite, sem PostCSS)
- [ ] Remover `.next/` do diretório (já está no `.gitignore`, mas verificar)
- [ ] Remover `next-env.d.ts` se existir

### 1.2 Instalar TanStack Start e dependências

- [ ] Instalar `@tanstack/start` (meta-framework)
- [ ] Instalar `@tanstack/react-router` (router)
- [ ] Instalar `@tanstack/react-query` (server state)
- [ ] Instalar `@tanstack/react-query-devtools` (devDependency)
- [ ] Instalar `vinxi` (bundler base do TanStack Start)
- [ ] Instalar `fontsource` equivalentes para Inter e JetBrains Mono:
  - `@fontsource-variable/inter`
  - `@fontsource-variable/jetbrains-mono`

### 1.3 Criar `app.config.ts` (substitui `next.config.ts`)

- [ ] Criar `app.config.ts` na raiz com configuração do TanStack Start
- [ ] Configurar `server.proxy` para redirecionar `/connect/*` → `VITE_API_URL` (substitui o `rewrites` do `next.config.ts`)
- [ ] Configurar `vite.resolve.alias` para manter os path aliases `@/*` e `@contracts/*`
- [ ] Configurar `tsr` (TanStack Router) com `routesDirectory: 'src/routes'` e `generatedRouteTree: 'src/routeTree.gen.ts'`

### 1.4 Atualizar `tsconfig.json`

- [ ] Remover referências a `next` nos types (`"types": ["next"]`)
- [ ] Manter `"moduleResolution": "bundler"` (compatível com Vite)
- [ ] Manter `"jsx": "react-jsx"`
- [ ] Verificar e manter path aliases `@/*` e `@contracts/*`
- [ ] Adicionar `src/routeTree.gen.ts` ao exclude se necessário

### 1.5 Atualizar `package.json` scripts

- [ ] `"dev": "vinxi dev"` (substitui `next dev`)
- [ ] `"build": "vinxi build"` (substitui `next build`)
- [ ] `"start": "vinxi start"` (substitui `next start`)
- [ ] Manter `"test"`, `"test:watch"`, `"test:coverage"`, `"storybook"`, `"test:visual"` sem mudança

### 1.6 Atualizar variáveis de ambiente

- [ ] Renomear `NEXT_PUBLIC_API_URL` → `VITE_API_URL` no `.env.example`
- [ ] Renomear `NEXT_PUBLIC_WS_URL` → `VITE_WS_URL` no `.env.example`
- [ ] Atualizar referências em `src/lib/api/transport.ts` (`process.env.NEXT_PUBLIC_API_URL` → `import.meta.env.VITE_API_URL`)
- [ ] Atualizar referências em qualquer outro arquivo que use `process.env.NEXT_PUBLIC_*`
- [ ] Remover `INTERNAL_API_URL` (era usada apenas no `next.config.ts` rewrites — agora o proxy fica no `app.config.ts`)

---

## Fase 2 — Estrutura de Rotas

### 2.1 Criar o Router

- [ ] Criar `src/router.ts` — instância do `createRouter()` com a routeTree gerada
- [ ] Criar `src/entry-client.tsx` — entry point do cliente com `RouterProvider` e `QueryClientProvider`
- [ ] Criar `src/entry-server.tsx` — entry point do servidor (SSR) com `StartServer`

### 2.2 Criar estrutura `src/routes/`

- [ ] Criar `src/routes/__root.tsx` — Root Route (substitui `src/app/layout.tsx`)
  - Importar e aplicar fontes via `@fontsource-variable/inter` e `@fontsource-variable/jetbrains-mono` (substitui `next/font/google`)
  - Manter o anti-flash script para tema dark/light (era `dangerouslySetInnerHTML` no layout do Next.js)
  - Manter `<html>`, `<body>`, e o `<Outlet />` do TanStack Router
  - Manter meta tags globais via `createRootRoute({ head: () => ({ meta: [...] }) })`

- [ ] Criar `src/routes/index.tsx` — Homepage (substitui `src/app/page.tsx`)
  - Importar e renderizar `<LandingPage locale={locale} />`
  - Locale via `beforeLoad` (lê cookie `apex20-locale`)

- [ ] Criar `src/routes/login.tsx` — Login (substitui `src/app/login/page.tsx`)
  - Importar e renderizar `<AuthLayout>` + `<SignInForm>`
  - beforeLoad: redirecionar para `/` se já autenticado (substitui o redirect do middleware)

- [ ] Criar `src/routes/signup.tsx` — Signup (substitui `src/app/signup/page.tsx`)
  - Importar e renderizar `<AuthLayout>` + `<SignUpForm>`
  - beforeLoad: redirecionar para `/` se já autenticado

- [ ] Criar `src/routes/design-system.tsx` — Design system (substitui `src/app/design-system/page.tsx`)

### 2.3 Remover estrutura `src/app/`

- [ ] Remover `src/app/layout.tsx`
- [ ] Remover `src/app/page.tsx`
- [ ] Remover `src/app/login/page.tsx`
- [ ] Remover `src/app/signup/page.tsx`
- [ ] Remover `src/app/design-system/page.tsx`
- [ ] Remover `src/app/globals.css` → mover conteúdo para `src/styles/globals.css`

---

## Fase 3 — Reescrever Middleware (i18n + Auth Guard)

### 3.1 Contexto

O `src/middleware.ts` do Next.js fazia duas coisas:
1. **i18n:** Lia cookie `apex20-locale` ou `Accept-Language`, definia header `x-locale`
2. **Auth guard:** Verificava JWT no cookie `apex20-token`, redirecionava rotas protegidas

No TanStack Router, essa lógica vai para `beforeLoad` nas rotas ou no `__root.tsx`.

### 3.2 Tarefas

- [ ] Criar `src/lib/locale.ts` — helper para ler locale do cookie no cliente (substitui a detecção do middleware)
  - `getLocaleFromCookie(): Locale` — lê cookie `apex20-locale`, fallback para `detectLocale(navigator.language)`
  - Exportar para uso nos `beforeLoad` das rotas

- [ ] Criar `src/lib/auth-guard.ts` — helper para verificar autenticação nas rotas
  - `requireAuth(context)` — lança `redirect({ to: '/login' })` se sem token
  - `requireGuest(context)` — lança `redirect({ to: '/' })` se já autenticado

- [ ] Atualizar `src/routes/index.tsx` — adicionar `beforeLoad` para detectar e passar locale
- [ ] Atualizar `src/routes/login.tsx` — adicionar `beforeLoad` com `requireGuest()`
- [ ] Atualizar `src/routes/signup.tsx` — adicionar `beforeLoad` com `requireGuest()`
- [ ] Criar `src/routes/_protected.tsx` — layout de rotas protegidas (futuras: `/dashboard`, `/campaigns`, `/vtt`)
  - `beforeLoad` com `requireAuth()`

- [ ] Remover `src/middleware.ts`

---

## Fase 4 — Substituir Imports do Next.js

### 4.1 `next/link` → `@tanstack/react-router`

Arquivos afetados (7):
- [ ] `src/modules/landing/components/navbar.tsx`
- [ ] `src/modules/auth/components/sign-in-form.tsx`
- [ ] `src/modules/auth/components/sign-up-form.tsx`
- [ ] Verificar demais arquivos com `grep -r "from 'next/link'" src/`

Mudança:
```ts
// Antes
import Link from "next/link";
<Link href="/login">Login</Link>

// Depois
import { Link } from "@tanstack/react-router";
<Link to="/login">Login</Link>
```

### 4.2 `next/navigation` → `@tanstack/react-router`

Arquivos afetados (2):
- [ ] `src/modules/auth/components/sign-in-form.tsx`
- [ ] `src/modules/auth/components/sign-up-form.tsx`

Mudança:
```ts
// Antes
import { useRouter } from "next/navigation";
const router = useRouter();
router.push("/");

// Depois
import { useNavigate } from "@tanstack/react-router";
const navigate = useNavigate();
navigate({ to: "/" });
```

### 4.3 `next/font` → fontsource

Arquivo afetado: `src/routes/__root.tsx` (novo arquivo, substitui `src/app/layout.tsx`)

Mudança:
```ts
// Antes (next/font/google em layout.tsx)
import { Inter, JetBrains_Mono } from "next/font/google";
const inter = Inter({ variable: "--font-sans", subsets: ["latin"] });

// Depois (fontsource importado uma vez no entry point ou __root.tsx)
import "@fontsource-variable/inter";
import "@fontsource-variable/jetbrains-mono";
// CSS variables definidas manualmente em globals.css:
// --font-sans: 'Inter Variable', sans-serif;
// --font-mono: 'JetBrains Mono Variable', monospace;
```

### 4.4 `next/server` → remover

- [ ] Remover `src/middleware.ts` (já coberto na Fase 3)
- [ ] Verificar se há outros imports de `next/server` com `grep -r "from 'next/server'" src/`

### 4.5 Verificação final de imports Next.js

- [ ] Executar `grep -r "from 'next" src/` — deve retornar zero resultados
- [ ] Executar `grep -r "from \"next" src/` — deve retornar zero resultados

---

## Fase 5 — Metadata e SEO

### 5.1 Substituir `generateMetadata()`

O `src/app/layout.tsx` usava `generateMetadata()` para injetar title e description com i18n.

- [ ] Em `src/routes/__root.tsx`, usar a API de `head` do TanStack Router:
  ```ts
  export const Route = createRootRoute({
    head: () => ({
      meta: [
        { title: t("common.meta.title", locale) },
        { name: "description", content: t("common.meta.description", locale) },
      ],
    }),
  });
  ```
- [ ] Garantir que o locale seja acessível no `head()` via search params ou context

---

## Fase 6 — Atualizar Testes

### 6.1 Mock de `next/link`

Existe `src/__mocks__/next-link.tsx` — esse mock não é mais necessário.

- [ ] Remover `src/__mocks__/next-link.tsx`
- [ ] Atualizar `vitest.config.ts` se houver alias de mock para `next/link`
- [ ] Verificar se algum teste usa `vi.mock('next/link')` → substituir pelo componente real do TanStack Router ou mock equivalente

### 6.2 Atualizar testes que usam `useRouter` do Next.js

- [ ] `src/modules/auth/components/sign-in-form.test.tsx` — mock de `useRouter` provavelmente referencia `next/navigation`
- [ ] `src/modules/auth/components/sign-up-form.test.tsx` — idem
- [ ] Substituir mocks de `next/navigation` por mocks do `@tanstack/react-router` (RouterProvider de teste)

### 6.3 Atualizar `vitest.config.ts`

- [ ] Remover qualquer alias que resolvia pacotes `next/*`
- [ ] Verificar se o `environment: 'jsdom'` continua adequado
- [ ] Manter `setupFiles` com `src/ui/test/setup.ts`

### 6.4 Executar suíte completa e corrigir

- [ ] `npm run test` — resolver todos os erros um a um
- [ ] `npm run typecheck` — resolver todos os erros TypeScript
- [ ] `npm run lint` — resolver todos os warnings

---

## Fase 7 — Testes Visuais (Playwright/Storybook)

- [ ] Verificar se as Stories do Storybook continuam funcionando (`npm run storybook`)
- [ ] As stories são componentes React puros — devem funcionar sem mudança
- [ ] Atualizar snapshots Playwright se o HTML estrutural mudar (ex: `<a>` vs `<Link>` renderiza diferente?)
- [ ] `npm run test:visual` — verificar e atualizar snapshots necessários com `--update-snapshots`

---

## Fase 8 — Build e Validação Final

- [ ] `npm run build` — build de produção sem erros
- [ ] Testar o servidor em produção localmente: `npm run start`
- [ ] Verificar rota `/` — landing page com i18n
- [ ] Verificar rota `/login` — form de login, redirect se autenticado
- [ ] Verificar rota `/signup` — form de cadastro, redirect se autenticado
- [ ] Verificar rota `/design-system` — página carrega
- [ ] Verificar proxy `/connect/*` → backend (via `app.config.ts` server.proxy)
- [ ] Verificar dark/light mode (anti-flash script no `__root.tsx`)
- [ ] Verificar language switcher — troca de locale persiste no cookie

---

## Fase 9 — Limpeza e PR

- [ ] Executar checklist completo de critério de aceite (ver topo do documento)
- [ ] Remover arquivos órfãos (src/app/, middleware.ts, next.config.ts, postcss.config.mjs)
- [ ] Atualizar `README.md` do `apex20-web` com os novos comandos (vinxi dev/build/start)
- [ ] Atualizar `.env.example` com `VITE_*` em vez de `NEXT_PUBLIC_*`
- [ ] Abrir PR `refactor/nextjs-to-tanstack` → `dev`
- [ ] CI verde obrigatório antes do merge

---

**Estimativa de esforço por fase:**

| Fase | Descrição | Complexidade |
|---|---|---|
| 1 | Setup da nova stack | Média |
| 2 | Estrutura de rotas | Alta |
| 3 | Middleware → beforeLoad | Alta |
| 4 | Substituir imports | Baixa |
| 5 | Metadata/SEO | Baixa |
| 6 | Atualizar testes | Média |
| 7 | Testes visuais | Baixa |
| 8 | Build e validação | Baixa |
| 9 | Limpeza e PR | Baixa |
