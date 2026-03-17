# ADR-008: Engine de Fichas Dinâmicas (Sandboxed JS via WASM) 🎲

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
Para garantir a longevidade do Apex20, precisamos que a criação de fichas seja acessível a desenvolvedores e membros da comunidade, similar ao modelo de sucesso do Foundry VTT e Roll20 (HTML/JS). No entanto, como somos uma plataforma local-first/cloud, permitir a execução de JavaScript arbitrário dos usuários representa um risco crítico de segurança (XSS, roubo de sessão e injeção de código).

## Decisão
Adotar uma arquitetura de **Sandboxed JavaScript utilizando WebAssembly (WASM)** para a lógica e **Templates Reativos** para a UI.

1.  **Lógica (JavaScript Seguro)**: 
    - Utilizar um motor JavaScript ultra-leve (como o **QuickJS**) compilado para **WASM**.
    - O código do usuário (lógica de cálculos, bônus, perícias) roda dentro deste motor isolado.
    - **Ponte de Dados**: O motor recebe um objeto JSON (dados da ficha), executa a lógica e retorna o JSON atualizado. Ele não tem acesso global a `window`, `document`, `fetch` ou `cookies`.
2.  **UI (Templates Dinâmicos)**:
    - Utilizar um sistema de templates baseado em componentes (ex: subconjunto de React ou Handlebars com componentes shadcn/ui).
    - O layout da ficha é definido em um formato declarativo que impede a inserção de tags `<script>` maliciosas no HTML final.

## Estratégia de Segurança e Anti-Backdoor
- **Execução Desconectada**: O ambiente WASM é instanciado sem nenhuma capacidade de IO (entrada/saída) por padrão. O script não pode "falar" com a rede nem com o disco.
- **Memória Isolada**: O script do usuário vive em um bloco de memória RAM separado e limitado, impossibilitando ataques de estouro de pilha (buffer overflow) contra o processo principal.
- **Validação de Saída**: O Core do Apex20 valida se o JSON retornado pelo script contém campos proibidos ou mudanças não autorizadas em metadados do personagem.
- **Time-outs**: Execução limitada a poucos milissegundos para evitar scripts de mineração de cripto ou ataques de negação de serviço (DoS).

## Justificativa
- **Experiência do Desenvolvedor (DX)**: Mantém a curva de aprendizado baixa, pois utiliza a linguagem mais popular da web (JavaScript).
- **Segurança Superior**: Diferente do Foundry (que confia no JS do usuário), o Apex20 isola o código em uma "jaula" WASM, eliminando riscos de XSS e roubo de credenciais.
- **Portabilidade**: A mesma engine WASM roda identicamente no Backend (Go), no Web (Next.js) e no Mobile (Expo/React Native).

## Consequências
- **Positivas**: Flexibilidade extrema para sistemas de RPG; comunidade familiarizada com a stack; segurança de nível empresarial.
- **Negativas**: Leve overhead de performance ao transitar dados entre o JS principal e o Sandbox WASM; necessidade de manter o motor QuickJS-WASM atualizado.

## Referências
- **ADR-014:** Arquitetura de Plugins (Iframes).
- **ADR-015:** Privacidade e Segurança.
