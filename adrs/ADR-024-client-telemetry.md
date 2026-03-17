# ADR-024: Estratégia de Telemetria e Error Tracking (Client-side) 📊📡

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 exige alta confiabilidade e performance em dispositivos heterogêneos (Navegadores e Mobile). Erros de runtime, vazamentos de memória ou latência de rede no lado do cliente impactam diretamente a imersão do jogo. Precisamos de ferramentas que capturem esses eventos de forma automática e correlacionada, permitindo uma depuração proativa e baseada em dados reais de uso.

## Decisão
Adotar um ecossistema de observabilidade de frontend composto por três pilares fundamentais:

### 1. Rastreamento de Erros (Sentry)
- **Ferramenta:** **Sentry** (SDK oficial para Next.js e Expo).
- **Finalidade:** Captura automática de exceções, erros de rede e crashes nativos em dispositivos móveis.
- **Diferencial:** Uso de *Source Maps* para visualização do código original (TypeScript) em vez de código minificado e *Breadcrumbs* para entender as ações que precederam o erro.

### 2. Telemetria de Performance (OpenTelemetry)
- **Ferramenta:** **OpenTelemetry (OTel) Web SDK**.
- **Finalidade:** Implementar o *Distributed Tracing* (Rastreamento Distribuído). O cliente injetará um `TraceID` nas requisições (ConnectRPC), permitindo correlacionar a latência percebida na UI com o processamento no backend e banco de dados.
- **Métricas:** Monitoramento de *Web Vitals* (LCP, FID) e latência de mensagens WebSocket (RTT).

### 3. Análise de Comportamento e Produto (PostHog)
- **Ferramenta:** **PostHog** (Cloud ou Self-hosted).
- **Finalidade:** *Session Replay* (Gravações de Sessão) para visualizar o exato momento em que um bug ocorreu e *Product Analytics* para medir o engajamento com recursos específicos (ex: frequência de uso da Command Palette).
- **Feature Flags:** Utilizar o PostHog para liberação gradual de funcionalidades e testes A/B.

## Justificativa
- **Visibilidade 360°:** A combinação de Sentry (Erros), OTel (Performance) e PostHog (Comportamento) cobre todas as necessidades técnicas e de produto da plataforma.
- **Depuração Acelerada:** O *Session Replay* reduz drasticamente o tempo gasto tentando reproduzir bugs reportados por usuários.
- **Interoperabilidade:** O uso do padrão aberto OpenTelemetry garante que o Apex20 não sofra de *vendor lock-in* para dados de performance, integrando-se nativamente com o Grafana (ADR-009).

## Consequências
- **Positivas:** Redução do MTTR (Mean Time To Repair); melhoria na qualidade percebida pelo usuário; decisões de produto baseadas em dados.
- **Negativas:** Overhead de processamento no dispositivo do usuário (mitigado via *Sampling*); custo de infraestrutura e assinaturas de SaaS; necessidade de configurar CSP (Content Security Policy) para permitir o tráfego de telemetria.

## Referências
- **ADR-009:** Estratégia de Observabilidade (Backend).
- **ADR-015:** Privacidade e Segurança (Data Scrubbing).
- **ADR-025:** Gestão de Estados de Jogo (Performance).
- **ADR-028:** Distribuição Global e Latência.
