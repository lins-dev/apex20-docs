# Detalhamento Tecnico: Sprint 5

**Objetivo:** Implementar as mecanicas core de backend, persistencia de estado e seguranca da supply chain do backend.
**Status Atual:** 🔴 Não Iniciado

---

## 1. Segurança da Supply Chain e Auditoria (Backend)
- [ ] **Go Vulnerability Check:** Configurar `govulncheck` no pipeline de CI e localmente para auditoria de dependencias Go.
- [ ] **Dependency Pinning:** Garantir que todas as dependencias no `go.mod` estao utilizando versoes exatas e validar `go.sum`.

## 2. Mecanicas Core (Backend Support)
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

## 3. Persistencia e Infra
- [ ] **Migrations Sprint 5:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
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
**Criterio de Aceite da Sprint:** O backend deve ser capaz de gerenciar múltiplas salas isoladas no WebSocket, persistir snapshots de cenas no PostgreSQL e servir assets via Cloudflare R2 com cache infinito.
