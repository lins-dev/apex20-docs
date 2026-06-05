# Detalhamento Tecnico: Sprint 2

**Objetivo:** Inicializar a aplicacao Web, definir a identidade visual base e implementar a base de autenticacao e gestao de campanhas.
**Status Atual:** Concluido

---

## 1. Fundacao do Front Web (TanStack Start) e Identidade Visual
- [x] **Scaffold:** Inicializar `apex20-web` com Next.js 16 (App Router) -> migrado para **React 19 + TanStack Start + Vite** (Sprint M / ADR-038).
- [x] **Definicao do Padrao Visual:** Estabelecer o guia de estilo para componentes (Typography, Spacing, Shadow patterns) em `apex20-web/src/ui/` para evitar estilos genericos.
- [x] **Integration UI:** Configurar o consumo de `@/ui` e tokens do Tailwind local.
- [x] **Landing Page (MVP):** Criar a pagina inicial com estetica "Linear-like", focada em alta performance e conversao.
  - Hero split 52/48 com AppMockup simulando VTT em tempo real
  - Systems bar, 3 secoes alternadas (Grid Sincronizado, Visao Computacional, IA & Automacao)
  - 6-card features grid, CTA banner e footer 4 colunas
  - Dark/light mode com `useTheme` hook, localStorage e anti-flash script
  - Menu hamburguer mobile com estado controlado
  - Link "Sobre nos" em todos os idiomas
- [x] **i18n Implementation:** Integrar `@/i18n` para suporte multi-idioma na interface.
  - 4 locales: `pt-br`, `en`, `es`, `fr`
  - `t()` com dot-notation (ex: `t("landing.nav.features", locale)`)
  - `detectLocale()` via header `Accept-Language` com fallback para `"en"`
  - Locale detectado via `beforeLoad` hook do TanStack Router (cookie > Accept-Language > fallback)
  - `LanguageSwitcher` dropdown customizado com bandeiras SVG (Brasil, EUA, Espanha, Franca)
- [x] **Testes Unitarios (TDD):** Configurar Vitest e implementar testes com abordagem red->green.
  - `vitest.config.ts` com jsdom e path aliases; arquivos de teste incluidos no tsconfig principal
  - 86 testes passando: i18n, locale-detection, language-switcher, navbar, button, clients, transport, nivo
  - Cobertura: deteccao de locale, dropdown de idioma, menu hamburguer, links traduzidos, ConnectRPC client
- [x] **Testes Visuais:** Implementar testes de regressao visual nos componentes da landing page.
  - Decisao (ADR-031): Storybook + Playwright visual regression (Docker para snapshots deterministicos)
  - Stories implementadas: `cta-banner`, `features`, `footer`, `systems`, `button`, `tokens`
  - Specs Playwright: `playwright/visual/landing.spec.ts` e `playwright/visual/ui.spec.ts`
- [x] **ConnectRPC Client:** Configurar o cliente de comunicacao tipada para consumir os contratos do submodule `./contracts/gen/ts/` (via alias `@contracts/*`).
  - Implementado em `src/lib/api/clients.ts` e `src/lib/api/transport.ts`
  - Env vars via `.env.example` (`VITE_API_URL`); proxy via Nitro `routeRules` em `vite.config.mts`

## 2. Autenticacao e Cadastro (Novo)
- [x] **Modelagem de Roles (DB + Contratos + Domain):** Refatorar toda a camada de dados para roles campaign-scoped (ADR-002):
  - `001_create_users.sql`: remove `role`, adiciona `is_admin BOOLEAN NOT NULL DEFAULT false`
  - `005_create_campaign_members.sql`: nova tabela `campaign_members (id, campaign_id, user_id, role, created_at, updated_at)` com UNIQUE `(campaign_id, user_id)`
  - `apex20-contracts`: `ROLE_ADMIN` removido do proto; contratos Go e TS regenerados
  - `domain/campaign/member.go`: nova entidade `Member` com roles GM, Player, Trusted
  - `domain/permission/role.go`: constante `RoleAdmin` removida
  - `sqlc.yaml` + `gen/`: override e modelos atualizados; novo `CampaignMember` gerado
  - `seed_role_permissions`: bloco admin removido; `/admin/roles` retorna apenas GM, Player, Trusted
