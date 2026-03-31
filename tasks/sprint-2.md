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
- [x] **Auth UI (Modules):** Criar o módulo de autenticação no frontend (`modules/auth`) com formulários e lógica de proteção de rotas por `is_admin`.
- [x] **JWT/RS256:** Implementar geração e validação de tokens assimétricos com claims `sub` e `is_admin`. Role de campanha é resolvida dinamicamente via `campaign_members` por `campaign_id` (ADR-002).
- [ ] **Navbar: redistribuição de espaço para evitar quebra de linha em locales longos (FR):** Em francês, os botões "Se connecter" e "Commencer gratuitement" quebram para duas linhas porque o espaço fixo dos nav links não se adapta ao tamanho do conteúdo. A navbar ainda tem espaço disponível — redistribuir usando `min-w-0` + `shrink` nos nav links e `shrink-0` nos botões de ação para garantir que os CTAs sempre fiquem em linha única.
- [x] **Auth UI Redesign (Padronização de Layout):** Ajustar as páginas de login e cadastro para seguirem o padrão visual do produto (Navbar + Footer compartilhados) com layout inspirado no Roll20:
  - `AuthLayout` (`modules/auth/components/auth-layout.tsx`): split panel — painel esquerdo de branding + painel direito com formulário
  - Navbar: botões "Login" e "Sign Up" convertidos para `<Link>` com destinos `/login` e `/signup`
  - `SignUpForm`: adicionar campo "Confirmar senha" com validação de igualdade via Zod
  - i18n: atualizar as 4 locales (`en`, `pt-br`, `es`, `fr`) com as chaves de confirmPassword
  - Testes: atualizar cobertura de `sign-up-form.test.tsx` e `navbar.test.tsx`

## 3. Gestão de Estado e Sincronização
- [ ] **State Orchestration:** Configurar **Zustand** para estado global e **XState** para máquinas de estado de jogo (ADR-025).
  - Confirmar `zustand@^5` e `xstate@^5` no `package.json` (já listados no ADR-006)
  - Criar `src/store/session.ts`: `useSessionStore` com `{ campaignId, sceneId, connected }`
  - Criar `src/machines/game-session.machine.ts`: estados `idle | connecting | connected | reconnecting | disconnected`
  - Criar `src/machines/token-movement.machine.ts`: estados `idle | dragging | pending | confirmed | rejected`
  - Tests: `useSessionStore` e transições das máquinas com Vitest
- [ ] **WebSocket Client:** Implementar o hook de conexão resiliente com o `ws-service` (exponential backoff).
  - Criar `src/hooks/use-websocket.ts`: wrapper sobre WebSocket nativo com reconexão automática
  - Backoff exponencial: 1s → 2s → 4s → 8s → 16s → 30s (cap)
  - Expor `status: 'idle' | 'connecting' | 'connected' | 'reconnecting' | 'error'`
  - Integrar com `useSessionStore` para sincronizar status global
  - Tests: mock de WebSocket, ciclos connect/disconnect/reconnect
- [ ] **Auth Integration:** Implementar o fluxo de persistência e envio do JWT (RS256) nos headers das requisições e no Handshake do WS (ADR-002).
  - Adicionar `token: string | null` e `setToken()` ao `useAuthStore` (Zustand)
  - Atualizar `src/lib/api/transport.ts`: injetar `Authorization: Bearer <token>` em todas as chamadas ConnectRPC
  - Implementar handshake WS: enviar `{ token }` como primeiro frame após conexão
  - Tests: transport com token, guard redirect, payload do handshake

## 4. Sistema de Grid (MVP)
- [ ] **Grid Canvas/SVG:** Implementar a renderização do grid baseada em coordenadas.
  - Criar módulo `src/modules/grid/` com estrutura: `components/`, `hooks/`, `types/`
  - Criar `types/grid.ts`: `GridCell { x, y }`, `Token { id, x, y, imageUrl, ownerId }`
  - Criar `components/GridCanvas.tsx`: renderização SVG do grid quadrado com tamanho de célula configurável
  - Criar `components/GridToken.tsx`: token SVG posicionado por coordenadas de grid
  - Criar `hooks/use-grid.ts`: estado local do grid (células, mapa de tokens)
  - Tests: `GridCanvas` renderiza células corretas, `GridToken` posiciona via coordenadas
- [ ] **Optimistic Movement:** Implementar o arraste de tokens com atualização instantânea local e reconciliação via servidor (ADR-011).
  - Criar `hooks/use-token-drag.ts`: drag com `pointermove` / `pointerup`
  - Ao soltar: aplicar movimento localmente (Optimistic UI) e emitir evento WS `TOKEN_MOVE`
  - Ao receber confirmação WS: confirmar posição (sem mudança visual se coincide)
  - Ao receber erro/conflito WS: reverter para posição anterior (rollback)
  - Integrar com `token-movement.machine.ts` para controle de estado do drag
  - Tests: drag emite evento correto, rollback reverte posição em caso de rejeição
