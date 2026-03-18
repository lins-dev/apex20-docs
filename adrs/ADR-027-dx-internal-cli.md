# ADR-027: Ferramentas de Developer Experience (DX) e CLI Interna 🛠️🚀

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 é um ecossistema polyrepo com múltiplas linguagens (Go, TypeScript), ferramentas de geração de código (sqlc, Protobuf, ConnectRPC) e serviços de infraestrutura (PostgreSQL, Redis). Sem ferramentas de automação robustas, o tempo de setup e o risco de inconsistências aumentam. Tradicionalmente, utiliza-se Makefiles para essas tarefas, porém eles apresentam limitações em ambientes Windows e dificuldade em lidar com lógicas de automação mais complexas (scaffolding dinâmico).

## Decisão
Criar um ecossistema de **Developer Experience (DX)** centralizado em uma CLI interna, priorizando-a sobre Makefiles complexos.

### 1. CLI Interna (`apex20`) em Go
- **Mestre de Automação:** A CLI escrita em Go será a ferramenta oficial de interação do desenvolvedor.
- **Superioridade ao Makefile:** 
    - **Cross-platform:** Execução idêntica em Windows, Linux e Mac sem dependências de shell específicas.
    - **Lógica Avançada:** Suporte a templates complexos, interatividade (prompts) e validações de ambiente que seriam frágeis em scripts shell.
    - **Manutenibilidade:** Código Go tipado e testável em vez de scripts Makefile densos.
- **Funcionalidades:** `dev`, `gen` (sqlc/proto/tern), `new` (scaffolding), `db` (migrations), `test` (Unit/Integration com Testcontainers).

### 2. Orquestração entre Repositórios
- A CLI `apex20` orquestra comandos que precisam atuar sobre múltiplos repositórios do polyrepo (ex: atualizar submodules, rodar geração de código nos contratos).
- Cada repositório individual utiliza seu próprio gerenciador (`npm` para o web, `go` para backend/ws).

### 3. Makefile Minimalista (Apenas como Ponte)
- O projeto poderá manter um `Makefile` extremamente simples apenas para atalhos de bootstrapping inicial ou comandos básicos em ambientes de CI/CD que exijam o padrão `make`.
- **Regra:** Toda lógica complexa deve residir na CLI, nunca no Makefile.

### 4. Automação de Qualidade (Git Hooks)
- Uso de **Husky** e **lint-staged** para garantir conformidade de estilo (Prettier/Gofmt) e linting antes de cada commit.

## Justificativa
- **Portabilidade:** Garante que qualquer desenvolvedor, independente do OS, tenha a mesma experiência.
- **Poder de Automação:** A CLI em Go permite automatizar a criação de serviços inteiros (Arquitetura Hexagonal boilerplate) com segurança de tipos.
- **Padronização:** Reduz a carga cognitiva ao centralizar todos os comandos em uma única ferramenta `apex20`.

## Consequências
- **Positivas:** Fluxo de trabalho de "Elite"; redução drástica de erros de setup; facilidade em escalar o ecossistema polyrepo.
- **Negativas:** Exige manutenção inicial da CLI (código Go extra); os desenvolvedores precisam ter o binário da CLI instalado ou disponível via `go run`.

## Referências
- **ADR-001:** Escolha da Stack Tecnológica.
- **ADR-006:** Padrão de Commits.
- **ADR-007:** Estratégia de Contratos (Geração de Código).
- **ADR-010:** Estratégia de Testes (TDD e Testcontainers).
