# apex20-docs

Fonte única de verdade para toda a documentação do ecossistema **Apex20** — um Virtual Tabletop (VTT) de alta performance para RPG.

Este repositório é consumido como **git submodule** em todos os outros repositórios do projeto.

## Estrutura

```
adrs/          Architectural Decision Records (ADR-001 a ADR-036+)
tasks/         Detalhamento técnico das sprints
wireframes/    Mockups e walkthrough da interface
ARCHITECTURE.md  Visão geral da arquitetura e fluxo de dados
PRD.md           Product Requirements Document
STANDARDS.md     Padrões de código, commits e fluxo de trabalho
TASKS.md         Roadmap de desenvolvimento (sprints)
```

## Atualizar o submodule em outro repositório

```bash
git submodule update --remote docs
git add docs && git commit -m "🚀 chore(docs): sync submodule"
```

## Adicionar como submodule

```bash
git submodule add https://github.com/lins-dev/apex20-docs.git docs
git submodule update --init --recursive
```
