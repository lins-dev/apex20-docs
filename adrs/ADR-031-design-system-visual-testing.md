# ADR-031: Documentação de Design System (Storybook) e Testes Visuais via Playwright 🎨🧪

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 exige consistência visual absoluta entre Web e Mobile. Para garantir a integridade do Design System em `packages/ui`, precisamos de um ambiente de desenvolvimento isolado e testes de regressão visual. Optamos pelo uso exclusivo do **Playwright** para manter a stack simplificada e evitar custos com serviços SaaS externos (como Chromatic) nesta fase.

## Decisão
Adotar o **Storybook** para documentação e o **Playwright** como ferramenta única de Testes de Regressão Visual, aplicando processos rigorosos para garantir determinismo e facilidade de revisão.

### 1. Storybook como Single Source of Truth
- **Localização:** `packages/ui`.
- **Uso:** Todo componente deve ter histórias (.stories.tsx) que cubram seus estados visuais. O Storybook servirá como a especificação visual para devs e designers.

### 2. Estratégia de Testes Visuais (Playwright)
Para mitigar as limitações do Playwright, o fluxo de trabalho seguirá estes pilares:

- **Ambiente Determinístico (Docker):** Todos os snapshots visuais serão gerados e validados dentro da imagem Docker oficial do Playwright. Desenvolvedores devem rodar `apex20 test:visual` que orquestra o container localmente para evitar discrepâncias entre OS (macOS/Windows vs Linux).
- **Estabilização Automática:** Os testes utilizarão configurações para desativar animações, esconder cursores e silenciar transições CSS (`prefers-reduced-motion`).
- **Mascaramento:** Elementos com dados dinâmicos (datas, IDs, nomes de usuários) serão mascarados (`mask`) durante o snapshot para evitar falsos positivos.
- **Tolerância Visual (Thresholds):** Configuração de um limite de similaridade (ex: `threshold: 0.2`) para ignorar variações de sub-pixel irrelevantes.

### 3. Workflow de Aprovação (Git-based)
- **CI Report:** O GitHub Actions gerará um relatório HTML comparando o "Expected", "Actual" e o "Diff".
- **Update Automático:** Caso uma mudança visual seja intencional, o desenvolvedor utilizará um comando via CLI (`apex20 test:visual --update-snapshots`) que atualizará as imagens de referência no repositório. O processo de aprovação será feito via Code Review padrão do GitHub.

## Justificativa
- **Custo e Simplicidade:** Utiliza uma ferramenta já presente na stack de testes, reduzindo a complexidade de autenticação e custos de terceiros.
- **Controle Total:** O repositório contém todas as referências visuais, garantindo que o histórico de mudanças da UI esteja rastreável via Git.
- **Confiabilidade:** O uso de Docker elimina o maior problema do Playwright (inconsistência de renderização entre máquinas).

## Consequências
- **Positivas:** UI consistente e documentada; zero custo extra de licenciamento; infraestrutura de testes unificada.
- **Negativas:** Aumento no tamanho do repositório Git devido ao armazenamento de snapshots binários (pode ser mitigado via Git LFS se necessário); processo de aprovação manual via CI é ligeiramente mais lento que ferramentas SaaS.

## Referências
- **ADR-010:** Estratégia de Testes e QA.
- **ADR-027:** Ferramentas de DX e CLI Interna (Comandos de Visual Test).
