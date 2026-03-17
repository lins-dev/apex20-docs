# ADR-020: Interface de Sensores e Estratégia de Fallback para CV/AR 👁️📐

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
O Apex20 propõe o uso de tecnologias avançadas como **MediaPipe** (para leitura de dados físicos e gestos) e **WebXR/ARCore** (para projeção 3D). No entanto, o ecossistema de dispositivos dos usuários é heterogêneo. Precisamos de uma arquitetura que:
1.  **Abstraia o Hardware:** Uma interface única para lidar com sensores no Web (Next.js) e Mobile (Expo).
2.  **Garante Inclusividade:** O sistema deve ser 100% funcional mesmo sem câmera ou suporte a AR (conforme ADR-013).
3.  **Performance:** Processamento de IA deve ser eficiente para não drenar a bateria de dispositivos móveis.

## Decisão

### 1. Sensor Abstraction Layer (SAL)
Implementar uma camada de abstração em `packages/sensors` que unifica o acesso aos sensores:
- **Web:** Utilizar a API nativa de `getUserMedia` e `WebXR Device API`.
- **Mobile:** Utilizar `expo-sensors` e `expo-camera`.
- **Interface:** Ambos retornarão streams de dados padronizados para os consumidores (ex: coordenadas de gestos ou resultados de rolagens).

### 2. Processamento Client-side (Edge AI)
- Toda a lógica de Visão Computacional (MediaPipe) será executada no **dispositivo do usuário** (Client-side).
- **Justificativa:** Redução de custos de infraestrutura no backend, latência zero para feedback visual e preservação da privacidade (as imagens da câmera nunca saem do dispositivo).

### 3. Estratégia de Fallback (Graceful Degradation)
O sistema operará sob o princípio de **Melhoria Progressiva**:

| Recurso Avançado | Detecção de Falha (Fallback) | Experiência de Fallback |
| :--- | :--- | :--- |
| **Leitura de Dados Físicos (CV)** | Câmera indisponível ou baixa luz. | Rolagem via botões na interface ou comandos de chat. |
| **Gestos de Navegação (CV)** | Sensor desativado ou hardware lento. | Navegação clássica via Mouse (Pan/Zoom) ou Touch. |
| **Miniaturas em AR** | Falta de suporte a WebXR/ARCore. | Visualização em Grid 2D/3D Top-down padrão. |
| **Reconhecimento de Mesa Física** | Falha na detecção de planos. | Calibração manual de escala e orientação do tabuleiro. |

### 4. Gestão de Recursos
- **Throttling:** Se o dispositivo detectar aquecimento excessivo ou bateria < 15%, o sistema reduzirá automaticamente a taxa de quadros (FPS) do processamento de CV ou desativará o AR.

## Justificativa
- **Universalidade:** Permite que o Apex20 seja "Elite" para quem tem hardware de ponta e "Sólido" para quem usa hardware básico.
- **Privacidade:** Ao manter o processamento local, evitamos problemas de conformidade com a LGPD/GDPR relativos a dados biométricos/imagens (ADR-015).

## Consequências
- **Positivas:** Experiência imersiva de última geração; custo operacional de backend reduzido; alta acessibilidade.
- **Negativas:** Maior complexidade no desenvolvimento do frontend para lidar com múltiplos estados de fallback; bundle size inicial maior devido às bibliotecas de CV (resolvido via code-splitting).

## Referências
- **ADR-013:** Acessibilidade.
- **ADR-015:** Privacidade e Segurança.
