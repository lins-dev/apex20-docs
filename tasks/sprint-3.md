# Detalhamento Tecnico: Sprint 3

**Objetivo:** Implementar o core gameplay, sincronizacao em tempo real via WebSocket e renderizacao do grid interativo.
**Status Atual:** Em Progresso

---

## 1. Polimento e Debitos Tecnicos (Migracao TanStack)
- [x] **Variaveis de Ambiente:** Corrigir `src/lib/api/transport.ts` para usar `import.meta.env.VITE_API_URL`.
- [x] **Limpeza de Arquivos:** Remover pasta `.next/` residual e referencias ao Next.js no `README.md`.
- [x] **GitIgnore:** Garantir que `.next/` e outros artefatos do Next.js sejam removidos do rastreamento.

## 2. Gestao de Estado e Sincronizacao
- [ ] **State Orchestration:** Configurar **Zustand** para estado global e **XState** para maquinas de estado de jogo (ADR-025).
  - [x] Confirmar `zustand@^5` e `xstate@^5` no `package.json` (ja listados no ADR-006)
  - [x] Criar `src/machines/game-session.machine.ts`: estados `idle | connecting | connected | reconnecting | disconnected`
  - [x] Criar `src/machines/token-movement.machine.ts`: estados `idle | dragging | pending | confirmed | rejected`
  - [x] Tests: `useSessionStore` e transicoes das maquinas com Vitest
- [x] **WebSocket Client:** Implementar o hook de conexao resiliente com o `ws-service` (exponential backoff).
  - [x] Criar `src/hooks/use-websocket.ts`: wrapper sobre WebSocket nativo com reconexao automatica
  - [x] Backoff exponencial: 1s -> 2s -> 4s -> 8s -> 16s -> 30s (cap)
  - [x] Expor `status: 'idle' | 'connecting' | 'connected' | 'reconnecting' | 'error'`
  - [x] Integrar com `useSessionStore` para sincronizar status global
  - [x] Tests: mock de WebSocket, ciclos connect/disconnect/reconnect
- [ ] **Auth Integration:** Implementar o fluxo de persistencia e envio do JWT (RS256) nos headers das requisicoes e no Handshake do WS (ADR-002).
  - Adicionar `token: string | null` e `setToken()` ao `useAuthStore` (Zustand)
  - Atualizar `src/lib/api/transport.ts`: injetar `Authorization: Bearer <token>` em todas as chamadas ConnectRPC
  - Implementar handshake WS: enviar `{ token }` como primeiro frame apos conexao
  - Tests: transport com token, guard redirect, payload do handshake

## 3. Sistema de Grid (MVP)
- [ ] **Grid Canvas/SVG:** Implementar a renderizacao do grid baseada em coordenadas.
  - Criar modulo `src/modules/grid/` com estrutura: `components/`, `hooks/`, `types/`
  - Criar `types/grid.ts`: `GridCell { x, y }`, `Token { id, x, y, imageUrl, ownerId }`
  - Criar `components/GridCanvas.tsx`: renderizacao SVG do grid quadrado com tamanho de celula configuravel
  - Criar `components/GridToken.tsx`: token SVG posicionado por coordenadas de grid
  - Criar `hooks/use-grid.ts`: estado local do grid (celulas, mapa de tokens)
  - Tests: `GridCanvas` renderiza celulas corretas, `GridToken` posiciona via coordenadas
- [ ] **Optimistic Movement:** Implementar o arraste de tokens com atualizacao instantanea local e reconciliacao via servidor (ADR-011).
  - Criar `hooks/use-token-drag.ts`: drag com `pointermove` / `pointerup`
  - Ao soltar: aplicar movimento localmente (Optimistic UI) e emitir evento WS `TOKEN_MOVE`
  - Ao receber confirmacao WS: confirmar posicao (sem mudanca visual se coincide)
  - Ao receber erro/conflito WS: reverter para posicao anterior (rollback)
  - Integrar com `token-movement.machine.ts` para controle de estado do drag
  - Tests: drag emite evento correto, rollback reverte posicao em caso de rejeicao
