# Detalhamento Técnico: Sprint 2 ⚔️

**Objetivo:** Inicializar a aplicação Web, definir a identidade visual base e implementar o grid interativo com sincronização.
**Status Atual:** 🟡 Em Progresso
---

## 1. Fundação do Front Web (Next.js) e Identidade Visual
- [x] **Scaffold Next.js:** Inicializar `apex20-web` usando Next.js 16+ (App Router) e TypeScript.
- [x] **Definição do Padrão Visual:** Estabelecer o guia de estilo para componentes (Typography, Spacing, Shadow patterns) em `apex20-web/src/ui/` para evitar estilos genéricos.
- [x] **Integration UI:** Configurar o consumo de `@/ui` e tokens do Tailwind local.
- [x] **Landing Page (MVP):** Criar a página inicial com estética "Linear-like", focada em alta performance e conversão.
  - Hero split 52/48 com AppMockup simulando VTT em tempo real
  - Systems bar, 3 seções alternadas (Grid Sincronizado, Visão Computacional, IA & Automação)
  - 6-card features grid, CTA banner e footer 4 colunas
  - Dark/light mode com `useTheme` hook, localStorage e anti-flash script
  - Menu hamburguer mobile com estado controlado
  - Link "Sobre nós" em todos os idiomas
- [x] **i18n Implementation:** Integrar `@/i18n` para suporte multi-idioma na interface.
  - 4 locales: `pt-br`, `en`, `es`, `fr`
  - `t()` com dot-notation (ex: `t("landing.nav.features", locale)`)
  - `detectLocale()` via header `Accept-Language` com fallback para `"en"`
  - Middleware Next.js detecta locale por cookie > Accept-Language > fallback
  - `LanguageSwitcher` dropdown customizado com bandeiras SVG (Brasil, EUA, Espanha, França)
- [x] **Testes Unitários (TDD):** Configurar Vitest e implementar testes com abordagem red→green.
  - `vitest.config.ts` com jsdom e path aliases
  - 50+ testes passando: i18n, locale-detection, language-switcher, navbar, button, clients, transport
  - Cobertura: detecção de locale, dropdown de idioma, menu hamburguer, links traduzidos, ConnectRPC client
- [x] **Testes Visuais:** Implementar testes de regressão visual nos componentes da landing page.
  - Decisão (ADR-031): Storybook + Playwright visual regression (Docker para snapshots determinísticos)
  - Stories implementadas: `cta-banner`, `features`, `footer`, `systems`, `button`, `tokens`
  - Specs Playwright: `playwright/visual/landing.spec.ts` e `playwright/visual/ui.spec.ts`
- [x] **ConnectRPC Client:** Configurar o cliente de comunicação tipada para consumir os contratos do submodule `./contracts/gen/ts/` (via alias `@contracts/*`).
  - Implementado em `src/lib/api/clients.ts` e `src/lib/api/transport.ts`
  - Env vars via `.env.example` (NEXT_PUBLIC_API_URL)

## 2. Autenticação e Cadastro (Novo 🔐)
- [x] **Modelagem de Roles (DB + Contratos + Domain):** Refatorar toda a camada de dados para roles campaign-scoped (ADR-002):
  - `001_create_users.sql`: remove `role`, adiciona `is_admin BOOLEAN NOT NULL DEFAULT false`
  - `005_create_campaign_members.sql`: nova tabela `campaign_members (id, campaign_id, user_id, role, created_at, updated_at)` com UNIQUE `(campaign_id, user_id)`
  - `apex20-contracts`: `ROLE_ADMIN` removido do proto; contratos Go e TS regenerados
  - `domain/campaign/member.go`: nova entidade `Member` com roles GM, Player, Trusted
  - `domain/permission/role.go`: constante `RoleAdmin` removida
  - `sqlc.yaml` + `gen/`: override e modelos atualizados; novo `CampaignMember` gerado
  - `seed_role_permissions`: bloco admin removido; `/admin/roles` retorna apenas GM, Player, Trusted
