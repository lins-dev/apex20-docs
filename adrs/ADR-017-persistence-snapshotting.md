# ADR-017: Persistência de Estado Real-time e Snapshotting da Mesa 💾

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
Em um VTT, o estado da mesa (posições de tokens, status de HP, iluminação, névoa de guerra) muda constantemente. Persistir cada pequeno movimento do mouse diretamente no banco de dados relacional (PostgreSQL) geraria um overhead imenso de IO e latência. Por outro lado, manter o estado apenas na memória do servidor WebSocket é arriscado (perda de dados em caso de crash). Precisamos de uma estratégia que garanta fluidez em tempo real e durabilidade dos dados.

## Decisão
Adotar uma estratégia de **Write-Behind Caching** utilizando o **Redis** como "Fonte da Verdade" em tempo real e o **PostgreSQL** para persistência de longo prazo (Snapshots).

1.  **Hot State (Redis)**:
    - O estado ativo de uma mesa (Grid State) é armazenado no Redis enquanto houver jogadores conectados.
    - Atualizações via WebSocket (ADR-011) modificam o Redis instantaneamente com latência sub-milissegundo.
2.  **Snapshotting Periódico**:
    - O `ws-service` realiza o "Snapshot" do estado do Redis para o PostgreSQL em intervalos regulares (ex: a cada 2 minutos) ou após um período de inatividade da mesa.
    - O snapshot é salvo como um objeto **JSONB** na tabela de campanhas/cenas, permitindo que o `sqlc` recupere o estado completo na próxima carga.
3.  **Persistência de Fechamento (On-Exit)**:
    - Quando o último jogador se desconecta de uma mesa, o sistema aguarda um breve "grace period" (ex: 30s) e realiza a persistência final obrigatória no PostgreSQL antes de limpar o cache do Redis.
4.  **Log de Eventos Críticos (Event Sourcing Light)**:
    - Ações que alteram regras de negócio permanentemente (ex: gastar recursos, causar dano, rolar dados) são enviadas imediatamente para o `apps/backend` para persistência síncrona, garantindo que o histórico de combate e chat nunca seja perdido.

## Justificativa
- **Performance de Grid**: Permite milhares de atualizações de posição por segundo sem estressar o banco de dados principal.
- **Resiliência**: O Redis (configurado com AOF/RDB) protege contra quedas do serviço de WebSocket, permitindo que o estado seja recuperado rapidamente.
- **Eficiência de Escrita**: Agrupar múltiplas atualizações visuais em um único Snapshot reduz drasticamente o número de transações no PostgreSQL.

## Consequências
- **Positivas**: Movimentação de tokens fluida e sem lag de persistência; escalabilidade horizontal do serviço de WebSocket facilitada pelo Redis compartilhado.
- **Negativas**: Pequeno risco de perda de dados visuais (ex: posição exata de um token) entre o último snapshot e um crash catastrófico do Redis (minimizado pela persistência on-exit e logs críticos síncronos).

## Alternativas Consideradas
- **Direct-to-DB**: Rejeitado pela baixa performance e alto custo de escrita para dados altamente voláteis (movimento de mouse).
- **In-Memory Only (WS-Service)**: Rejeitado pelo risco inaceitável de perda de dados e dificuldade de escalonamento (sticky sessions seriam obrigatórias).
