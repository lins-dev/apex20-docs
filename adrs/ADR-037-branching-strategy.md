# ADR-037: Estratégia de Branching (GitHub Flow Adaptado) 🌿

**Data:** 2026-03-18
**Status:** Aceito

## Contexto

O Apex20 é um ecossistema polyrepo com múltiplos serviços. Sem uma estratégia de branching bem definida, o risco de código não validado atingir produção aumenta consideravelmente, especialmente em um projeto com uma única pessoa de manutenção onde a disciplina de processo substitui a revisão por pares.

O `STANDARDS.md` menciona o GitHub Flow de forma resumida. Este ADR formaliza o fluxo completo, incluindo a integração com a esteira de CI/CD (ADR-035) e os hooks de pre-commit (Husky, ADR-006).

## Decisão

Adotar o **GitHub Flow Adaptado** com duas branches de longa duração (`dev` e `master`) e branches de curta duração por demanda.

### Branches de Longa Duração

| Branch | Papel | Proteção |
|---|---|---|
| `master` | Código em produção — sempre estável e deployável | Push direto bloqueado; merge apenas via PR da `dev` |
| `dev` | Integração contínua — ponto de partida e destino de todas as features | Push direto bloqueado; merge via PR de feature branches |

### Nomenclatura de Branches de Feature

```
<type>/<short-description>
```

Onde `<type>` segue os mesmos tipos do padrão de commits (ADR-006):

| Type | Uso |
|---|---|
| `feat/` | Nova funcionalidade (ex: `feat/grid-token-movement`) |
| `fix/` | Correção de bug (ex: `fix/ws-reconnect-loop`) |
| `refactor/` | Refatoração sem mudança de comportamento |
| `test/` | Adição ou correção de testes |
| `chore/` | Tarefas de manutenção (deps, configs) |
| `docs/` | Alterações exclusivas em documentação |

### Fluxo Completo por Demanda

```
dev
 └─► feat/nova-funcionalidade   (branch criada a partir de dev)
       │
       ├─ desenvolvimento local
       ├─ testes unitários / visuais (ADR-010, ADR-031)
       ├─ commit → Husky (pre-commit: lint + format) (ADR-035)
       ├─ push origin feat/nova-funcionalidade
       │
       └─► Pull Request → dev
             │
             ├─ CI Pipeline (GitHub Actions) (ADR-035)
             │    ├─ lint + typecheck
             │    ├─ testes unitários
             │    └─ testes visuais (Playwright)
             │
             └─ [CI verde] → Merge para dev (squash ou merge commit)

dev  ──────────────────────────────────────────────────────────►
 └─► Pull Request → master   (promoção de release)
       │
       ├─ CI Pipeline completo (revalidação)
       └─ [aprovado] → Merge para master → Deploy
```

### Regras Obrigatórias

1. **Nunca trabalhar diretamente na `dev` ou `master`** — toda mudança parte de uma branch de feature.
2. **Branch sempre criada a partir da `dev` atualizada:**
   ```bash
   git checkout dev && git pull origin dev
   git checkout -b feat/minha-feature
   ```
3. **Commit só é aceito após Husky aprovar** (lint, format, testes via lint-staged).
4. **PR para `dev` requer CI verde** — nenhum merge com pipeline com falha.
5. **`master` só recebe merge via PR da `dev`** — promoções manuais de branches de feature para `master` são proibidas.
6. **Branches de feature são deletadas após o merge** para manter o repositório limpo.
7. **Autoria dos commits:** os commits devem refletir exclusivamente a autoria humana do desenvolvedor responsável. Ferramentas de assistência (ex: IAs, copilotos) **não devem figurar como autores ou co-autores** (`Co-Authored-By`) em nenhuma mensagem de commit.

### Integração com CI/CD (ADR-035)

- **Push em feature branch:** Dispara pipeline de qualidade (lint, typecheck, testes).
- **PR aberto para `dev`:** Pipeline completo obrigatório para habilitar o merge.
- **Merge em `dev`:** Dispara pipeline de integração e, opcionalmente, deploy em ambiente de staging.
- **Merge em `master`:** Dispara pipeline de produção (build, deploy, smoke tests).

## Justificativa

- **Simplicidade:** GitHub Flow é o modelo mais simples que ainda oferece proteção de `master`. Adequado para time pequeno.
- **Rastreabilidade:** Cada feature branch corresponde a uma demanda do sprint, facilitando o histórico via `git log`.
- **Integração com o processo existente:** Complementa ADR-006 (commits), ADR-010 (testes), ADR-031 (visual tests) e ADR-035 (CI/CD) sem introduzir overhead.

## Consequências

- **Positivas:** `master` sempre estável; histórico linear e rastreável; CI obrigatório antes de qualquer integração.
- **Negativas:** Exige disciplina de não fazer push direto nas branches protegidas; pequeno overhead de PR mesmo para mudanças pequenas.

## Referências

- **ADR-006:** Padrão de Commits (nomenclatura de tipos).
- **ADR-010:** Estratégia de Testes (critério de aceite antes do merge).
- **ADR-031:** Testes Visuais (Playwright, parte do CI).
- **ADR-035:** Esteira de CI/CD (pipeline de qualidade).
