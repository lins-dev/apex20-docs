# ADR-011: Estratégia de Sincronização e Resolução de Conflitos 🔄

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
Em um Virtual Tabletop (VTT), a percepção de latência é o maior inimigo da imersão. Se um jogador move um token e ele demora 200ms para responder, a experiência parece "pesada". Ao mesmo tempo, múltiplos jogadores podem tentar interagir com o mesmo objeto simultaneamente, o que exige uma estratégia clara de sincronização para evitar estados inconsistentes (ex: um token em duas posições diferentes).

## Decisão
Adotar uma estratégia híbrida de **Optimistic UI** com **Autoridade do Servidor** e **Resolução de Conflitos baseada em LWW (Last Write Wins)**.

1.  **Optimistic UI + Reconciliation**:
    - O Cliente (Web/Mobile) aplica a alteração localmente de forma instantânea.
    - O Servidor (`ws-service`) valida a ação. Se for inválida (ex: movimento proibido), o servidor envia um comando de `REJECT` e o cliente realiza o **Rollback** suave do estado.
2.  **Autoridade do Servidor**:
    - O Servidor é o juiz final para todas as regras de negócio (permissões, fórmulas de dados, névoa de guerra).
3.  **Resolução de Conflitos (LWW)**:
    - Para atualizações de estado simples (posição de tokens, HP, status), a última mensagem processada pelo servidor (baseada no timestamp de chegada) é a vencedora.
4.  **Soft Locking (Prevenção Visual)**:
    - Quando um jogador inicia o arraste de um token, um evento de `interaction_start` é enviado. Outros jogadores recebem um sinal visual (ex: borda colorida) indicando que o objeto está em uso, desencorajando conflitos.
5.  **Deltas e Snapshots**:
    - **Deltas**: Enviar apenas os campos alterados (ex: `{ x: 10 }`) via Protobuf para minimizar o tráfego.
    - **Snapshots**: O estado atual da mesa é mantido em cache (Redis) pelo `ws-service` para permitir sincronização instantânea de novos jogadores (Initial Sync).

## Justificativa
- **Experiência do Usuário (UX)**: A Optimistic UI elimina a sensação de lag de rede, fazendo o sistema parecer local.
- **Simplicidade e Performance**: LWW é computacionalmente barato e suficiente para a maioria das interações de um VTT, evitando a complexidade extrema de CRDTs (Conflict-free Replicated Data Types) onde não são necessários.
- **Eficiência de Rede**: O uso de Deltas reduz drasticamente o payload das mensagens de WebSocket.

## Consequências
- **Positivas**: Interface extremamente responsiva; baixo overhead de processamento no servidor de WebSockets.
- **Negativas**: Requer lógica complexa no frontend para gerenciar estados "pendentes" e possíveis rollbacks; possibilidade teórica de "teletransporte" visual de tokens se a latência for muito alta e houver conflito real.

## Alternativas Consideradas
- **Locking Estrito (Hard Lock)**: Rejeitado por prejudicar a fluidez (exigiria esperar o servidor confirmar o clique antes de permitir o arraste).
- **CRDTs (Conflict-free Replicated Data Types)**: Rejeitado pela complexidade de implementação e overhead de memória para este estágio do projeto (pode ser reconsiderado para edição colaborativa de notas de texto longo).

## Referências
- **ADR-007:** Estratégia de Contratos (Protobuf).
