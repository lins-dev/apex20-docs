# Mandatos da IA (GEMINI.md)

Este documento estabelece as diretrizes fundamentais para qualquer agente de IA (como Gemini CLI) operando neste repositório.

## 📋 Regras de Ouro

1.  **Linguagem:**
    *   **Código:** Deve ser escrito exclusivamente em **Inglês** (comentários de código, nomes de variáveis, funções, etc.).
    *   **Documentação e Diálogo:** Deve ser escrito em **Português do Brasil (PT-BR)**, mantendo termos técnicos consagrados em Inglês.

2.  **Segurança e Integridade:**
    *   Prioridade absoluta em segurança (proteção de segredos, validação rigorosa de entrada).
    *   NUNCA expor segredos ou chaves de API em logs ou commits.

3.  **Qualidade Técnica:**
    *   **Type-safety:** Todo o código deve ser fortemente tipado. Utilize Protobuf para contratos e sqlc para interações com o banco de dados.
    *   **Arquitetura:** Siga rigorosamente a **Arquitetura Hexagonal** no backend (Go Chi) e padrões de **Local-first** no frontend.

4.  **Estilo de Código:**
    *   Siga as convenções de estilo definidas em `STANDARDS.md`.
    *   Commits devem seguir o padrão **Semantic Commit + Gitmoji** (ADR-006).

5.  **Contexto e Eficiência:**
    *   Analise o contexto do monorepo antes de propor mudanças.
    *   Mantenha a consistência entre os pacotes do Turborepo.

6.  **Foco na Demanda:**
    *   A IA deve realizar **estritamente o que for solicitado** pelo usuário.
    *   Não inicie novas tarefas, implementações ou criação de arquivos sem um comando explícito.

## 🛠 Ferramentas Preferenciais

*   **Backend:** Go 1.22+, Chi, sqlc, Protobuf.
*   **Frontend:** Next.js (App Router), TailwindCSS, shadcn/ui.
*   **Mobile:** Expo, NativeWind.
*   **Gestão:** Turborepo, pnpm.

## 🛡 Validação Obrigatória

Antes de finalizar qualquer tarefa, a IA deve:
1.  Verificar se o código compila e passa nos testes (se houver).
2.  Garantir que os tipos estão corretos em todo o fluxo de dados (API <-> Frontend).
3.  Validar se a documentação necessária foi atualizada.
