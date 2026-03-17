# ADR-028: Estratégia de Distribuição Global e Minimização de Latência 🌐⚡

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 é uma plataforma global onde a percepção de latência é crítica para a imersão. A meta estabelecida no PRD é uma latência de sincronização de WebSockets **inferior a 100ms**. Jogadores em diferentes continentes acessando um único servidor central (ex: EUA) sofreriam com atrasos superiores a 200ms, prejudicando a movimentação de tokens e a fluidez do chat.

## Decisão
Adotar uma arquitetura de **Distribuição Multi-regional** com roteamento inteligente na borda (Edge).

### 1. Roteamento Inteligente (Anycast/Geo-DNS)
- Utilizar a infraestrutura da **Cloudflare** para detectar a localização do usuário e rotear o tráfego de WebSocket e API para o Data Center mais próximo geograficamente.
- **Protocolo:** Priorizar conexões via WSS (Secure WebSockets) com suporte a TLS 1.3 para reduzir o overhead de handshake.

### 2. Clusters Regionais (PoPs)
- Implementar clusters independentes do `apps/ws-service` em regiões estratégicas (ex: América do Sul - São Paulo, América do Norte - Virgínia, Europa - Frankfurt).
- Cada cluster regional possui seu próprio **Redis local** para Pub/Sub de alta velocidade (ADR-019), garantindo que mensagens entre jogadores da mesma região não saiam do backbone local.

### 3. Afinidade de Campanha (Room Pinning)
- Para evitar inconsistências e latência de rede inter-continental em uma única sessão, cada Campanha/Sala será vinculada a uma **Região Mestre** (geralmente a região do Mestre da mesa ou a de menor latência para a maioria).
- Todos os jogadores de uma mesma sala se conectam ao cluster daquela região específica.

### 4. Estratégia de Persistência Distribuída
- **Escrita:** Banco de Dados primário (PostgreSQL) centralizado para garantir integridade absoluta das regras de negócio.
- **Leitura:** Utilizar réplicas de leitura (Read Replicas) em regiões distantes para acelerar o carregamento inicial de fichas e assets sem onerar a conexão transatlântica.
- **Assets:** Distribuição global automática via **Cloudflare CDN** para mapas e imagens (ADR-023).

## Justificativa
- **Experiência do Usuário (UX):** A proximidade física reduz o RTT (Round Trip Time), essencial para a meta de < 100ms.
- **Escalabilidade:** Permite o crescimento horizontal por região conforme a base de usuários se expande em diferentes mercados.
- **Resiliência:** Se uma região sofrer instabilidade, usuários de outras regiões não serão afetados.

## Consequências
- **Positivas:** Latência mínima garantida; conformidade com exigências de localidade de dados; performance de elite.
- **Negativas:** Maior custo de infraestrutura operacional; complexidade adicional na gestão de réplicas de banco de dados e roteamento.

## Referências
- **ADR-019:** Escalonamento de WebSockets.
- **ADR-023:** Versionamento de Assets (CDN).
- **ADR-034:** Resiliência de Dados (Localidade).
