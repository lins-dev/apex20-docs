# ADR-013: Padrões de Acessibilidade (a11y) e Design Inclusivo ♿

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O RPG de mesa é uma atividade social por natureza. Como um Virtual Tabletop (VTT), o Apex20 tem a responsabilidade de não criar barreiras digitais que excluam jogadores com deficiências visuais, motoras, auditivas ou cognitivas. Interfaces complexas, como grids de combate e fichas densas, são historicamente difíceis de tornar acessíveis. Precisamos de um compromisso arquitetural com a acessibilidade (a11y) desde o dia zero.

## Decisão
Adotar os padrões **WCAG 2.1 (Web Content Accessibility Guidelines) no nível AA** como requisito mandatório para o desenvolvimento da interface.

1.  **Semântica e ARIA**:
    - Utilizar HTML semântico rigoroso em todos os componentes.
    - Implementar atributos ARIA (Accessible Rich Internet Applications) em elementos dinâmicos, especialmente em componentes complexos como o *Combat Tracker* e o *Dice Roller*.
2.  **Keyboard-First UX**:
    - Garantir que 100% das funcionalidades (incluindo movimentação de tokens e navegação no mapa) sejam operáveis via teclado.
    - Integração profunda com a **Command Palette (ADR-026)** para atalhos rápidos de ações de jogo.
3.  **Design Visual Inclusivo**:
    - Suporte nativo a modos de **Alto Contraste** e temas específicos para **Daltônicos** (Protanopia, Deuteranopia e Tritanopia).
    - Garantir razões de contraste de cores de no mínimo 4.5:1 para texto normal e 3:1 para texto grande.
4.  **Feedback Auditivo e Visual Duplicado**:
    - Todas as informações críticas (ex: "é o seu turno") devem ser transmitidas por mais de um canal (ex: um som de notificação + um destaque visual + um anúncio para leitor de tela).
5.  **Touch Targets e Espaçamento**:
    - Manter áreas de clique/toque de no mínimo 44x44 pixels para garantir acessibilidade motora em dispositivos mobile e desktop.

## Justificativa
- **Inclusão**: O RPG deve ser para todos. Acessibilidade não é um "recurso", é um direito fundamental do usuário.
- **UX Superior para Todos**: Padrões de acessibilidade (como navegação via teclado e boa hierarquia visual) melhoram a experiência de uso até para usuários sem deficiências.
- **Conformidade Legal**: Atende a legislações internacionais (como a ADA nos EUA e a LBI no Brasil) e aumenta o alcance de mercado do produto.

## Consequências
- **Positivas**: Plataforma inclusiva e socialmente responsável; melhor SEO e estrutura de código; UX refinada e rápida através de atalhos de teclado.
- **Negativas**: Requer esforço adicional de desenvolvimento e testes (uso de ferramentas como Axe-core ou leitores de tela durante o QA); pode impor restrições criativas em designs puramente estéticos que não respeitam contraste ou legibilidade.

## Alternativas Consideradas
- **Acessibilidade como "Nice-to-have"**: Rejeitado, pois implementar a11y retroativamente em sistemas complexos é extremamente caro e ineficiente.
- **Plugins de Acessibilidade de Terceiros**: Rejeitado, pois geralmente são ineficazes e não resolvem problemas de semântica estrutural profunda.
