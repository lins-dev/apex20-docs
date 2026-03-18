# ADR-019: Estratégia de Escalonamento e Service Discovery para WebSockets 🌐

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
O Apex20 exige que o serviço de WebSockets (`apex20-ws`) suporte milhares de conexões simultâneas com latência mínima (PRD: < 100ms). Como as conexões WebSocket são persistentes e mantêm estado em memória (quais usuários estão em quais salas), o escalonamento horizontal apresenta desafios:
1.  **Roteamento Inter-instância:** Jogadores da mesma sala podem estar conectados em servidores físicos diferentes.
2.  **Service Discovery:** Como o cliente sabe para qual réplica apontar.
3.  **Balanceamento de Carga:** Distribuir as conexões de forma justa sem quebrar o handshake do protocolo.

## Decisão
Adotar uma arquitetura de **Escalonamento Horizontal Stateless** utilizando **Redis Pub/Sub** como barramento de mensagens inter-instância.

1.  **Redis Pub/Sub como Backbone**: 
    - Cada instância do `ws-service` se inscreve em canais do Redis baseados no `campaign_id`.
    - Quando um evento ocorre em uma instância (ex: movimento de token), ele é publicado no Redis e todas as outras instâncias que possuem jogadores daquela campanha conectados recebem e retransmitem o evento localmente.
2.  **Load Balancing com Sticky Sessions (Handshake)**:
    - Utilizar um Load Balancer (Nginx/Caddy/Envoy) configurado com **Sticky Sessions** (via Cookies) apenas durante a fase de Upgrade (HTTP -> WS). Isso garante que o handshake ocorra na mesma instância que iniciou a requisição.
3.  **Monitoramento de Densidade**:
    - Cada réplica deve expor métricas de `active_connections` (Prometheus/ADR-009) para que o orquestrador (Docker/K8s) possa decidir o escalonamento.
4.  **Local-first Discovery**:
    - O cliente tentará se conectar ao cluster geograficamente mais próximo (ADR-028) para minimizar o RTT (Round Trip Time).

## Justificativa
- **Desacoplamento**: As instâncias de WS não precisam conhecer umas às outras, apenas o Redis.
- **Performance**: O Redis Pub/Sub possui latência sub-milissegundo, o que mantém o broadcast dentro do orçamento de 100ms.
- **Custo-Benefício**: Evita a complexidade de soluções como Mesh de WebSockets customizado ou protocolos de gossip.

## Consequências
- **Positivas**: Escalabilidade horizontal virtualmente ilimitada; resiliência (uma queda de instância afeta apenas uma parcela dos usuários que reconectarão automaticamente).
- **Negativas**: Pequeno overhead de serialização/deserialização ao passar mensagens pelo Redis; risco de "Race Conditions" em escritas distribuídas (mitigado pela estratégia de Resiliência da **ADR-034**).

## Alternativas Consideradas
- **Nats.io**: Excelente, mas introduziria mais uma peça de infraestrutura. O Redis já é requisito.
- **Distribuição Direta via gRPC**: Complexo de gerenciar as rotas de cada conexão ativa dinamicamente.

## Referências
- **ADR-002:** Autenticação JWT (RS256).
- **ADR-007:** Estratégia de Contratos (Protobuf).
- **ADR-034:** Resiliência de Dados.
