# ADR-030: Orçamento de Performance e Monitoramento de WebVitals ⚡⏱️

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
A performance é uma funcionalidade central do Apex20. Com a meta de **30 FPS** estável (ADR-025) e latência **< 100ms** (ADR-028), não podemos permitir que o acúmulo de funcionalidades degrade a experiência do usuário. Sem orçamentos de performance (Performance Budgets) definidos e monitorados, o custo computacional e o tamanho do bundle tendem a crescer, afetando principalmente usuários em dispositivos móveis ou com hardware modesto.

## Decisão
Estabelecer limites mandatórios de performance e integrar o monitoramento de Web Vitals ao ciclo de vida de desenvolvimento e produção.

### 1. Orçamentos de Carregamento (Web Vitals)
- **LCP (Largest Contentful Paint):** < 2.5s (Tempo para o mapa ou conteúdo principal estar visível).
- **CLS (Cumulative Layout Shift):** < 0.1 (Garantir que a UI não "pule" durante o carregamento de assets).
- **INP (Interaction to Next Paint):** < 200ms (Essencial para a percepção de fluidez ao clicar em tokens ou abrir a Command Palette).

### 2. Orçamentos de Execução e Bundle
- **Taxa de Quadros (FPS):** Mínimo de **30 FPS** estável em dispositivos de entrada. A lógica de renderização do grid deve ser otimizada para não exceder 33ms por quadro.
- **JS Bundle Size:** Máximo de **250KB (Gzip)** para o bundle inicial. Funcionalidades pesadas (MediaPipe, Three.js, LiveKit) devem ser carregadas via *Code Splitting* apenas quando necessário.
- **Asset Loading:** Imagens de mapas devem ser servidas via WebP/AVIF com resoluções adaptativas para não travar a thread principal (ADR-023).

### 3. Monitoramento e Enforcement
- **CI/CD (Lighthouse CI):** Todo Pull Request será testado automaticamente. Se os orçamentos de performance forem excedidos (ex: bundle cresceu 20%), o build falhará.
- **Real User Monitoring (RUM):** Utilizar a biblioteca `web-vitals` integrada à nossa telemetria (ADR-024) para coletar dados reais de performance dos usuários em produção.
- **Hardware Throttling:** Realizar testes locais simulando CPUs lentas e redes 3G para garantir que o "piso" de performance seja respeitado.

## Justificativa
- **Inclusividade:** Garante que o Apex20 seja acessível para jogadores que não possuem hardware de ponta.
- **Sustentabilidade:** Evita a "obesidade de software", mantendo a aplicação rápida e eficiente a longo prazo.
- **Retenção de Usuários:** Há uma correlação direta entre baixos tempos de carregamento e o engajamento dos usuários em plataformas digitais.

## Consequências
- **Positivas:** Experiência de uso sempre fluida; redução de bounce rate; otimização de custos de banda e processamento.
- **Negativas:** Exige maior esforço de engenharia para otimizar componentes; pode limitar a adoção imediata de bibliotecas externas pesadas sem avaliação prévia.

## Referências
- **ADR-023:** Versionamento de Assets (Otimização).
- **ADR-024:** Telemetria e Error Tracking (RUM).
- **ADR-025:** Gestão de Estados de Jogo (Foco em 30 FPS).
- **ADR-028:** Distribuição Global e Latência.
