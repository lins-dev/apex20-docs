# Product Requirements Document (PRD) - Apex20 📝

## 1. Visão Geral do Produto

**Apex20** é um Virtual Tabletop (VTT) focado em **alta performance**, **imersão tecnológica** e **experiência do usuário (UX)**. O objetivo é remover as barreiras digitais que muitas vezes dificultam o ritmo das sessões de RPG, oferecendo uma plataforma fluida tanto para jogadores presenciais quanto remotos.

## 2. Público-Alvo

- Mestres de RPG que buscam automação e imersão.
- Jogadores que apreciam visuais ricos e interação em tempo real.
- Grupos que jogam presencialmente utilizando TV/Projetor (foco em AR e Mobile).

## 3. Requisitos de Produto

### 3.1. Funcionalidades Core (VTT)

- **Grid Sincronizado:** Mapas dinâmicos com Fog of War e iluminação dinâmica.
- **Gerenciamento de Personagens:** Fichas integradas com suporte a múltiplos sistemas.
- **Combat Tracker:** Automação de turnos, efeitos e danos.
- **Chat & Dice:** Rolagens de dados 3D sincronizadas com suporte a fórmulas complexas.
- **Foundry VTT Importer:** Importação total de cenas, atores e itens de mundos Foundry.

### 3.2. Visão Computacional (CV) - MediaPipe

- **Dice Reading:** Leitura automática de dados físicos via câmera.
- **Gestos de Navegação:** Zoom e pan no mapa através de gestos (ideal para mestres).
- **Detecção de Miniaturas:** Sincronização de posição de miniaturas físicas no mapa digital.

### 3.3. Realidade Aumentada (AR) - WebXR

- **Visualização 3D:** Projeção de miniaturas e elementos de cenário em 3D sobre o tabuleiro físico ou digital.
- **Imersão Mobile:** Uso do smartphone/tablet para "espiar" o tabuleiro em AR.

### 3.4. IA & Automação

- **Session Summaries:** Geração de resumos de sessões a partir dos logs de chat e combate.
- **NPC Generator:** Criação dinâmica de NPCs com artes e fichas geradas via IA.

### 3.5. Internacionalização (i18n)

- Suporte completo para **EN, PT-BR, ES, FR**.

## 4. Requisitos Não Funcionais

- **Performance:** Latência de sincronização de WebSockets < 100ms.
- **Escalabilidade:** Suporte a 10.000 usuários simultâneos no serviço de WS.
- **Local-first:** Funcionamento off-line básico com sincronização posterior.
- **Design:** UI moderna baseada em shadcn/ui com suporte a temas Dark/Light.

## 5. Estratégia de Armazenamento

- Imagens de mapas e assets de usuários armazenados no **Cloudflare R2** para baixa latência e custo reduzido de saída de dados.

## 6. Métrica de Sucesso

- Redução de 30% no tempo médio de rodada em combate comparado ao Roll20.
- Taxa de sucesso de importação do Foundry VTT > 90%.
