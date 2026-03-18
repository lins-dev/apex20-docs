# ADR-005: Estratégia de Repositórios e Gestão de Dependências (Polyrepo + Git Submodules) 📦

**Data:** 2026-03-12
**Status:** Supersedido — revisado em 2026-03-17 (ver MIGRATION_PLAN.md)

## Contexto
O Apex20 originalmente foi estruturado como um monorepo com pnpm Workspaces e Turborepo. Com o crescimento do projeto, a complexidade de build (Next.js canary + Turbopack, eslint compartilhado, tsconfig encadeados) passou a consumir mais tempo de manutenção do que o desenvolvimento do produto em si, especialmente para um único mantenedor.

## Decisão
Migrar de pnpm Workspaces/Turborepo para **repositórios Git independentes (polyrepo)**, compartilhando código via **Git Submodules**.

### Estrutura de Repositórios
| Repositório | Stack | Gerenciador |
|---|---|---|
| `apex20-docs` | Markdown | — |
| `apex20-contracts` | Protobuf / Buf CLI | — |
| `apex20-web` | Next.js 16, TypeScript | npm |
| `apex20-backend` | Go | Go modules |
| `apex20-ws` | Go | Go modules |

### Compartilhamento de Código
- **`apex20-docs`** e **`apex20-contracts`** são consumidos como **git submodules** em `./docs/` e `./contracts/` de cada repositório.
- O código TypeScript gerado dos contratos é consumido via path alias no `tsconfig.json`: `@contracts/* → ./contracts/gen/ts/*`.
- O código Go gerado é consumido via `replace` no `go.mod`: `replace github.com/apex20/contracts => ./contracts/gen/go`.

### Pacotes Internalizados
- `packages/ui` → internalizado em `apex20-web/src/ui/` (design system).
- `packages/i18n` → internalizado em `apex20-web/src/i18n/` (módulo i18n sem dependência externa).

## Justificativa
- **Simplicidade:** Cada repositório tem seu próprio ciclo de vida, tooling e CI/CD sem interferência cruzada.
- **Velocidade:** Sem overhead de resolução de workspace; `npm install` e `go mod download` são operações diretas.
- **Manutenibilidade:** Um único mantenedor consegue operar cada repositório de forma independente e focada.

## Consequências
- **Positivas:** Setup drasticamente mais simples; cada repo é autocontido; sem conflitos de versão entre apps.
- **Negativas:** Mudanças em contratos exigem atualização explícita do submodule em cada repositório consumidor; sem cache de build compartilhado entre repositórios.

## Workflow de Atualização de Submodule
```bash
# Após alterar apex20-contracts:
git submodule update --remote contracts
git add contracts && git commit -m "🚀 chore(contracts): sync submodule"
```

## Referências
- **MIGRATION_PLAN.md:** Detalhamento completo do processo de migração.
- **ADR-007:** Estratégia de Contratos (Protobuf + Buf).
