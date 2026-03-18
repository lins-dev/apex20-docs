# Padrões e Convenções (STANDARDS.md) 📏

Este documento define as diretrizes de desenvolvimento para garantir a consistência e qualidade do código no **Apex20**.

## 🖋️ Convenções de Código

### 1. Linguagem
- **Código:** Todo o código (variáveis, funções, classes, comentários) deve ser em **Inglês**.
- **Docs:** Documentação técnica externa e logs amigáveis ao usuário devem ser em **Português do Brasil (PT-BR)**.

### 2. Nomenclatura (Naming)
- **Go:** PascalCase para tipos exportados, camelCase para variáveis internas e funções.
- **TypeScript/React:** PascalCase para Componentes e Interfaces, camelCase para funções e constantes.
- **CSS (Tailwind):** Siga a ordem padrão de classes recomendada pelo plugin oficial do Tailwind.

## 🛠️ Linting & Formatação

- **Go:** `gofmt` e `golangci-lint`.
- **TypeScript:** `eslint` com configurações estritas e `prettier`.
- **Automático:** Utilize as extensões recomendadas no VSCode e hooks de pré-commit (husky/lint-staged).

## 🚀 Padrão de Commits (ADR-006)

Seguimos o padrão **Semantic Commit** enriquecido com **Gitmoji**.

### Estrutura:
`<emoji> <type>(<scope>): <description>`

### Tipos Comuns:
- `feat`: Uma nova funcionalidade.
- `fix`: Correção de bug.
- `docs`: Alterações na documentação.
- `style`: Mudanças que não afetam o significado do código (espaço em branco, formatação, etc).
- `refactor`: Alteração de código que não corrige um bug nem adiciona uma funcionalidade.
- `test`: Adição de testes ausentes ou correção de testes existentes.
- `chore`: Atualização de tarefas de build, pacotes, etc.

### Exemplos:
- ✨ `feat(auth): add google oauth2 provider`
- 🐛 `fix(grid): resolve token overlap on zoom`
- 📝 `docs(arch): update data flow diagram`
- 🚀 `chore(deps): update all npm packages`

## 🛠️ Stack Tecnológica (Versões Oficiais)

Para garantir o padrão **Elite Standard**, o projeto utiliza as seguintes versões:
- **Node.js:** v24 (Latest LTS).
- **Go:** v1.26.
- **npm:** v10+ (gerenciador de pacotes do `apex20-web`).
- **PostgreSQL:** v16.
- **Redis:** v7.

## 🏗️ Fluxo de Trabalho (GitHub Flow Adaptado)

1.  Crie uma branch a partir da `dev` (ex: `feat/new-dice-roller`).
2.  Desenvolva seguindo o **TDD Rigoroso** (ADR-010).
3.  Abra um Pull Request (PR) para a branch `dev`.
4.  Após a aprovação e passagem da esteira de CI (`dev`), realize o Merge.
5.  A branch `master` é atualizada apenas via promoção da branch `dev` (PR `dev` -> `master`).

## 🛡️ Segurança

- Jamais commite arquivos `.env` ou segredos.
- Valide todas as entradas de usuário no Backend utilizando schemas rigorosos.
- Utilize HTTPS e conexões seguras de WebSocket (WSS) em produção.
