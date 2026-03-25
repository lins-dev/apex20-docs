# Roadmap de Desenvolvimento (TASKS.md) 🚀

Este documento detalha as Sprints planejadas para o desenvolvimento do **Apex20**.

## 🏗️ Sprint 0: Definições Arquiteturais (Formalização)
**Objetivo:** Consolidar as decisões críticas de infraestrutura e governança.

- [x] Criar ADR-002: Estratégia de Autenticação Cross-Service (JWT/RS256).
- [x] Criar ADR-003: Padronização de Erros e Respostas (RFC 7807) p/ i18n.
- [x] Criar ADR-004: Escolha de Ferramenta de Migração (Tern).
- [x] Criar ADR-005: Definição do Gerenciador de Pacotes (pnpm Workspaces).
- [x] Criar ADR-007: Estratégia de Contratos e Serialização (Protobuf + ConnectRPC).
- [x] Criar ADR-008: Engine de Fichas Dinâmicas (JSONB vs EAV vs Scripting).
- [x] Criar ADR-009: Estratégia de Observabilidade (OpenTelemetry + Prometheus).
- [x] Criar ADR-010: Estratégia de Pirâmide de Testes e QA (Playwright + Testcontainers).
- [x] Criar ADR-011: Estratégia de Sincronização e Resolução de Conflitos.
- [x] Criar ADR-012: Estratégia de Voz e Vídeo Integrada (WebRTC/LiveKit).
- [x] Criar ADR-013: Padrões de Acessibilidade (a11y) e Design Inclusivo.
- [x] Criar ADR-014: Arquitetura de Plugins e Extensibilidade (Sandboxing).
- [x] Criar ADR-015: Conformidade com Privacidade (LGPD/GDPR) e Segurança.
- [x] Criar ADR-016: Estratégia de Geração de Números Aleatórios (RNG Verificável).
- [x] Criar ADR-017: Persistência de Estado Real-time e Snapshotting da Mesa.
- [x] Criar ADR-018: Gestão de Cotas de Recursos (Storage/IA) e Limites.
- [x] Criar ADR-019: Estratégia de Escalonamento e Service Discovery para WebSockets.
- [x] Criar ADR-020: Interface de Sensores e Estratégia de Fallback para CV/AR.
- [x] Criar ADR-021: Rate Limiting, Throttling e Proteção de WebSocket (Anti-Griefing).
- [x] Criar ADR-022: Estratégia de Zero-Downtime Deployment para Sockets ativos.
- [x] Criar ADR-034: Resiliência de Dados e Consistência Distribuída.
- [x] Criar ADR-023: Versionamento de Assets e Invalidação de Cache (Content Hashing).
- [x] Criar ADR-024: Estratégia de Telemetria e Error Tracking (Client-side).
- [x] Criar ADR-025: Gestão de Estados de Jogo e Máquinas de Estado (XState/Zustand).
- [x] Criar ADR-026: Infraestrutura de Keyboard-First UX e Command Palette.
- [x] Criar ADR-027: Ferramentas de Developer Experience (DX) e CLI Interna.
- [x] Criar ADR-028: Estratégia de Distribuição Global e Minimização de Latência.
- [x] Criar ADR-029: Estratégia de Licenciamento de Conteúdo (OGL/ORC/SRD).
- [x] Criar ADR-030: Orçamento de Performance e Monitoramento de WebVitals.
- [x] Criar ADR-031: Documentação de Design System (Storybook) e Testes Visuais.
- [x] Criar ADR-032: Estratégia de Monetização e Pagamentos (Stripe vs Gateways Locais).
- [x] Criar ADR-033: Automação de Faturamento e Emissão de Notas Fiscais (NF-e).
- [x] Criar ADR-035: Esteira de Integridade e Qualidade (CI/CD).
- [x] Criar ADR-036: Arquitetura Frontend Modular (Feature-based).
- [x] Criar ADR-037: Estratégia de Branching (GitHub Flow Adaptado).
- [x] Validar Fluxo de CI/CD inicial e Docker Registry.

