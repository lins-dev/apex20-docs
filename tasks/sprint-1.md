# Detalhamento Técnico: Sprint 1 🛠️

**Objetivo:** Estabelecer a base do monorepo, contratos de comunicação e infraestrutura inicial.
**Status Atual:** 🟢 Concluído

---

## 1. Fundação do Monorepo (Turborepo + pnpm)
- [x] Criar `pnpm-workspace.yaml` definindo `apps/*` e `packages/*`.
- [x] Inicializar `package.json` na raiz com scripts de orquestração.
- [x] Configurar `turbo.json` com pipelines de `build`, `lint`, `test` e `dev`.
- [x] Configurar TypeScript Base (`packages/config-typescript`) para compartilhamento de regras.
- [x] Configurar ESLint e Prettier globais integrados ao Turborepo.
- [x] Configurar Husky e lint-staged para Git Hooks (Pre-commit linting).

## 2. Contratos e Serialização (Protobuf)
- [x] Configurar `packages/contracts` com a ferramenta **Buf**.
- [x] Definir `handshake.proto` (Autenticação e Inicialização).
- [x] Definir `chat.proto` (Mensageria em tempo real).
- [x] Definir `grid_events.proto` (Movimentação e Estados do mapa).
- [x] Implementar scripts de geração de código para Go e TypeScript.

## 3. Scaffold de Infraestrutura e Backend
- [x] Criar `docker-compose.yml` (PostgreSQL 16, Redis 7, Prometheus).
- [x] Configurar `apps/backend` com estrutura de **Arquitetura Hexagonal**.
- [x] Configurar `apps/ws-service` com estrutura de **Arquitetura Hexagonal**.
- [x] Inicializar ferramenta de migração **Tern** e configuração do **sqlc**.

## 4. Pacotes Compartilhados (Core)
- [x] Setup de `packages/i18n` com suporte inicial a EN e PT-BR.
- [x] Setup de `packages/ui` com **Storybook** e componentes base **shadcn/ui**.
- [x] Configurar `packages/sensors` (Sensor Abstraction Layer - ADR-020).


## 5. Validação de Integração Inicial
- [x] Implementar teste de "Ping-Pong" entre `backend` e `ws-service` via Redis.
- [x] Validar pipeline de CI rodando os primeiros testes de infraestrutura.

---
**Critério de Aceite da Sprint:** O comando `apex20 dev` deve subir todos os serviços e o comando `apex20 test` deve passar com 100% de cobertura nos boilerplates.
