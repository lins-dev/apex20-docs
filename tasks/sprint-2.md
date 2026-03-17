# Detalhamento Técnico: Sprint 2 ⚔️

**Objetivo:** Inicializar a aplicação Web, definir a identidade visual base e implementar o grid interativo com sincronização.
**Status Atual:** 🟡 Em Progresso
---

## 1. Fundação do Front Web (Next.js) e Identidade Visual
- [x] **Scaffold Next.js:** Inicializar `apps/web` usando Next.js 15+ (App Router) e TypeScript.
- [x] **Definição do Padrão Visual:** Estabelecer o guia de estilo para componentes (Typography, Spacing, Shadow patterns) em `packages/ui` para evitar estilos genéricos.
- [x] **Integration UI:** Configurar o consumo de `@apex20/ui` e sincronizar o Tailwind local com os tokens do monorepo.
- [ ] **Landing Page (MVP):** Criar a página inicial com estética "Linear-like", focada em alta performance e conversão.
- [ ] **ConnectRPC Client:** Configurar o cliente de comunicação tipada para consumir os contratos de `packages/contracts`.
- [ ] **i18n Implementation:** Integrar `@apex20/i18n` para suporte multi-idioma na interface.

## 2. Autenticação e Cadastro (Novo 🔐)
- [ ] **Auth Schema:** Criar migração para a tabela `users` (UUIDv7, Argon2 hashing) com suporte a Roles (**GM, Player, Trusted**) e Permissions.
- [ ] **Auth API:** Implementar endpoints de `SignUp` e `SignIn` no `apps/backend` via ConnectRPC, incluindo a atribuição inicial de Role.
- [ ] **Auth UI (Modules):** Criar o módulo de autenticação no frontend (`modules/auth`) com formulários e lógica de proteção de rotas por Role.
- [ ] **JWT/RS256:** Implementar a geração e validação de tokens assimétricos contendo a claim `role` para autorização cross-service (ADR-002).

## 3. Gestão de Estado e Sincronização
...
- [ ] **State Orchestration:** Configurar **Zustand** para estado global e **XState** para máquinas de estado de jogo (ADR-025).
- [ ] **WebSocket Client:** Implementar o hook de conexão resiliente com o `ws-service` (exponential backoff).
- [ ] **Auth Integration:** Implementar o fluxo de persistência e envio do JWT (RS256) nos headers das requisições e no Handshake do WS (ADR-002).

## 3. Sistema de Grid (MVP)
- [ ] **Grid Canvas/SVG:** Implementar a renderização do grid baseada em coordenadas.
- [ ] **Optimistic Movement:** Implementar o arraste de tokens com atualização instantânea local e reconciliação via servidor (ADR-011).
- [ ] **Soft Locking:** Implementar sinais visuais quando um token está sendo manipulado por outro jogador.

## 4. Mecânicas Core (Backend Support)
- [ ] **Server-side Roller:** Implementar gerador de dados no `backend` usando `crypto/rand` (ADR-016).
- [ ] **Room Isolation:** Finalizar o isolamento de salas por `campaign_id` no `ws-service` via Redis (ADR-019).
- [ ] **Asset Proxy:** Implementar a entrega de imagens via Cloudflare CDN com suporte a Content Hashing (ADR-023).

## 5. Persistência e Infra
- [ ] **Migrations Sprint 2:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
- [ ] **sqlc CRUD:** Implementar os repositórios básicos para carregar o estado inicial da mesa.

---
**Critério de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apps/web`, ver a interface seguindo o novo padrão visual definido, conectar-se ao WebSocket e mover um token que seja sincronizado com outra aba do navegador.
