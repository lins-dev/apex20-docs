# Detalhamento Tecnico: Sprint M - Migracao Next.js -> TanStack

**Objetivo:** Remover completamente o Next.js do `apex20-web` e substituir por React + TanStack (Router + Start + Query), mantendo 100% da funcionalidade existente e todos os testes passando.
**Status Atual:** Concluido
**Prioridade:** ---
**Branch:** `refactor/nextjs-to-tanstack` (Finalizada e Merged)
**ADR:** [ADR-038](../adrs/ADR-038-nextjs-to-tanstack-migration.md)

---

## Criterio de Aceite (Definition of Done)

Antes de abrir o PR para `dev`, todos os itens abaixo devem estar verdes:

- [x] Zero imports de `next/*` em qualquer `.ts`/`.tsx`
- [x] `npm run typecheck` sem erros
- [x] `npm run lint` sem warnings
- [x] `npm run test` - todos os 50+ testes Vitest passando
- [x] `npm run test:visual` - todos os testes Playwright passando
- [x] `npm run build` sem erros
- [x] Rota `/` - landing page renderizando com i18n correto
- [x] Rota `/login` e `/signup` - formularios funcionando
- [x] Rota `/design-system` - pagina carregando
- [x] Auth guard - redirecionando rotas protegidas sem token
- [x] i18n - deteccao de locale via cookie e Accept-Language funcionando
- [x] ConnectRPC proxy `/connect/*` -> backend funcionando

---

## Fase 1 - Setup da Nova Stack

### 1.1 Remover Next.js e dependencias

- [x] Remover `next` do `package.json` (dependencies)
- [x] Remover `@types/node` se exclusivo do Next.js (verificar se ainda e necessario)
- [x] Remover scripts do `package.json` que referenciam `next` (`dev`, `build`, `start`)
- [x] Remover `next.config.ts`
- [x] Remover `postcss.config.mjs` (Tailwind v4 tem integracao direta com Vite, sem PostCSS)
- [x] Remover `.next/` do diretorio (ja esta no `.gitignore`, mas verificar)
- [x] Remover `next-env.d.ts` se existir

### 1.2 Instalar TanStack Start e dependencias

- [x] Instalar `@tanstack/start` (meta-framework)
- [x] Instalar `@tanstack/react-router` (router)
- [x] Instalar `@tanstack/react-query` (server state)
- [x] Instalar `@tanstack/react-query-devtools` (devDependency)
- [x] Instalar `vinxi` (bundler base do TanStack Start)
- [x] Instalar `fontsource` equivalentes para Inter e JetBrains Mono:
  - `@fontsource-variable/inter`
  - `@fontsource-variable/jetbrains-mono`

### 1.3 Criar `app.config.ts` (substitui `next.config.ts`)

- [x] Criar `app.config.ts` na raiz com configuracao do TanStack Start
- [x] Configurar `server.proxy` para redirecionar `/connect/*` -> `VITE_API_URL` (substitui o `rewrites` do `next.config.ts`)
- [x] Configurar `vite.resolve.alias` para manter os path aliases `@/*` e `@contracts/*`
- [x] Configurar `tsr` (TanStack Router) com `routesDirectory: 'src/routes'` e `generatedRouteTree: 'src/routeTree.gen.ts'`

### 1.4 Atualizar `tsconfig.json`

- [x] Remover referencias a `next` nos types (`"types": ["next"]`)
- [x] Manter `"moduleResolution": "bundler"` (compativel com Vite)
- [x] Manter `"jsx": "react-jsx"`
- [x] Verificar e manter path aliases `@/*` e `@contracts/*`
- [x] Adicionar `src/routeTree.gen.ts` ao exclude se necessario

### 1.5 Atualizar `package.json` scripts

- [x] `"dev": "vinxi dev"` (substitui `next dev`)
- [x] `"build": "vinxi build"` (substitui `next build`)
- [x] `"start": "vinxi start"` (substitui `next start`)
- [x] Manter `"test"`, `"test:watch"`, `"test:coverage"`, `"storybook"`, `"test:visual"` sem mudanca

### 1.6 Atualizar variaveis de ambiente

- [x] Renomear `NEXT_PUBLIC_API_URL` -> `VITE_API_URL` no `.env.example`
- [x] Renomear `NEXT_PUBLIC_WS_URL` -> `VITE_WS_URL` no `.env.example`
- [x] Atualizar referencias em `src/lib/api/transport.ts` (`process.env.NEXT_PUBLIC_API_URL` -> `import.meta.env.VITE_API_URL`)
- [x] Atualizar referencias em qualquer outro arquivo que use `process.env.NEXT_PUBLIC_*`
- [x] Remover `INTERNAL_API_URL` (era usada apenas no `next.config.ts` rewrites - agora o proxy fica no `app.config.ts`)