- [x] **Auth Schema (Fluxo de Aplicação):** Implementar a lógica de negócio que garante a integridade da modelagem:
  - Ao criar campanha: inserir automaticamente o criador em `campaign_members` como `gm`
  - Ao convidar usuário: inserir em `campaign_members` como `player` ou `trusted`
  - Validação de role por `campaign_id` nas requisições HTTP e handshake WS
  - Use cases: `CreateCampaign`, `InviteMember`, `GetMemberRole`
  - Repositórios: `PostgresCampaignRepository` (transação atômica), `PostgresCampaignMemberRepository`
  - Handler HTTP: `POST /campaigns`
- [x] **Campaign CRUD:** Implementar os endpoints REST completos de campanhas no `apex20-backend`:
  - `GET /campaigns` — listar campanhas do usuário autenticado
  - `GET /campaigns/{id}` — obter campanha por ID
  - `PUT /campaigns/{id}` — atualizar nome e descrição (`description` nullable)
  - `DELETE /campaigns/{id}` — soft delete
- [x] **Campaign Members API:** Gerenciar membros de uma campanha:
  - `POST /campaigns/{id}/members` — convidar jogador (`player` ou `trusted`)
  - `DELETE /campaigns/{id}/members/{userId}` — remover jogador da campanha
- [x] **Auth API:** Implementar endpoints de `SignUp` e `SignIn` no `apex20-backend` via ConnectRPC, incluindo hashing Argon2 e geração de JWT RS256.
- [ ] **Auth UI (Modules):** Criar o módulo de autenticação no frontend (`modules/auth`) com formulários e lógica de proteção de rotas por `is_admin`.
- [ ] **JWT/RS256:** Implementar geração e validação de tokens assimétricos com claims `sub` e `is_admin`. Role de campanha é resolvida dinamicamente via `campaign_members` por `campaign_id` (ADR-002).

## 3. Gestão de Estado e Sincronização
- [ ] **State Orchestration:** Configurar **Zustand** para estado global e **XState** para máquinas de estado de jogo (ADR-025).
- [ ] **WebSocket Client:** Implementar o hook de conexão resiliente com o `ws-service` (exponential backoff).
- [ ] **Auth Integration:** Implementar o fluxo de persistência e envio do JWT (RS256) nos headers das requisições e no Handshake do WS (ADR-002).

## 4. Sistema de Grid (MVP)
- [ ] **Grid Canvas/SVG:** Implementar a renderização do grid baseada em coordenadas.
- [ ] **Optimistic Movement:** Implementar o arraste de tokens com atualização instantânea local e reconciliação via servidor (ADR-011).
- [ ] **Soft Locking:** Implementar sinais visuais quando um token está sendo manipulado por outro jogador.

## 5. Mecânicas Core (Backend Support)
- [ ] **Server-side Roller:** Implementar gerador de dados no `backend` usando `crypto/rand` (ADR-016).
- [ ] **Room Isolation:** Finalizar o isolamento de salas por `campaign_id` no `ws-service` via Redis (ADR-019).
- [ ] **Asset Proxy:** Implementar a entrega de imagens via Cloudflare CDN com suporte a Content Hashing (ADR-023).

## 6. Padronização de Pacotes Frontend (ADR-019)
- [ ] **Adotar pacotes padrão ausentes no `apex20-web`:** Incorporar as bibliotecas definidas como padrão em todos os projetos frontend:
  - `zod` — validação de esquemas (forms e I/O de API)
  - `react-hook-form` + `@hookform/resolvers` — gerenciamento de formulários (necessário para Auth UI)
  - `@tanstack/react-query` — estado de servidor e cache de dados
  - `@nivo/core` + `@nivo/bar` + `@nivo/line` — visualização de dados (stats de sessão, histórico de rolagens, player analytics)

## 7. Persistência e Infra
- [ ] **Migrations Sprint 2:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
- [ ] **sqlc CRUD:** Implementar os repositórios básicos para carregar o estado inicial da mesa.

---
**Critério de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apex20-web`, ver a interface seguindo o novo padrão visual definido, conectar-se ao WebSocket e mover um token que seja sincronizado com outra aba do navegador.
