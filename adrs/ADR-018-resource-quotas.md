# ADR-018: Gestão de Cotas de Recursos (Storage/IA) e Limites 📈

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O Apex20 utiliza recursos que geram custos variáveis e diretos para a infraestrutura, como o armazenamento de arquivos no Cloudflare R2, o tráfego de saída (egress), o processamento de modelos de IA para resumos de sessão e a manutenção de conexões WebSocket persistentes. Sem uma gestão rigorosa de cotas, o projeto fica vulnerável a abusos, custos imprevistos e degradação de performance para outros usuários.

## Decisão
Implementar um sistema centralizado de **Gestão de Cotas e Limites** baseado em níveis de assinatura (Tiers).

1.  **Níveis de Assinatura (Quota Tiers)**:
    - **Free**: Limites básicos de armazenamento (ex: 100MB) e jogadores simultâneos por mesa (ex: 4).
    - **Pro/Master**: Limites expandidos ou ilimitados de armazenamento e acesso a recursos premium (IA, AR, alta fidelidade).
2.  **Gestão de Armazenamento (Cloudflare R2)**:
    - O `apex20-backend` deve validar o espaço disponível do usuário antes de gerar URLs pré-assinadas para upload.
    - Implementar um serviço de contagem de bytes que atualiza o uso total do usuário no PostgreSQL após cada upload/deleção bem-sucedida.
3.  **Créditos de IA e Rate Limiting**:
    - O uso de IA (Session Summaries) será controlado por um sistema de créditos mensais ou limites diários rígidos para evitar custos explosivos com APIs de terceiros.
4.  **Limites de Conexão e Performance WebSocket**:
    - O `apex20-ws` deve consultar o plano do Mestre da mesa no momento do handshake (ADR-002) e aplicar os limites de conexão e **taxa de quadros (FPS)**.
    - **Cota de FPS:** Conforme **ADR-025**, o padrão de entrega é **30 FPS**, com o desbloqueio para **60 FPS** reservado a planos premium.
5.  **Enforcement Híbrido**:
    - **Redis**: Utilizado para contadores rápidos e temporários (ex: rate limit de mensagens no chat, conexões ativas).
    - **PostgreSQL**: Fonte da verdade para limites persistentes e de longo prazo (ex: GBs de armazenamento).

## Justificativa
- **Sustentabilidade Econômica**: Garante que o custo de cada usuário seja proporcional ao seu plano, evitando que usuários gratuitos gerem prejuízo financeiro direto.
- **Proteção de Infraestrutura**: Evita ataques de "Resource Exhaustion", onde um usuário malicioso tenta preencher o disco ou saturar os servidores de WebSocket.
- **Fair Use**: Garante que a performance do sistema seja consistente para todos, impedindo que uma única campanha "monopolize" os recursos do servidor.

## Consequências
- **Positivas**: Previsibilidade de custos de infraestrutura; monetização facilitada através de tiers de recursos; segurança aprimorada contra abusos.
- **Negativas**: Introduz latência mínima para validação de cotas em operações de escrita; complexidade adicional no desenvolvimento para lidar com estados de "Cota Esgotada" na interface do usuário (UI/UX).

## Alternativas Consideradas
- **Pay-as-you-go Puro**: Rejeitado pela dificuldade de planejamento financeiro para o usuário final.
- **Sem Limites (Tamanho Único)**: Rejeitado por ser insustentável em um modelo SaaS gratuito/freemium.

## Referências
- **ADR-023:** Versionamento de Assets (Deduplicação).
- **ADR-025:** Gestão de Estados de Jogo (30/60 FPS).
