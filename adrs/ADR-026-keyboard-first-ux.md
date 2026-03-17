# ADR-026: Infraestrutura de Keyboard-First UX e Command Palette ⌨️🚀

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
Virtual Tabletops (VTTs) tradicionais dependem excessivamente de cliques em menus aninhados, o que torna a gestão do jogo lenta para o mestre e interrompe o fluxo narrativo. Para o Apex20 atingir sua meta de performance e imersão, precisamos de uma interface que permita a execução de ações em milissegundos. Um design "Keyboard-First" não apenas melhora a UX para usuários avançados, mas também é um pilar fundamental de acessibilidade (ADR-013).

## Decisão
Implementar uma arquitetura de interface centrada em uma **Command Palette** global e um sistema unificado de atalhos de teclado.

### 1. Command Palette (O Centro de Comando)
- **Atalho Principal:** `Cmd+K` (macOS) ou `Ctrl+K` (Windows/Linux).
- **Funcionalidades:**
    - Busca difusa (fuzzy search) de comandos, personagens, cenas e itens.
    - Execução de macros e rolagens de dados rápidas (ex: digitar `/roll 2d20`).
    - Navegação entre diferentes módulos do app (Grid, Chat, Configurações).
- **Tecnologia:** Utilizar a biblioteca `cmdk` (padrão em componentes shadcn/ui) para garantir performance e suporte nativo a leitores de tela.

### 2. Sistema Global de Atalhos (Hotkeys)
- **Consistência:** Definir um mapa de atalhos padrão (ex: `G` para Grid, `C` para Chat, `I` para Iniciativa).
- **Conscientização:** Implementar dicas visuais de atalhos (tooltips) em todos os botões da UI para educar o usuário progressivamente.
- **Escopo:** Utilizar hooks como `react-hotkeys-hook` para gerenciar o escopo dos atalhos (ex: a tecla `M` move um token se o grid estiver focado, mas digita a letra "m" se o chat estiver ativo).

### 3. Navegação por Foco (Focus Management)
- Garantir que todos os elementos interativos sejam acessíveis via `Tab` com indicadores de foco claros.
- Implementar "Focus Traps" em modais e menus suspensos para manter a navegação lógica e segura via teclado.

## Justificativa
- **Performance de Jogo:** Reduz drasticamente o tempo entre a intenção do mestre e a execução da ação, mantendo o ritmo da sessão.
- **Acessibilidade:** Cumpre requisitos de conformidade (ADR-013) para usuários que não podem utilizar mouse ou dispositivos apontadores.
- **Diferenciação:** Posiciona o Apex20 como uma ferramenta profissional para mestres "power-users", similar a IDEs modernas (VS Code).

## Consequências
- **Positivas:** Fluxo de jogo extremamente rápido; interface limpa (menos botões visíveis necessários); excelente suporte a acessibilidade.
- **Negativas:** Requer cuidado extra no desenvolvimento para evitar conflitos de atalhos entre o navegador e a aplicação; curva de aprendizado inicial para usuários que não estão acostumados com Command Palettes.

## Referências
- **ADR-013:** Padrões de Acessibilidade (a11y).
- **ADR-025:** Gestão de Estados de Jogo (Ações via Máquinas de Estado).
- **ADR-027:** Ferramentas de DX (Experiência similar à CLI interna).