- [ ] **Soft Locking:** Implementar sinais visuais quando um token está sendo manipulado por outro jogador.
  - Definir eventos WS: `TOKEN_LOCK { tokenId, lockedBy }` / `TOKEN_UNLOCK { tokenId }` em `apex20-contracts`
  - Adicionar `lockedTokens: Map<tokenId, userId>` ao store de grid
  - `GridToken`: aplicar `ring-2 ring-amber-400 animate-pulse` quando locked por outro usuário
  - Bloquear drag em tokens locked por outro usuário
  - Tests: token locked renderiza indicador visual, drag é bloqueado

## 5. Mecânicas Core (Backend Support)
- [ ] **Server-side Roller:** Implementar gerador de dados no `backend` usando `crypto/rand` (ADR-016).
  - Criar `internal/domain/dice/roller.go`: método `Roll(sides int) (int, error)` via `crypto/rand`
  - Criar `internal/domain/dice/roll_result.go`: struct `RollResult { Dice, Result, Timestamp }`
  - Criar `internal/application/port/dice_roller.go`: interface `DiceRoller`
  - Adicionar `RollDice(RollRequest) → RollResult` aos contratos ConnectRPC em `apex20-contracts`
  - Handler HTTP fallback: `POST /dice/roll`
  - Tests: `roller_test.go` com tabela de N rolls por tipo de dado (d4, d6, d8, d10, d12, d20, d100)
- [ ] **Room Isolation:** Finalizar o isolamento de salas por `campaign_id` no `ws-service` via Redis (ADR-019).
  - Padronizar canal Redis: `apex20:room:{campaign_id}:events`
  - Ao receber handshake com `campaign_id` válido: subscrever no canal correspondente
  - Ao desconectar: cancelar subscrição e remover da sala
  - Validar `campaign_id` via claim do JWT (sem round-trip ao backend)
  - Tests: `room_isolation_test.go` — mensagem publicada na sala A não chega na sala B
- [ ] **Asset Proxy:** Implementar a entrega de imagens via Cloudflare CDN com suporte a Content Hashing (ADR-023).
  - Configurar bucket Cloudflare R2 (vars: `R2_ACCOUNT_ID`, `R2_BUCKET`, `R2_ACCESS_KEY`, `R2_SECRET_KEY`)
  - Criar `internal/infrastructure/adapter/outbound/storage/r2_client.go`: upload e URL pré-assinada
  - Criar `internal/application/port/asset_storage.go`: interface `AssetStorage { Upload, GetURL }`
  - Endpoint `POST /assets/upload`: recebe multipart, retorna `{ url, hash }`
  - Endpoint `GET /assets/{hash}/{filename}`: proxy com `Cache-Control: public, max-age=31536000, immutable`
  - Tests: mock de R2, handler de upload, headers de cache

## 6. Padronização de Pacotes Frontend (ADR-019)
- [ ] **Adotar pacotes padrão ausentes no `apex20-web`:** Incorporar as bibliotecas definidas como padrão em todos os projetos frontend:
  - `zod` — validação de esquemas (forms e I/O de API)
  - `react-hook-form` + `@hookform/resolvers` — gerenciamento de formulários (necessário para Auth UI)
  - `@tanstack/react-query` — estado de servidor e cache de dados
  - `@nivo/core` + `@nivo/bar` + `@nivo/line` — visualização de dados (stats de sessão, histórico de rolagens, player analytics)

## 7. Persistência e Infra
- [ ] **Migrations Sprint 2:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
  - `006_create_scenes.sql`: `scenes (id UUID PK, campaign_id UUID FK, name TEXT, order INT, created_at, updated_at, deleted_at)`
  - `007_add_scene_snapshot.sql`: coluna `snapshot JSONB NOT NULL DEFAULT '{}'` em `scenes` (estado serializável: tokens, fog, background)
  - `008_create_characters.sql`: `characters (id UUID PK, campaign_id UUID FK, owner_id UUID FK, name TEXT, sheet JSONB NOT NULL DEFAULT '{}', created_at, updated_at, deleted_at)`
  - Uma migration por operação (ADR-004); modificar in-place apenas em dev
- [ ] **sqlc CRUD:** Implementar os repositórios básicos para carregar o estado inicial da mesa.
  - `queries/scenes.sql`: `CreateScene`, `GetScene`, `ListScenesByCampaign`, `UpdateScene`, `SoftDeleteScene`
  - `queries/characters.sql`: `CreateCharacter`, `GetCharacter`, `ListCharactersByCampaign`, `UpdateCharacterSheet`, `SoftDeleteCharacter`
  - Rodar `sqlc generate` e commitar código gerado em `gen/`
  - Implementar `PostgresSceneRepository` e `PostgresCharacterRepository` em `internal/infrastructure/adapter/outbound/repository/`
  - Tests de integração com Testcontainers (ADR-010)

---
**Critério de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apex20-web`, ver a interface seguindo o novo padrão visual definido, conectar-se ao WebSocket e mover um token que seja sincronizado com outra aba do navegador.
