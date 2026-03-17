# ADR-034: Resiliência de Dados e Consistência Distribuída 🛡️

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
O Apex20 é um sistema de alta performance projetado para ser **Local-first** e escalável horizontalmente (ADR-019). Para garantir a integridade dos dados em um ambiente distribuído (múltiplos nós/VPS) e facilitar a migração futura de Docker Compose para K3s, precisamos de uma estratégia rigorosa de identificação, isolamento e persistência.

## Decisão

### 1. Identificadores Globais (PKs) via UUIDv7
- **Padrão:** Todas as chaves primárias (PK) e estrangeiras (FK) do sistema utilizarão o padrão **UUIDv7 (RFC 9562)**.
- **Justificativa:** 
    - **Ordenação por Tempo:** Mantém a performance de escrita no PostgreSQL (evita fragmentação de índices B-tree).
    - **Offline-ready:** Permite que clientes Mobile/Web gerem IDs válidos sem conexão com o servidor.
    - **Privacidade:** IDs não sequenciais evitam a exposição do volume total de dados via URL.
    - **Sharding-ready:** Garante unicidade global, permitindo a união ou divisão de bancos de dados sem conflitos de chave.

### 2. Isolamento por Tenant (Campaign-scoped Data)
- **Regra:** Todas as tabelas que contenham dados dinâmicos de jogo (Chat, Tokens, Fichas, Logs) devem possuir obrigatoriamente a coluna `campaign_id`.
- **Finalidade:** 
    - **Localidade de Dados:** Garante que todos os dados de uma mesma campanha possam ser movidos ou "sharded" para servidores diferentes de forma atômica.
    - **Segurança:** Facilita a implementação de Row Level Security (RLS) no PostgreSQL, garantindo que um jogador nunca acesse dados de outra mesa.

### 3. Infraestrutura de Resiliência Distribuída
- **Pooling de Conexões (PgBouncer):** Uso mandatório de PgBouncer em modo `transaction` para gerenciar o escalonamento de milhares de conexões das instâncias de WS-Service sem exaurir o PostgreSQL.
- **Locks Distribuídos (Redlock):** Utilizar o Redis para coordenar operações críticas (como Snapshotting da mesa da memória para o disco) evitando que múltiplas instâncias de WS escrevam simultaneamente na mesma campanha.
- **Idempotência:** Todas as mutações de banco devem ser idempotentes (UPSERT) baseadas no UUIDv7 para prevenir duplicação em caso de re-tentativas de rede (retries).

## Justificativa
- **Arquitetura "Future-proof":** O uso de UUIDv7 e Tenant-isolation permite que o projeto cresça de um único VPS (Docker Compose) para um cluster global (K3s/Cloud) sem refatoração de esquema.
- **Performance Local-first:** A capacidade de gerar IDs no cliente reduz o "perceived lag" do usuário, permitindo que a interface responda instantaneamente antes mesmo da confirmação do servidor.

## Consequências
- **Positivas:** Sistema extremamente resiliente e pronto para escala global; facilidade de auditoria e depuração; conformidade com padrões modernos da indústria.
- **Negativas:** Armazenamento de UUIDs consome 16 bytes (vs 8 bytes de um BigInt), gerando um aumento marginal no tamanho do banco de dados (compensado pela eficiência de índice).

## Referências
- **ADR-004:** Migrações (Uso de Tern e UUIDv7).
- **ADR-017:** Persistência de Snapshotting.
- **ADR-019:** Escalonamento de WebSockets.
