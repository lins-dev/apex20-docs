# ADR-006: Padrão de Commits Semânticos + Gitmoji 🚀

**Data:** 2023-10-27
**Status:** Aceito

## Contexto
Em um monorepo com múltiplos serviços e desenvolvedores, o histórico de commits pode se tornar caótico rapidamente. Precisamos de um padrão que facilite a leitura do histórico, automatize a geração de changelogs e proporcione uma identidade visual clara para as mudanças.

## Decisão
Adotar o padrão **Semantic Commits** (baseado no Conventional Commits) enriquecido com **Gitmoji**.

### Formato Obrigatório:
`<emoji> <type>(<scope>): <description>`

### Tabela de Emojis Recomendados:
- ✨ `feat` - Nova funcionalidade.
- 🐛 `fix` - Correção de bug.
- 📝 `docs` - Documentação.
- 🚀 `chore` - Tarefas gerais (deps, config).
- 🎨 `style` - Formatação/Estilo.
- ♻️ `refactor` - Refatoração de código.
- ✅ `test` - Testes.
- ⚡️ `perf` - Performance.

## Justificativa

- **Semântica:** O uso de tipos (`feat`, `fix`, etc) permite que ferramentas automatizadas (como Semantic Release) gerenciem versões e changelogs de forma autônoma.
- **Gitmoji:** O uso de emojis facilita a identificação visual rápida de cada tipo de mudança no histórico do Git ou em interfaces como o GitHub.
- **Escopo:** O uso de escopos (`auth`, `api`, `web`) ajuda a identificar rapidamente qual parte do monorepo foi alterada.

## Consequências
- **Positivas:** Histórico de commits extremamente organizado, facilidade em gerar changelogs, melhor comunicação visual entre os membros da equipe.
- **Negativas:** Exige um pequeno esforço adicional na hora de escrever os commits. Requer que os desenvolvedores tenham acesso a uma lista de emojis ou ferramentas auxiliares (como `gitmoji-cli`).

## Exemplos
- ✨ `feat(ws): implement redis pub/sub integration`
- 🐛 `fix(web): correct dice animation glitch`
- 📝 `docs(adr): create commit standard ADR`
