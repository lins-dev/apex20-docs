# ADR-022: Estratégia de Zero-Downtime Deployment para Sockets ativos 🔄🔌

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
Diferente de APIs REST, as conexões WebSocket são persistentes e de longa duração. Ao realizar o deploy de uma nova versão do `ws-service`, o reinício das instâncias (containers) desconectaria milhares de usuários simultaneamente, gerando:
1.  **Impacto na UX:** Interrupção abrupta no meio de um combate ou narração crítica.
2.  **Thundering Herd:** Uma avalanche de tentativas de reconexão instantânea no novo cluster, podendo causar saturação de CPU e rede.
3.  **Perda de Imersão:** Exibição de alertas de erro ou "recarregar página".

## Decisão

### 1. Graceful Shutdown (Drenagem de Conexões)
Implementar um ciclo de vida de terminação suave para o serviço de WebSocket:
- **SIGTERM Handling:** Ao receber um sinal de desligamento do orquestrador (Docker/K3s), a instância entra em modo de "Drenagem" (Drain).
- **Interrupção de Handshakes:** A instância para de aceitar novas conexões imediatamente (os Load Balancers e o roteamento regional por Geo-DNS conforme **ADR-028** redirecionarão novas conexões para as novas instâncias).
- **Grace Period (Janela de Drenagem):** As conexões ativas são mantidas por um período determinado (ex: 5 a 10 minutos) antes do encerramento forçado.

### 2. Reconnection Hinting (Reconexão Escalonada)
Para evitar o efeito de "Manada", a instância em drenagem enviará mensagens de sistema via Protobuf para os clientes:
- **Evento `SERVER_RECONNECT_HINT`:** O servidor solicita que o cliente se desconecte e reconecte voluntariamente.
- **Jittering:** As mensagens são enviadas em lotes aleatórios ao longo da janela de drenagem para distribuir o tráfego de reconexão de forma linear no novo cluster.

### 3. Sessão Persistente (Redis State)
Graças à **ADR-034** (Resiliência de Dados), o estado do grid e do chat reside no Redis. Quando um cliente se reconecta a uma nova instância:
- A nova instância recupera o estado atual do `campaign_id` do Redis instantaneamente.
- O usuário retoma o jogo exatamente de onde parou sem necessidade de novo handshake de autenticação pesado (reuso do JWT válido).

### 4. Client-side Resilience (Reconexão Transparente)
O SDK do frontend (Web/Mobile) deve tratar desconexões como eventos "normais":
- **Exponential Backoff:** Tentativas de reconexão com intervalos crescentes.
- **Optimistic UI Buffer:** Manter as ações do usuário em uma fila local durante a breve desconexão e dispará-las assim que o socket for restabelecido.

## Justificativa
- **Experiência Ininterrupta:** Permite atualizações contínuas de segurança e funcionalidades sem "janelas de manutenção" que frustram os jogadores.
- **Estabilidade de Infraestrutura:** O escalonamento das reconexões protege o banco de dados e o Redis contra picos artificiais de carga.

## Consequências
- **Positivas:** Deployments totalmente transparentes; aumento na confiabilidade percebida do sistema.
- **Negativas:** Deploys tornam-se mais lentos (precisam aguardar o `grace_period` das instâncias antigas); o orquestrador (K3s) precisa de mais recursos (CPU/RAM) temporários para rodar a versão Antiga e a Nova simultaneamente durante o rollout.

## Referências
- **ADR-007:** Estratégia de Contratos (Protobuf).
- **ADR-019:** Escalonamento de WebSockets.
- **ADR-028:** Distribuição Global e Latência.
- **ADR-034:** Resiliência de Dados.
