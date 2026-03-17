# ADR-025: Gestão de Estados de Jogo e Máquinas de Estado (XState/Zustand) 🧠

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 é um sistema **Local-first** onde a interface deve reagir de forma fluida às ações do usuário. A principal usabilidade envolve a manipulação de tokens em um mapa enquanto mestre e jogadores narram a cena. Diferente de um jogo de ação frenético, o VTT exige precisão e estabilidade de estado, não necessariamente altas taxas de quadros para todos os usuários. Gerenciar o fluxo de combate, turnos e sincronização exige uma lógica determinística para evitar inconsistências visuais.

## Decisão
Adotar uma arquitetura de estado híbrida no Frontend (Next.js/Expo), combinando **Zustand** para armazenamento de dados e **XState** para orquestração de lógica complexa, com foco em eficiência computacional.

### 1. Estado Global Reativo (Zustand)
- **Papel:** Atuar como o repositório central de dados (Store) para o Grid, Chat e Fichas.
- **Otimização de Performance:** Configurado para garantir uma taxa estável de **30 FPS** no padrão de entrega. Isso reduz o overhead de renderização e o consumo de bateria/CPU, focando na legibilidade da cena e fluidez da narração.
- **Escalabilidade:** A estrutura deve permitir o desbloqueio para **60 FPS** como um recurso premium futuro, sem exigir refatoração da lógica de estado.

### 2. Orquestração de Lógica (XState)
- **Papel:** Gerenciar fluxos que possuem estados finitos e transições complexas (Combate, Handshake, Processos de Regras).
- **Determinismo:** Garante que, independente da taxa de quadros (30 ou 60 FPS), as regras de jogo sejam aplicadas de forma idêntica e correta em todos os clientes.

### 3. Reconciliação e Fluxo de Dados
- **Ação Local (Optimistic):** O Zustand atualiza a posição do token localmente de forma instantânea.
- **Sincronização:** Os eventos são enviados via WebSocket (ADR-007). Em caso de conflito, a XState Machine coordena o ajuste suave da posição ou o rollback do estado no Zustand.

## Justificativa
- **Eficiência de Recursos:** 30 FPS é perfeitamente adequado para a movimentação de tokens e acompanhamento de narrativa, permitindo que o sistema rode em hardware mais modesto sem drenar recursos desnecessariamente.
- **Monetização e Diferenciação:** Estabelece uma base técnica sólida que permite oferecer maior fidelidade visual (60 FPS) para usuários que optarem por planos premium.
- **Confiabilidade:** O uso de XState remove a ambiguidade de estados complexos (ex: quem é o próximo no turno), garantindo que a narração não seja interrompida por falhas de lógica.

## Consequências
- **Positivas:** Baixo consumo de recursos; UI previsível e estável; base técnica pronta para diferenciação de tiers de serviço.
- **Negativas:** Requer disciplina na implementação de seletores no Zustand para evitar re-renders desnecessários que possam degradar os 30 FPS alvo.

## Referências
- **ADR-007:** Estratégia de Contratos (Protobuf).
- **ADR-011:** Sincronização e Resolução de Conflitos.
- **ADR-018:** Gestão de Cotas de Recursos (Impacto em Tiers).
