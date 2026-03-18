# ADR-035: Esteira de Integridade e Qualidade (CI/CD) 🛡️🚀

**Data:** 2026-03-15
**Status:** Aceito

## Contexto
O Apex20 é um ecossistema polyrepo (Go, TypeScript, Protobuf) onde a consistência do código e a integridade dos contratos são vitais. Em um ambiente de desenvolvimento distribuído (Navegador, Mobile, Backend) e heterogêneo (Host OS vs Docker), pequenas derivações de configuração podem levar a quebras silenciosas no pipeline de CI/CD ou à inserção de código fora do padrão **Elite Standard**. Precisamos de uma "Esteira de Proteção" em múltiplas camadas que seja resiliente e transparente.

## Decisão
Adotar uma arquitetura de **Defesa em Profundidade** para a qualidade do código, dividida em três camadas obrigatórias:

### 1. Camada de Pre-commit (Local Enforcement)
- **Ferramentas:** **Husky** + **lint-staged**.
- **Configuração:** O arquivo `.lintstagedrc` reside na raiz de cada repositório.
- **Isolamento de Binários:** O ESLint e o Prettier devem estar instalados como `devDependencies` no `package.json` da raiz. Isso garante que o Husky encontre os executáveis tanto no Host quanto dentro de containers Docker (evitando erros de `ENOENT`).
- **Regras:**
    - `*.{js,ts,tsx}`: Executa `eslint --fix` e `prettier --write`.
    - `*.go`: Executa `golangci-lint run`.
    - `*.proto`: Executa `buf lint` e `buf format`.

### 2. Camada de Nuvem (CI Pipeline)
- **Ferramenta:** GitHub Actions (`.github/workflows/pipeline.yml`).
- **Jobs Críticos:**
    - **Quality & Tests:** Executa em paralelo no CI (jobs independentes por repositório).
    - **Security Audit:** Validação de dependências (`npm audit` no web, `govulncheck` no Go) e segredos.
- **Imutabilidade:** O uso de `npm ci` é mandatório no `apex20-web` para garantir que o ambiente de CI seja idêntico ao de desenvolvimento.

### 3. Camada de Padronização (DX)
- **Regra de Execução:** Todo comando de desenvolvimento (Lint, Build, Test) **deve** ser executável via Docker para garantir que o ambiente seja determinístico.
- **Precedência:** O linter na raiz de cada repositório é soberano para garantir um estilo de código unificado.

## Justificativa
- **Resiliência:** Centralizar os binários de linting na raiz resolve problemas de resolução de caminhos em containers Docker.
- **Velocidade:** O lint-staged reduz o tempo de validação ao processar apenas os arquivos alterados, não o projeto inteiro.
- **Confiança:** Garante que nenhum commit chegue ao repositório sem passar pelos critérios de aceite automáticos.

## Consequências
- **Positivas:** Redução drástica de falhas de CI por erros de linting; histórico de commits limpo e padronizado; facilidade no onboarding de novos desenvolvedores.
- **Negativas:** Leve overhead na instalação inicial de dependências (instalar eslint na raiz e nos pacotes); obrigatoriedade de ter o Docker rodando para commits em ambientes que não possuem Node/Go instalados no host.

## Referências
- **ADR-006:** Padrão de Commits.
- **ADR-010:** Estratégia de Testes.
- **ADR-027:** Ferramentas de DX e CLI Interna.
