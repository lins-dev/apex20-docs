# Detalhamento Tecnico: Sprint 4

**Objetivo:** Implementar o Sistema de Grid (MVP) com renderização interativa e sincronização básica de tokens.
**Status Atual:** 🟡 Em Progresso

---

## 1. Sistema de Grid (MVP)
- [x] **Grid Canvas/SVG:** Implementar a renderizacao do grid baseada em coordenadas.
  - Criar modulo `src/modules/grid/` com estrutura: `components/`, `hooks/`, `types/`
  - Utilizar os tipos Protobuf gerados (`Vec2`, `TokenDelta`) em substituição a tipos locais.
  - Criar `components/GridCanvas.tsx`: renderizacao SVG do grid quadrado com tamanho de celula configuravel
  - Criar `components/GridToken.tsx`: token SVG posicionado por coordenadas de grid
  - Criar `store/grid-store.ts`: estado local do grid (configurações, mapa de tokens)
  - Tests: `GridCanvas` renderiza celulas corretas, `GridToken` posiciona via coordenadas
- [x] **Optimistic Movement:** Implementar o arraste de tokens com atualizacao instantanea local e reconciliacao via servidor (ADR-011).
  - Criar `hooks/use-token-drag.ts`: drag com `pointermove` / `pointerup`
  - Ao soltar: aplicar movimento localmente (Optimistic UI) e emitir evento WS `TOKEN_MOVE`
  - Ao receber confirmacao WS: confirmar posicao (sem mudanca visual se coincide)
  - Ao receber erro/conflito WS: reverter para posicao anterior (rollback)
  - Tests: drag emite evento correto, rollback reverte posicao em caso de rejeicao
- [x] **Soft Locking:** Implementar sinais visuais quando um token esta sendo manipulado por outro jogador.
  - Bloquear drag em tokens locked por outro usuario
  - `GridToken`: aplicar `ring-2 ring-amber-400 animate-pulse` quando locked por outro usuario
  - Tests: token locked renderiza indicador visual, drag e bloqueado

---
**Criterio de Aceite da Sprint:** O usuário deve conseguir mover um token no grid de forma fluida (Optimistic UI) e a movimentação deve ser refletida em outras sessões via WebSocket com suporte a soft-locking.
