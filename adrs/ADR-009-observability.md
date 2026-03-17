# ADR-009: Estratégia de Observabilidade (OpenTelemetry + Prometheus) 📊

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O Apex20 é um sistema distribuído e de alta performance. Problemas de latência no WebSocket, falhas silenciosas na API ou gargalos no banco de dados podem arruinar a experiência de jogo. Precisamos de uma visão holística e em tempo real do que está acontecendo em todo o ecossistema para identificar e resolver problemas rapidamente.

## Decisão
Adotar o ecossistema **OpenTelemetry (OTel)** para rastreamento e **Prometheus** para métricas.

1.  **Distributed Tracing (OpenTelemetry)**: 
    - Instrumentar o `apps/backend` e o `apps/ws-service` para gerar traces de cada requisição/mensagem.
    - Propagar o `TraceID` do Frontend até a camada de persistência.
2.  **Metrics (Prometheus)**: 
    - Exportar métricas técnicas (CPU, Memória, Latência HTTP/gRPC).
    - Criar métricas de negócio (sessões de jogo ativas, número de rolagens de dados por segundo, latência de broadcast no WebSocket).
3.  **Structured Logging**: 
    - Utilizar logs estruturados em JSON (Go Zap ou Slog).
    - Incluir obrigatoriamente o `TraceID` nos logs para permitir a correlação entre um log de erro e o rastro completo da requisição.
4.  **Visualização (Grafana)**: 
    - Centralizar a visualização de métricas e traces em dashboards do Grafana.
5.  **Integração Client-side (Sentry/PostHog)**:
    - Correlacionar erros do frontend capturados pelo **Sentry** com os Traces do backend via `TraceID`.
    - Utilizar o **PostHog** para observar a jornada do usuário e identificar onde gargalos de performance (vistos no Grafana) afetam a experiência real.

## Justificativa
- **Vendor Agnostic**: O OpenTelemetry permite trocar o backend de observabilidade (ex: mudar de Jaeger para Honeycomb ou Datadog) sem alterar o código da aplicação.
- **Depuração Precisa**: O rastreamento distribuído permite ver exatamente em qual microservço uma requisição falhou ou ficou lenta.
- **Escalabilidade**: Prometheus é o padrão da indústria para monitoramento de sistemas em containers (Docker/Kubernetes).

## Consequências
- **Positivas**: Redução drástica no Tempo Médio de Reparo (MTTR); visibilidade total da performance real do usuário.
- **Negativas**: Pequeno overhead de performance devido à coleta de dados (pode ser mitigado via amostragem/sampling); maior complexidade no setup inicial da infraestrutura local (Docker Compose).

## Alternativas Consideradas
- **Logs Simples (STDOUT)**: Rejeitado por ser insuficiente para sistemas distribuídos e de tempo real.
- **Soluções Proprietárias (New Relic/Datadog)**: Rejeitado nesta fase para evitar custos elevados e lock-in de fornecedor.

## Referências
- **ADR-024:** Telemetria e Error Tracking (Client-side).
