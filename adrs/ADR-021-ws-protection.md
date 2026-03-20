# ADR-021: Rate Limiting, Throttling e Proteção de WebSocket (Anti-Griefing) 🛡️🔌

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
O serviço de WebSockets (`ws-service`) é o componente mais vulnerável do Apex20 a ataques de negação de serviço (DoS) e abusos de usuários ("Griefing"). Como o sistema é **Local-first** e focado em tempo real, comportamentos como spam de mensagens de chat, movimentação frenética de tokens ou rolagens de dados infinitas podem degradar a performance do cluster e aumentar os custos de infraestrutura.

## Decisão

### 1. Camadas de Proteção (Multi-tier Defense)
Implementar uma estratégia de defesa em profundidade utilizando o **Redis** como o motor de contagem distribuída:

| Camada | Mecanismo | Ação |
| :--- | :--- | :--- |
| **Conexão (Handshake)** | Limite de conexões por IP. | Rejeitar `HTTP 429` se ultrapassar 5 conexões/min por IP. |
| **Payload (L3)** | Limite de tamanho de mensagem. | Cortar a conexão se o frame exceder 32KB (binário Protobuf). |
| **Taxa Global (L7)** | Limite de mensagens por socket. | Drop de mensagens se exceder 20 msgs/seg (ajustável por plano). |
| **Lógica de Jogo** | Throttling semântico. | Limitar ações específicas (ex: 1 rolagem de dado a cada 500ms). |

### 2. Throttling de Movimentação (Grid Events)
Para eventos de alta frequência (movimento de tokens), o servidor aplicará um **Server-side Throttling**:
- Se um cliente enviar 60 eventos de posição por segundo (movimento fluido do mouse), o servidor processará apenas 20-30 para broadcast global, utilizando interpolação no cliente receptor para manter a fluidez.
- Isso reduz a carga no barramento Redis Pub/Sub em até 50%.

### 3. Sistema de "Cooldown" e Reputação
- Usuários que atingirem repetidamente os limites de rate limit entrarão em um estado de **Cooldown** (bloqueio temporário de 60 segundos).
- Incidentes recorrentes serão logados para auditoria do mestre da campanha ou banimento automático do sistema.

### 4. Validação de Autoridade (Anti-Griefing)
- **Permissões de Escrita:** O `ws-service` validará em tempo real se o `user_id` tem permissão para mover o `token_id` solicitado. A role do usuário é resolvida via `campaign_members` (usando o `campaign_id` da sala) e cruzada com `role_permissions` — conforme ADR-002 e ADR-034.
- **Sanitização:** Bloqueio de injeção de scripts em mensagens de chat (UTF-8 validation + HTML escaping no frontend).

## Justificativa
- **Estabilidade do Cluster:** O limite de mensagens evita que o Redis Pub/Sub sofra com "Explosão de Mensagens" (Fan-out excessivo).
- **Equidade:** Garante que o uso abusivo de um jogador não prejudique a latência dos demais.
- **Segurança de Custo:** Protege contra picos de tráfego de saída (egress) que gerariam cobranças inesperadas no Cloudflare/Cloud Provider.

## Consequências
- **Positivas:** Sistema resiliente a ataques coordenados e abusos de usuários; previsibilidade de custos de infraestrutura.
- **Negativas:** Usuários com conexões instáveis ou scripts legítimos de automação de terceiros podem ser bloqueados se não seguirem as janelas de rate limit (necessário fornecer SDK de plugins com throttling embutido, ADR-014).

## Referências
- **ADR-002:** Autenticação JWT (RS256).
- **ADR-007:** Estratégia de Contratos (Protobuf).
- **ADR-014:** Arquitetura de Plugins.
- **ADR-018:** Gestão de Cotas de Recursos.
- **ADR-019:** Escalonamento de WebSockets.
- **ADR-034:** Resiliência de Dados.