---

## Fase 2 - Estrutura de Rotas

### 2.1 Criar o Router

- [x] Criar `src/router.ts` - instancia do `createRouter()` com a routeTree gerada
- [x] Criar `src/entry-client.tsx` - entry point do cliente com `RouterProvider` e `QueryClientProvider`
- [x] Criar `src/entry-server.tsx` - entry point do servidor (SSR) com `StartServer`

### 2.2 Criar estrutura `src/routes/`

- [x] Criar `src/routes/__root.tsx` - Root Route (substitui `src/app/layout.tsx`)
  - Importar e aplicar fontes via `@fontsource-variable/inter` e `@fontsource-variable/jetbrains-mono` (substitui `next/font/google`)
  - Manter o anti-flash script para tema dark/light (era `dangerouslySetInnerHTML` no layout do Next.js)
  - Manter `<html>`, `<body>`, e o `<Outlet />` do TanStack Router
  - Manter meta tags globais via `createRootRoute({ head: () => ({ meta: [...] }) })`

- [x] Criar `src/routes/index.tsx` - Homepage (substitui `src/app/page.tsx`)
  - Importar e renderizar `<LandingPage locale={locale} />`
  - Locale via `beforeLoad` (le cookie `apex20-locale`)

- [x] Criar `src/routes/login.tsx` - Login (substitui `src/app/login/page.tsx`)
  - Importar e renderizar `<AuthLayout>` + `<SignInForm>`
  - beforeLoad: redirecionar para `/` se ja autenticado (substitui o redirect do middleware)

- [x] Criar `src/routes/signup.tsx` - Signup (substitui `src/app/signup/page.tsx`)
  - Importar e renderizar `<AuthLayout>` + `<SignUpForm>`
  - beforeLoad: redirecionar para `/` se ja autenticado

- [x] Criar `src/routes/design-system.tsx` - Design system (substitui `src/app/design-system/page.tsx`)

### 2.3 Remover estrutura `src/app/`

- [x] Remover `src/app/layout.tsx`
- [x] Remover `src/app/page.tsx`
- [x] Remover `src/app/login/page.tsx`
- [x] Remover `src/app/signup/page.tsx`
- [x] Remover `src/app/design-system/page.tsx`
- [x] Remover `src/app/globals.css` -> mover conteudo para `src/styles/globals.css`

---

## Fase 3 - Reescrever Middleware (i18n + Auth Guard)

### 3.1 Contexto

O `src/middleware.ts` do Next.js fazia duas coisas:
1. i18n: Lia cookie `apex20-locale` ou `Accept-Language`, definia header `x-locale`
2. Auth guard: Verificava JWT no cookie `apex20-token`, redirecionava rotas protegidas

No TanStack Router, essa logica vai para `beforeLoad` nas rotas ou no `__root.tsx`.

### 3.2 Tarefas

- [x] Criar `src/lib/locale.ts` - helper para ler locale do cookie no cliente (substitui a deteccao do middleware)
  - `getLocaleFromCookie(): Locale` - le cookie `apex20-locale`, fallback para `detectLocale(navigator.language)`
  - Exportar para uso nos `beforeLoad` das rotas

- [x] Criar `src/lib/auth-guard.ts` - helper para verificar autenticacao nas rotas
  - `requireAuth(context)` - lanca `redirect({ to: '/login' })` se sem token
  - `requireGuest(context)` - lanca `redirect({ to: '/' })` se ja autenticado

- [x] Atualizar `src/routes/index.tsx` - adicionar `beforeLoad` para detectar e passar locale
- [x] Atualizar `src/routes/login.tsx` - adicionar `beforeLoad` com `requireGuest()`
- [x] Atualizar `src/routes/signup.tsx` - adicionar `beforeLoad` com `requireGuest()`
- [x] Criar `src/routes/_protected.tsx` - layout de rotas protegidas (futuras: `/dashboard`, `/campaigns`, `/vtt`)
  - `beforeLoad` com `requireAuth()`

- [x] Remover `src/middleware.ts`

---

## Fase 4 - Substituir Imports do Next.js

### 4.1 `next/link` -> `@tanstack/react-router`

Arquivos afetados (7):
- [x] `src/modules/landing/components/navbar.tsx`
- [x] `src/modules/auth/components/sign-in-form.tsx`
- [x] `src/modules/auth/components/sign-up-form.tsx`
- [x] Verificar demais arquivos com `grep -r "from 'next/link'" src/`

Mudanca:
```ts
// Antes
import Link from "next/link";
<Link href="/login">Login</Link>

// Depois
import { Link } from "@tanstack/react-router";
<Link to="/login">Login</Link>
```

### 4.2 `next/navigation` -> `@tanstack/react-router`