## 🛠️ Sprint 1: Infraestrutura e Contratos (Core)
**Objetivo:** Estabelecer a base dos repositórios independentes (polyrepo) e comunicação entre serviços.
**Detalhamento:** Ver [docs/tasks/sprint-1.md](tasks/sprint-1.md)

- [x] Setup dos repositórios independentes (polyrepo): `apex20-web`, `apex20-backend`, `apex20-ws`.
- [x] Definição de Schemas Protobuf em `apex20-contracts` (Handshake, Chat e GridEvents).
- [x] Implementação do Middleware de Permissões de Grid (ACL: Quem move o quê).
- [x] Configuração do Backend Go (Chi + Arquitetura Hexagonal + sqlc).
- [x] Setup do Docker Compose local (PostgreSQL, Redis, Prometheus/Grafana).
- [x] Implementação do serviço de WebSocket básico com Redis Pub/Sub.
- [x] Boilerplate da aplicação Next.js com shadcn/ui.
- [x] Setup inicial do módulo i18n em `apex20-web/src/i18n/` (EN, PT-BR, ES, FR).

## ⚔️ Sprint 2: Mecânicas de Jogo e Sincronização (🟡 Em Progresso)
**Objetivo:** Implementar o grid e o sistema de combate em tempo real.

- [x] Auth API (SignUp/SignIn via ConnectRPC, Argon2, JWT RS256).
- [x] Auth UI (`modules/auth`: SignInForm, SignUpForm, useAuth, route guards).
- [x] CRUD de Campanhas (criação, listagem, edição, exclusão soft-delete).
- [x] Campaign Members API (convidar/remover jogadores).
- [ ] CRUD de Personagens (Fichas dinâmicas).
- [ ] Sistema de Grid com movimentação de tokens (Optimistic UI).
- [ ] Rolagem de dados e Chat sincronizado via WebSocket.
- [ ] Implementação do Storage via **Cloudflare R2** (Maps, Tokens e Assets).
- [ ] Pipeline de Otimização Automática de Assets (WebP/AVIF).
- [ ] Sistema de internacionalização (i18n) funcional em todos os apps.

## 📱 Sprint 3: Mobile e Experiência do Jogador
**Objetivo:** Lançar o app mobile e refinar a UI.

- [ ] Desenvolvimento do App Mobile (Expo + NativeWind).
- [ ] Integração Mobile com WS-Service para controle remoto de ficha.
- [ ] Implementação de Realidade Aumentada (WebXR) inicial para projeção de grid.
- [ ] Sistema de autenticação (OAuth2/JWT compartilhado entre API e WS).
- [ ] Início da implementação do **Foundry VTT Importer** (JSON mapping).

## 🤖 Sprint 4: Inteligência e Visão
**Objetivo:** Adicionar os recursos avançados de IA e CV.

- [ ] Integração **MediaPipe** para reconhecimento de gestos (Lançar dados).
- [ ] Sistema de **Logging de Eventos** para geração de histórico de sessão.
- [ ] Automação de **Resumos de Sessão via IA** consumindo logs estruturados.
- [ ] Disparo de resumos via Microserviço externo (WhatsApp, Telegram, E-mail).
- [ ] Refinamento da iluminação dinâmica e névoa de guerra (FOW).

## 💻 Sprint 5: Ecossistema Desktop e Polimento
**Objetivo:** Criar a versão desktop e preparar para o lançamento.

- [ ] Empacotamento Desktop: Escolha final entre **Tauri** ou **Electron**.
- [ ] Integração do App Desktop com o código-fonte de `apex20-web`.
- [ ] Refatoração e otimização de performance (Go Profiling e JS bundle size).
- [ ] Documentação completa de API para desenvolvedores externos.
- [ ] Lançamento da versão Alpha/Beta pública.

---

**Legenda de Status:**
- 🔴 Não Iniciado
- 🟡 Em Progresso
- 🟢 Concluído
- ⚙️ Bloqueado
