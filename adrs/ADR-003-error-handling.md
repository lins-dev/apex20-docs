# ADR-003: Padronização de Erros e Respostas (RFC 7807) p/ i18n 🛠️

**Data:** 2026-03-12
**Status:** Proposto

## Contexto
O Apex20 é um sistema multi-idioma (EN, PT-BR, ES, FR) e distribuído. Sem um padrão de erro estruturado, cada serviço (Backend Go, WS-Service) pode retornar mensagens em formatos diferentes, dificultando a tradução no Frontend (Next.js/Mobile) e a observabilidade. Precisamos de uma forma de desacoplar a "identidade do erro" da "mensagem exibida ao usuário".

## Decisão
Adotar a **RFC 7807 (Problem Details for HTTP APIs)** como o formato mandatório para todas as respostas de erro (4xx e 5xx).

### Especificação da Resposta:
Todas as APIs devem retornar o `Content-Type: application/problem+json` com a seguinte estrutura mínima:

1.  **`type` (String/URI)**: O identificador único do erro (ex: `https://apex20.com/errors/insufficient-funds`). Este será usado como **chave de tradução** no Frontend.
2.  **`title` (String)**: Resumo curto e estático do erro em Inglês (para desenvolvedores).
3.  **`status` (Number)**: O código de status HTTP (ex: 400, 403, 500).
4.  **`detail` (String)**: Explicação detalhada da ocorrência específica (em Inglês).
5.  **`instance` (URI)**: Um identificador de rastreio (Request ID) para correlação em logs.

### Fluxo de i18n:
- O **Backend** envia apenas o `type`.
- O **Frontend** (Web/Mobile) possui um dicionário em `apex20-web/src/i18n/` que mapeia o `type` para a frase traduzida no idioma selecionado pelo usuário.

## Justificativa
- **Desacoplamento**: O backend não precisa conhecer o idioma do usuário para reportar erros.
- **Consistência**: Garante que o App Mobile e a Web exibam a mesma mensagem para o mesmo erro.
- **Tipagem Forte**: Facilita a criação de tipos TypeScript para tratar erros de forma programática.

## Consequências
- **Positivas**: Facilita enormemente a manutenção de traduções; melhora a depuração através do campo `instance`.
- **Negativas**: Pequeno overhead de implementação inicial nos middlewares de erro dos serviços Go.

## Alternativas Consideradas
- **Simple JSON (`{"error": "message"}`)**: Rejeitado por ser ambíguo e dificultar a tradução automática.
- **Tradução no Backend**: Rejeitado pois exigiria que o backend gerenciasse estados de localidade e arquivos de tradução duplicados com o frontend.
