# ADR-002: Estratégia de Autenticação Cross-Service (JWT/RS256) 🔐

**Data:** 2026-03-12
**Status:** Proposto

## Contexto
O Apex20 é composto por múltiplos serviços (`apex20-backend`, `apex20-ws`) que precisam validar a identidade do usuário de forma consistente e performática. Em um ambiente de alta concorrência, como o serviço de WebSockets, consultar o banco de dados ou um serviço central de sessão para cada mensagem/conexão é inviável.

## Decisão
Adotar **JWT (JSON Web Tokens)** utilizando o algoritmo de assinatura **RS256 (RSA Signature com SHA-256)**.

1.  **Assinatura Assimétrica**: 
    - O serviço de Autenticação (ou Identity Provider) possui a **Chave Privada** para gerar os tokens.
    - Todos os outros serviços possuem apenas a **Chave Pública** para validar a autenticidade do token.
2.  **Claims Padrão**: O token deve conter `sub` (User ID), `exp` (Expiração), `iat` (Emitido em) e `role` (Permissões).
3.  **Distribuição de Chaves**: Implementar um endpoint `.well-known/jwks.json` no serviço de autenticação para rotação automática de chaves (futuro).

## Justificativa

- **Performance**: A validação RS256 é feita em memória pelos serviços satélites, sem chamadas de rede ou IO de banco.
- **Segurança**: Mesmo que a chave pública de um serviço (como o WS) seja exposta, o atacante não pode gerar novos tokens válidos.
- **Escalabilidade**: Facilita o escalonamento horizontal, pois os serviços são *stateless* em relação à sessão.

## Consequências
- **Positivas**: Desacoplamento total entre os serviços para validação de identidade; facilidade de integração com Mobile (Expo) e Web (Next.js).
- **Negativas**: Tokens não podem ser facilmente revogados antes da expiração (requer Blacklist em Redis se a revogação imediata for crítica); payload do token aumenta o overhead de cada requisição (manter o JWT enxuto).

## Alternativas Consideradas
- **HS256 (Symmetric)**: Rejeitado porque exige que todos os serviços compartilhem o mesmo segredo, aumentando o risco de vazamento.
- **Opaque Tokens**: Rejeitado pelo custo de latência de consultar um store central (Redis/DB) em cada validação.
