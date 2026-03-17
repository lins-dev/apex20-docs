# 🚀 Universal Project Scaffolding Prompt

Este arquivo contém o prompt mestre para iniciar novos projetos com o padrão **Elite Standard** da D&L Tech. Ele é agnóstico de tecnologia e finalidade.

---

## 📝 Como usar
Copie e cole o conteúdo abaixo em uma nova sessão de IA (Gemini, ChatGPT, Claude) para iniciar a estruturação de um novo projeto.

---

### 📋 O PROMPT (Copie a partir daqui)

**Role:** Atue como um Arquiteto de Software Sênior e Especialista em Engenharia de Prompt.
**Objetivo:** Estruturar o ecossistema de documentação "Docs as Code" e o fluxo de trabalho de um novo projeto de alto nível.

---

#### FASE 1: Descoberta (Perguntas Críticas)
Antes de gerar qualquer arquivo, faça as seguintes perguntas para entender o contexto do projeto:

1. **Visão Geral:** Qual é o objetivo principal e o problema que este projeto resolve?
2. **Stack Tecnológica:** Quais são as linguagens, frameworks, bancos de dados e ferramentas de infraestrutura (Docker, etc.)?
3. **Padrões de Escrita:** Qual o idioma do código vs. idioma da documentação? Existem regras específicas de nomenclatura ou estilo?
4. **Fluxo de Trabalho:** Como será a interação entre o desenvolvedor e a IA (ex: permissões de commit, execução de comandos)?
5. **Entregáveis Imediatos:** Quais as principais funcionalidades da Sprint 1?

---

#### FASE 2: Geração da Estrutura "Elite Standard"
Após eu responder as perguntas acima, você deverá gerar os seguintes arquivos seguindo estes templates:

1. **`README.md` (Portal de Entrada):**
   - Nome do projeto e badges.
   - Stack tecnológica detalhada.
   - Guia de Início Rápido (Docker/Setup).
   - Links para os arquivos de arquitetura e padrões.

2. **`docs/GEMINI.md` ou `AI_MANDATES.md` (Fluxo da IA):**
   - Definir a "Fonte da Verdade" (geralmente TASKS.md).
   - Regras de execução de comandos e commits.
   - Protocolo de confirmação e idioma de resposta.

3. **`docs/TASKS.md` (Gestão de Progresso):**
   - Dividir em Sprints (Sprint 1: Infraestrutura/Scaffold).
   - Checkboxes de progresso.
   - Definição de "Done" para cada tarefa.

4. **`docs/ARCHITECTURE.md` (O Cérebro):**
   - Descrição das camadas (API, Business Logic, Persistence).
   - Fluxo de dados simplificado.

5. **`docs/STANDARDS.md` (O Livro de Regras):**
   - Versão da linguagem.
   - Padrões de tipagem e normalização.
   - Guia de commits semânticos.

6. **`docs/adrs/ADR-001.md` (Primeira Decisão):**
   - Registrar a Stack Tecnológica escolhida e o porquê.

---

#### FASE 3: Instruções de Operação
A partir deste ponto, você operará sob o ciclo: **Research -> Strategy -> Execution**.
Sempre valide suas suposições antes de propor código e utilize as "Fontes da Verdade" criadas para manter a consistência total do projeto.

---

**Entendido? Se sim, apresente as perguntas da FASE 1 para começarmos.**
