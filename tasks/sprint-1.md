# Detalhamento Técnico: Sprint 1 🛠️

**Objetivo:** Estabelecer a base do polyrepo, contratos de comunicação e infraestrutura inicial.
**Status Atual:** 🟢 Concluído

---

## 1. Fundação do Polyrepo (Git Submodules)
- [x] Criar os repositórios independentes: `apex20-docs`, `apex20-contracts`, `apex20-web`, `apex20-backend`, `apex20-ws`.
- [x] Adicionar `apex20-docs` e `apex20-contracts` como git submodules em cada repositório consumidor.
- [x] Configurar TypeScript Base (`tsconfig.json`) no `apex20-web` com path aliases para `@/*` e `@contracts/*`.
- [x] Configurar ESLint e Prettier no `apex20-web` (por repositório, sem configuração compartilhada de workspace).
- [x] Configurar Husky e lint-staged para Git Hooks (Pre-commit linting).

## 2. Contratos e Serialização (Protobuf)
- [x] Configurar `apex20-contracts` com a ferramenta **Buf**.
- [x] Definir `handshake.proto` (Autenticação e Inicialização).
- [x] Definir `chat.proto` (Mensageria em tempo real).
- [x] Definir `grid_events.proto` (Movimentação e Estados do mapa).
- [x] Implementar geração de código para Go (`gen/go/`) e TypeScript (`gen/ts/`) via `buf generate` — código gerado commitado no repositório.

## 3. Scaffold de Infraestrutura e Backend
- [x] Criar `docker-compose.yml` (PostgreSQL 16, Redis 7, Prometheus).
- [x] Configurar `apex20-backend` com estrutura de **Arquitetura Hexagonal**.
- [x] Configurar `apex20-ws` com estrutura de **Arquitetura Hexagonal**.
- [x] Inicializar ferramenta de migração **Tern** e configuração do **sqlc**.

## 4. Frontend e Módulos Internalizados
- [x] Setup do módulo i18n em `apex20-web/src/i18n/` com suporte inicial a EN e PT-BR.
- [x] Setup do design system em `apex20-web/src/ui/` com componentes base **shadcn/ui**.
- [x] Configurar `apex20-web/src/sensors/` (Sensor Abstraction Layer - ADR-020).

## 5. Validação de Integração Inicial
- [x] Implementar teste de "Ping-Pong" entre `apex20-backend` e `apex20-ws` via Redis.
- [x] Validar pipeline de CI rodando os primeiros testes de infraestrutura.

---
**Critério de Aceite da Sprint:** O comando `apex20 dev` deve subir todos os serviços e o comando `apex20 test` deve passar com 100% de cobertura nos boilerplates.
