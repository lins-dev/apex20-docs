# Detalhamento Tecnico: Sprint 3

**Objetivo:** Garantir a integridade da supply chain do frontend e consolidar a gestão de estado inicial.
**Status Atual:** 🟡 Em Progresso

---

## 1. Polimento e Debitos Tecnicos (Migracao TanStack)
- [x] **Variaveis de Ambiente:** Corrigir `src/lib/api/transport.ts` para usar `import.meta.env.VITE_API_URL`.
- [x] **Limpeza de Arquivos:** Remover pasta `.next/` residual e referencias ao Next.js no `README.md`.
- [x] **GitIgnore:** Garantir que `.next/` e outros artefatos do Next.js sejam removidos do rastreamento.

## 2. Segurança da Supply Chain e Auditoria (Frontend)
- [x] **Configuração Zero Trust:** Implementar `.npmrc` com `ignore-scripts`, `min-release-age=7d` e `save-exact=true` (ADR-015).
- [ ] **Instalação Blindada:** Criar script `scripts/secure-install.js` para validação rigorosa de idade de pacotes (constante 7 dias) e integração com workflow.
- [ ] **Correção de Vulnerabilidades:** Atualizar pacotes críticos seguindo as regras de segurança:
    - [ ] `vitest` (Critical)
    - [ ] `vite` (High)
    - [ ] `h3` / `nitropack` (High)
    - [ ] `lodash` / `defu` (High)
- [ ] **Enforcement de Segurança:** Adicionar Git Hook (Husky) para impedir commits com vulnerabilidades não auditadas.

## 3. Gestao de Estado e Sincronizacao (Base)
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
- [x] **Auth Integration:** Implementar o fluxo de persistencia e envio do JWT (RS256) nos headers das requisicoes e no Handshake do WS (ADR-002).
  - [x] Adicionar `token: string | null` e `setToken()` ao `useAuthStore` (Zustand)
  - [x] Atualizar `src/lib/api/transport.ts`: injetar `Authorization: Bearer <token>` em todas as chamadas ConnectRPC
  - [x] Implementar handshake WS: enviar `{ token }` como primeiro frame apos conexao
  - [x] Tests: transport com token, guard redirect, payload do handshake

---
**Criterio de Aceite da Sprint:** O ecossistema de segurança do frontend deve estar funcional (zero vulnerabilidades críticas/altas) e a base de sincronização via WebSocket deve estar validada com testes de reconexão.