Arquivos afetados (2):
- [x] `src/modules/auth/components/sign-in-form.tsx`
- [x] `src/modules/auth/components/sign-up-form.tsx`

Mudanca:
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

### 4.3 `next/font` -> fontsource

Arquivo afetado: `src/routes/__root.tsx` (novo arquivo, substitui `src/app/layout.tsx`)

Mudanca:
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

### 4.4 `next/server` -> remover

- [x] Remover `src/middleware.ts` (ja coberto na Fase 3)
- [x] Verificar se ha outros imports de `next/server` com `grep -r "from 'next/server'" src/`

### 4.5 Verificacao final de imports Next.js

- [x] Executar `grep -r "from 'next" src/` - deve retornar zero resultados
- [x] Executar `grep -r "from \"next" src/` - deve retornar zero resultados

---

## Fase 5 - Metadata e SEO

### 5.1 Substituir `generateMetadata()`

O `src/app/layout.tsx` usava `generateMetadata()` para injetar title e description com i18n.

- [x] Em `src/routes/__root.tsx`, usar a API de `head` do TanStack Router:
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
- [x] Garantir que o locale seja acessivel no `head()` via search params ou context

---

## Fase 6 - Atualizar Testes

### 6.1 Mock de `next/link`

Existe `src/__mocks__/next-link.tsx` - esse mock nao e mais necessario.

- [x] Remover `src/__mocks__/next-link.tsx`
- [x] Atualizar `vitest.config.ts` se houver alias de mock para `next/link`
- [x] Verificar se algum teste usa `vi.mock('next/link')` -> substituir pelo componente real do TanStack Router ou mock equivalente

### 6.2 Atualizar testes que usam `useRouter` do Next.js

- [x] `src/modules/auth/components/sign-in-form.test.tsx` - mock de `useRouter` provavelmente referencia `next/navigation`
- [x] `src/modules/auth/components/sign-up-form.test.tsx` - idem
- [x] Substituir mocks de `next/navigation` por mocks do `@tanstack/react-router` (RouterProvider de teste)

### 6.3 Atualizar `vitest.config.ts`

- [x] Remover qualquer alias que resolvia pacotes `next/*`
- [x] Verificar se the `environment: 'jsdom'` continua adequado
- [x] Manter `setupFiles` com `src/ui/test/setup.ts`

### 6.4 Executar suite completa e corrigir

- [x] `npm run test` - resolver todos os erros um a um
- [x] `npm run typecheck` - resolver todos os erros TypeScript
- [x] `npm run lint` - resolver todos os warnings

---

## Fase 7 - Testes Visuais (Playwright/Storybook)

- [x] Verificar se as Stories do Storybook continuam funcionando (`npm run storybook`)
- [x] As stories sao componentes React puros - devem funcionar sem mudanca
- [x] Atualizar snapshots Playwright se o HTML estrutural mudar (ex: `<a>` vs `<Link>` renderiza diferente?)
- [x] `npm run test:visual` - verificar e atualizar snapshots necessarios com `--update-snapshots`

---

## Fase 8 - Build e Validacao Final

- [x] `npm run build` - build de producao sem erros
- [x] Testar o servidor em producao localmente: `npm run start`
- [x] Verificar rota `/` - landing page com i18n
- [x] Verificar rota `/login` - form de login, redirect se autenticado
- [x] Verificar rota `/signup` - form de cadastro, redirect se autenticado
- [x] Verificar rota `/design-system` - pagina carrega
- [x] Verificar proxy `/connect/*` -> backend (via `app.config.ts` server.proxy)
- [x] Verificar dark/light mode (anti-flash script no `__root.tsx`)
- [x] Verificar language switcher - troca de locale persiste no cookie

---

## Fase 9 - Limpeza e PR

- [x] Executar checklist completo de criterio de aceite (ver topo do documento)
- [x] Remover arquivos orfaos (src/app/, middleware.ts, next.config.ts, postcss.config.mjs)
- [x] Atualizar `README.md` do `apex20-web` com os novos comandos (vinxi dev/build/start)
- [x] Atualizar `.env.example` com `VITE_*` em vez de `NEXT_PUBLIC_*`
- [x] Abrir PR `refactor/nextjs-to-tanstack` -> `dev`
- [x] CI verde obrigatorio antes do merge

---

**Estimativa de esforco por fase:**

| Fase | Descricao | Complexidade |
|---|---|---|
| 1 | Setup da nova stack | Media |
| 2 | Estrutura de rotas | Alta |
| 3 | Middleware -> beforeLoad | Alta |
| 4 | Substituir imports | Baixa |
| 5 | Metadata/SEO | Baixa |
| 6 | Atualizar testes | Media |
| 7 | Testes visuais | Baixa |
| 8 | Build e validacao | Baixa |
| 9 | Limpeza e PR | Baixa |
