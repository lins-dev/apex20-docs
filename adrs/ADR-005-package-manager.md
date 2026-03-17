# ADR-005: Definição do Gerenciador de Pacotes (pnpm Workspaces) 📦

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O Apex20 é estruturado como um monorepo que contém múltiplas aplicações (`apps/`) e pacotes compartilhados (`packages/`). Gerenciar dependências entre diferentes linguagens (Node.js/Next.js e Go) e garantir que os pacotes internos (como `contracts`, `ui` e `i18n`) sejam compartilhados de forma eficiente e segura é um desafio crítico de arquitetura.

## Decisão
Adotar o **pnpm** como o gerenciador de pacotes oficial do monorepo, utilizando o recurso de **Workspaces**.

1.  **Estrutura de Workspaces**: Definir a raiz do projeto com um arquivo `pnpm-workspace.yaml` que inclua as pastas `apps/*` e `packages/*`.
2.  **Strict Mode**: Utilizar a arquitetura de `node_modules` não-plana (non-flat) do pnpm para garantir que os sub-projetos só acessem dependências declaradas explicitamente em seus próprios `package.json`.
3.  **Local Packages**: Consumir pacotes internos (ex: `@apex20/ui`) utilizando o protocolo `workspace:*` para garantir que as alterações locais sejam refletidas instantaneamente sem necessidade de publicação.
4.  **Orquestração com Turborepo**: O pnpm será o motor que alimenta o **Turborepo** para o cache de builds e execução de tarefas em paralelo.

## Justificativa
- **Eficiência de Disco**: O uso de Hard Links pelo pnpm economiza gigabytes de espaço em disco ao compartilhar uma única *content-addressable store* para todas as dependências.
- **Velocidade**: O pnpm é comprovadamente mais rápido que o npm e o yarn v1 em operações de instalação e resolução de dependências em monorepos.
- **Segurança e Previsibilidade**: O isolamento estrito de dependências (non-hoisting) previne o uso acidental de pacotes fantasma (phantom dependencies), garantindo builds determinísticos em CI/CD.

## Consequências
- **Positivas**: Builds mais rápidos; menor consumo de recursos; estrutura de monorepo extremamente robusta e organizada.
- **Negativas**: Curva de aprendizado inicial para desenvolvedores acostumados apenas com o npm padrão; necessidade de configurar ferramentas de IDE (como VSCode) para entender a estrutura de links simbólicos em alguns cenários específicos.

## Alternativas Consideradas
- **npm**: Rejeitado pela lentidão e instabilidade ao gerenciar workspaces complexos.
- **yarn (v1/v3)**: Rejeitado pela inconsistência no gerenciamento de links simbólicos e maior overhead de manutenção em comparação ao pnpm nativo.
