# ADR-015: Conformidade com Privacidade (LGPD/GDPR) e Segurança 🔒

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O Apex20 gerencia dados de usuários (nomes, e-mails, logs de chat) e ativos de criação (mapas, textos de campanhas) que estão sujeitos a leis de proteção de dados como a **LGPD (Brasil)** e **GDPR (União Europeia)**. Além da conformidade legal, a segurança e a privacidade são fundamentais para a confiança dos mestres de RPG ao confiarem suas criações autorais à nossa plataforma.

## Decisão
Adotar o princípio de **Privacy by Design** e estabelecer camadas rigorosas de proteção de dados.

1.  **Minimização de Dados (Data Minimization)**: 
    - Coletar apenas o estritamente necessário para o funcionamento do serviço (ex: e-mail e nome/pseudônimo).
    - Evitar a coleta de dados sensíveis desnecessários (ex: documentos de identificação) a menos que exigido por processadores de pagamento (Stripe/NF-e).
2.  **Criptografia e Proteção de Dados**:
    - **Em Trânsito**: Uso mandatório de TLS 1.3+ para todas as conexões (HTTPS e WSS).
    - **Em Repouso (At Rest)**: Criptografia de todos os volumes de banco de dados (PostgreSQL) e buckets de armazenamento (Cloudflare R2).
    - **Hashing de Senhas**: Utilizar algoritmos modernos (ex: Argon2 ou BCrypt com salt) conforme padrão da indústria.
3.  **Direitos do Titular (LGPD/GDPR)**:
    - Implementar funcionalidade de **Exclusão de Conta (Direito ao Esquecimento)**, garantindo a remoção de todos os dados pessoais, mantendo apenas o necessário para conformidade fiscal.
    - Implementar funcionalidade de **Exportação de Dados (Portabilidade)** em formato JSON legível por máquina para fichas e campanhas.
4.  **Gestão de Sessão Segura**:
    - Utilizar JWTs (conforme ADR-002) com tempo de expiração curto. O token carrega `sub` e `is_admin`; a role de campanha é resolvida dinamicamente via `campaign_members`, nunca persistida no token.
    - Implementar Blacklisting de tokens revogados no Redis para garantir encerramento imediato de sessões comprometidas.

## Estratégia de Segurança (Anti-Backdoor e Proteção)
- **Sanitização Estrita**: Validação e sanitização rigorosa de todas as entradas de usuário (schemas Protobuf e sqlc) para prevenir SQL Injection e XSS.
- **Isolamento de Tenant**: Garantir que as consultas ao banco de dados sempre incluam filtros de `user_id` ou `campaign_id` para evitar que um jogador acesse dados de outra mesa sem permissão.
- **WAF (Web Application Firewall)**: Utilizar o firewall da Cloudflare para mitigar ataques DDoS e proteger as rotas de API contra explorações comuns.

## Justificativa
- **Confiança do Usuário**: Protege a propriedade intelectual dos mestres e a privacidade dos jogadores.
- **Redução de Riscos**: Minimiza o impacto de possíveis vazamentos de dados através da minimização e criptografia.
- **Conformidade Legal**: Evita sanções administrativas e multas pesadas decorrentes de descumprimento de leis de privacidade.

## Consequências
- **Positivas**: Plataforma segura, ética e legalmente resiliente; vantagem competitiva sobre sistemas que não respeitam a privacidade.
- **Negativas**: Maior overhead de desenvolvimento para gerenciar fluxos de exportação e exclusão de dados; latência mínima introduzida por camadas de criptografia e WAF.

## Alternativas Consideradas
- **Segurança Reativa**: Rejeitado, pois lidar com segurança apenas após incidentes é inaceitável para um produto de alta disponibilidade.
- **Hospedagem em Servidores Próprios (Sem Cloudflare)**: Rejeitado pela complexidade e custo de gerenciar mitigação de DDoS e conformidade física de data center.

## Referências
- **ADR-008:** Engine de Fichas (WASM).
- **ADR-014:** Arquitetura de Plugins (Iframes).