- [ ] **Soft Locking:** Implementar sinais visuais quando um token esta sendo manipulado por outro jogador.
  - Definir eventos WS: `TOKEN_LOCK { tokenId, lockedBy }` / `TOKEN_UNLOCK { tokenId }` em `apex20-contracts`
  - Adicionar `lockedTokens: Map<tokenId, userId>` ao store de grid
  - `GridToken`: aplicar `ring-2 ring-amber-400 animate-pulse` quando locked por outro usuario
  - Bloquear drag em tokens locked por outro usuario
  - Tests: token locked renderiza indicador visual, drag e bloqueado

## 4. Mecanicas Core (Backend Support)
- [ ] **Server-side Roller:** Implementar gerador de dados no `backend` usando `crypto/rand` (ADR-016).
  - Criar `internal/domain/dice/roller.go`: metodo `Roll(sides int) (int, error)` via `crypto/rand`
  - Criar `internal/domain/dice/roll_result.go`: struct `RollResult { Dice, Result, Timestamp }`
  - Criar `internal/application/port/dice_roller.go`: interface `DiceRoller`
  - Adicionar `RollDice(RollRequest) -> RollResult` aos contratos ConnectRPC em `apex20-contracts`
  - Handler HTTP fallback: `POST /dice/roll`
  - Tests: `roller_test.go` com tabela de N rolls por tipo de dado (d4, d6, d8, d10, d12, d20, d100)
- [ ] **Room Isolation:** Finalizar o isolamento de salas por `campaign_id` no `ws-service` via Redis (ADR-019).
  - Padronizar canal Redis: `apex20:room:{campaign_id}:events`
  - Ao receber handshake com `campaign_id` valido: subscrever no canal correspondente
  - Ao desconectar: cancelar subscricao e remover da sala
  - Validar `campaign_id` via claim do JWT (sem round-trip ao backend)
  - Tests: `room_isolation_test.go` - mensagem publicada na sala A nao chega na sala B
- [ ] **Asset Proxy:** Implementar a entrega de imagens via Cloudflare CDN com suporte a Content Hashing (ADR-023).
  - Configurar bucket Cloudflare R2 (vars: `R2_ACCOUNT_ID`, `R2_BUCKET`, `R2_ACCESS_KEY`, `R2_SECRET_KEY`)
  - Criar `internal/infrastructure/adapter/outbound/storage/r2_client.go`: upload e URL pre-assinada
  - Criar `internal/application/port/asset_storage.go`: interface `AssetStorage { Upload, GetURL }`
  - Endpoint `POST /assets/upload`: recebe multipart, retorna `{ url, hash }`
  - Endpoint `GET /assets/{hash}/{filename}`: proxy com `Cache-Control: public, max-age=31536000, immutable`
  - Tests: mock de R2, handler de upload, headers de cache

## 5. Persistencia e Infra
- [ ] **Migrations Sprint 3:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
  - `006_create_scenes.sql`: `scenes (id UUID PK, campaign_id UUID FK, name TEXT, order INT, created_at, updated_at, deleted_at)`
  - `007_add_scene_snapshot.sql`: coluna `snapshot JSONB NOT NULL DEFAULT '{}'` em `scenes` (estado serializavel: tokens, fog, background)
  - `008_create_characters.sql`: `characters (id UUID PK, campaign_id UUID FK, owner_id UUID FK, name TEXT, sheet JSONB NOT NULL DEFAULT '{}', created_at, updated_at, deleted_at)`
  - Uma migration por operacao (ADR-004); modificar in-place apenas em dev
- [ ] **sqlc CRUD:** Implementar os repositorios basicos para carregar o estado inicial da mesa.
  - `queries/scenes.sql`: `CreateScene`, `GetScene`, `ListScenesByCampaign`, `UpdateScene`, `SoftDeleteScene`
  - `queries/characters.sql`: `CreateCharacter`, `GetCharacter`, `ListCharactersByCampaign`, `UpdateCharacterSheet`, `SoftDeleteCharacter`
  - Rodar `sqlc generate` e commitar codigo gerado em `gen/`
  - Implementar `PostgresSceneRepository` e `PostgresCharacterRepository` em `internal/infrastructure/adapter/outbound/repository/`
  - Tests de integracao com Testcontainers (ADR-010)

---
**Criterio de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apex20-web`, ver a interface seguindo o novo padrao visual definido, conectar-se ao WebSocket e mover um token que seja sincronizado com outra aba do navegador.
