# ADR-014: Arquitetura de Plugins e Extensibilidade (Sandboxing) 🔌

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O ecossistema de um VTT depende fortemente de extensões da comunidade (novas ferramentas de desenho, integração com apps de música, automações de grid). No entanto, permitir a execução de código de terceiros em um ambiente SaaS (Software as a Service) cria riscos críticos de segurança (roubo de tokens de sessão, XSS, exfiltração de dados). Precisamos de uma arquitetura que permita extensibilidade sem comprometer a integridade do Core.

## Decisão
Adotar uma arquitetura de **Sandboxed Plugins via Iframe Isolado** com um **SDK de Comunicação Estruturada**.

1.  **Isolamento Visual (Guest Iframes)**: 
    - Cada plugin será renderizado dentro de um `<iframe>` com o atributo `sandbox="allow-scripts"`.
    - Os iframes serão carregados de uma origem distinta (ex: `guest.apex20.app`) para ativar a **Same-Origin Policy**, impedindo o acesso ao `localStorage`, `cookies` ou `DOM` da aplicação principal (Host).
2.  **Manifesto de Permissões**: 
    - Todo plugin deve declarar um `manifest.json` com as permissões solicitadas (ex: `chat:write`, `grid:read`). O Mestre da mesa deve aprovar essas permissões explicitamente.
3.  **Host-Guest SDK (A Ponte)**:
    - O plugin não acessa APIs globais. Ele interage com o Host exclusivamente via `window.postMessage`, encapsulado em um SDK oficial (@apex20/plugin-sdk).
    - **Documentação Viva**: O SDK será definido via TypeScript/Protobuf, e a documentação será gerada automaticamente a partir do código-fonte para garantir 100% de fidelidade.
4.  **Estilização e Tematização**:
    - O Host injetará variáveis CSS (Tokens de Design) no iframe do plugin para garantir consistência visual com o tema (shadcn/ui) do Apex20.

## Estratégia de Segurança (Anti-Backdoor)
- **Null Origin Enforcement**: O Host validará a origem de cada mensagem recebida. Mensagens de origens não registradas ou sem permissão para a ação solicitada serão descartadas e logadas como incidentes de segurança.
- **Rate Limiting**: O volume de mensagens que um plugin pode enviar ao Host será limitado para evitar ataques de negação de serviço (DoS) na interface do usuário.
- **Content Security Policy (CSP)**: O Host aplicará políticas estritas que impedem que plugins carreguem scripts externos não autorizados ou façam conexões de rede não declaradas.

## Justificativa
- **Segurança de Nível Bancário**: Protege os dados sensíveis dos usuários contra plugins maliciosos ou vulneráveis.
- **Estabilidade**: Um erro fatal no código de um plugin não derruba a aplicação principal.
- **Clareza para Desenvolvedores**: O uso de um SDK tipado e documentação autogerada reduz a barreira de entrada e erros de integração.

## Consequências
- **Positivas**: Ecossistema seguro e escalável; facilidade de manutenção de APIs de terceiros; confiança do usuário final ao instalar extensões.
- **Negativas**: Leve overhead de memória por cada iframe aberto; latência mínima na comunicação via `postMessage`.

## Alternativas Consideradas
- **Execução Direta (Estilo Foundry)**: Rejeitado por ser inseguro em ambiente Cloud/SaaS.
- **Web Workers + Shadow DOM**: Rejeitado pois o isolamento de dados e origem é inferior ao oferecido por iframes sandboxed.

## Referências
- **ADR-008:** Engine de Fichas (WASM).
- **ADR-015:** Privacidade e Segurança.