- [x] **Auth Schema (Fluxo de Aplicacao):** Implementar a logica de negocio que garante a integridade da modelagem:
  - Ao criar campanha: inserir automaticamente o criador em `campaign_members` como `gm`
  - Ao convidar usuario: inserir em `campaign_members` como `player` ou `trusted`
  - Validacao de role por `campaign_id` nas requisicoes HTTP e handshake WS
  - Use cases: `CreateCampaign`, `InviteMember`, `GetMemberRole`
  - Repositorios: `PostgresCampaignRepository` (transacao atomica), `PostgresCampaignMemberRepository`
  - Handler HTTP: `POST /campaigns`
- [x] **Campaign CRUD:** Implementar os endpoints REST completos de campanhas no `apex20-backend`:
  - `GET /campaigns` - listar campanhas do usuario autenticado
  - `GET /campaigns/{id}` - obter campanha por ID
  - `PUT /campaigns/{id}` - atualizar nome e descricao (`description` nullable)
  - `DELETE /campaigns/{id}` - soft delete
- [x] **Campaign Members API:** Gerenciar membros de uma campanha:
  - `POST /campaigns/{id}/members` - convidar jogador (`player` ou `trusted`)
  - `DELETE /campaigns/{id}/members/{userId}` - remover jogador da campanha
- [x] **Auth API:** Implementar endpoints de `SignUp` e `SignIn` no `apex20-backend` via ConnectRPC, incluindo hashing Argon2 e geracao de JWT RS256.
- [x] **Auth UI (Modules):** Criar o modulo de autenticacao no frontend (`modules/auth`) com formularios e logica de protecao de rotas por `is_admin`.
- [x] **JWT/RS256:** Implementar geracao e validacao de tokens assimetricos com claims `sub` e `is_admin`. Role de campanha e resolvida dinamicamente via `campaign_members` por `campaign_id` (ADR-002).
- [x] **Navbar: redistribuicao de espaco para evitar quebra de linha em locales longos (FR):** Em frances, os botoes "Se connecter" e "Commencer gratuitement" quebram para duas linhas porque o espaco fixo dos nav links nao se adapta ao tamanho do conteudo. A navbar ainda tem espaco disponivel - redistribuir usando `min-w-0` + `shrink` nos nav links e `shrink-0` nos botoes de acao para garantir que os CTAs sempre fiquem em linha unica.
- [x] **Auth UI Redesign (Padronizacao de Layout):** Ajustar as paginas de login e cadastro para seguirem o padrao visual do produto (Navbar + Footer compartilhados) com layout inspirado no Roll20:
  - `AuthLayout` (`modules/auth/components/auth-layout.tsx`): split panel - painel esquerdo de branding + painel direito com formulario
  - Navbar: botoes "Login" e "Sign Up" convertidos para `<Link>` com destinos `/login` e `/signup`
  - `SignUpForm`: adicionar campo "Confirmar senha" com validacao de igualdade via Zod
  - i18n: atualizar as 4 locales (`en`, `pt-br`, `es`, `fr`) com as chaves de confirmPassword
  - Testes: atualizar cobertura de `sign-up-form.test.tsx` e `navbar.test.tsx`

## 3. Padronizacao de Pacotes Frontend (ADR-019)
- [x] **Adotar pacotes padrao ausentes no `apex20-web`:** Incorporar as bibliotecas definidas como padrao em todos os projetos frontend:
  - `zod` - validacao de esquemas (forms e I/O de API)
  - `react-hook-form` + `@hookform/resolvers` - gerenciamento de formularios (necessario para Auth UI)
  - `@tanstack/react-query` - estado de servidor e cache de dados
  - `@nivo/core` + `@nivo/bar` + `@nivo/line` - visualizacao de dados (stats de sessao, historico de rolagens, player analytics)

---
**Criterio de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apex20-web`, ver a interface seguindo o novo padrao visual definido e autenticar-se com sucesso.
