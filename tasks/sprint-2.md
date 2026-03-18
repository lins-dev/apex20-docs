# Detalhamento Técnico: Sprint 2 ⚔️

**Objetivo:** Inicializar a aplicação Web, definir a identidade visual base e implementar o grid interativo com sincronização.
**Status Atual:** 🟡 Em Progresso
---

## 1. Fundação do Front Web (Next.js) e Identidade Visual
- [x] **Scaffold Next.js:** Inicializar `apex20-web` usando Next.js 16+ (App Router) e TypeScript.
- [x] **Definição do Padrão Visual:** Estabelecer o guia de estilo para componentes (Typography, Spacing, Shadow patterns) em `apex20-web/src/ui/` para evitar estilos genéricos.
- [x] **Integration UI:** Configurar o consumo de `@/ui` e tokens do Tailwind local.
- [x] **Landing Page (MVP):** Criar a página inicial com estética "Linear-like", focada em alta performance e conversão.
  - Hero split 52/48 com AppMockup simulando VTT em tempo real
  - Systems bar, 3 seções alternadas (Grid Sincronizado, Visão Computacional, IA & Automação)
  - 6-card features grid, CTA banner e footer 4 colunas
  - Dark/light mode com `useTheme` hook, localStorage e anti-flash script
  - Menu hamburguer mobile com estado controlado
  - Link "Sobre nós" em todos os idiomas
- [x] **i18n Implementation:** Integrar `@/i18n` para suporte multi-idioma na interface.
  - 4 locales: `pt-br`, `en`, `es`, `fr`
  - `t()` com dot-notation (ex: `t("landing.nav.features", locale)`)
  - `detectLocale()` via header `Accept-Language` com fallback para `"en"`
  - Middleware Next.js detecta locale por cookie > Accept-Language > fallback
  - `LanguageSwitcher` dropdown customizado com bandeiras SVG (Brasil, EUA, Espanha, França)
- [x] **Testes Unitários (TDD):** Configurar Vitest e implementar testes com abordagem red→green.
  - `vitest.config.ts` com jsdom e path aliases
  - 50 testes passando: i18n, locale-detection, language-switcher, navbar, button
  - Cobertura: detecção de locale, dropdown de idioma, menu hamburguer, links traduzidos
- [x] **Testes Visuais:** Implementar testes de regressão visual nos componentes da landing page.
  - Decisão (ADR-031): Storybook + Playwright visual regression (Docker para snapshots determinísticos)
- [ ] **ConnectRPC Client:** Configurar o cliente de comunicação tipada para consumir os contratos do submodule `./contracts/gen/ts/` (via alias `@contracts/*`).

## 2. Autenticação e Cadastro (Novo 🔐)
- [ ] **Auth Schema:** Criar migração para a tabela `users` (UUIDv7, Argon2 hashing) com suporte a Roles (**GM, Player, Trusted**) e Permissions.
- [ ] **Auth API:** Implementar endpoints de `SignUp` e `SignIn` no `apex20-backend` via ConnectRPC, incluindo a atribuição inicial de Role.
- [ ] **Auth UI (Modules):** Criar o módulo de autenticação no frontend (`modules/auth`) com formulários e lógica de proteção de rotas por Role.
- [ ] **JWT/RS256:** Implementar a geração e validação de tokens assimétricos contendo a claim `role` para autorização cross-service (ADR-002).

## 3. Gestão de Estado e Sincronização
...
- [ ] **State Orchestration:** Configurar **Zustand** para estado global e **XState** para máquinas de estado de jogo (ADR-025).
- [ ] **WebSocket Client:** Implementar o hook de conexão resiliente com o `ws-service` (exponential backoff).
- [ ] **Auth Integration:** Implementar o fluxo de persistência e envio do JWT (RS256) nos headers das requisições e no Handshake do WS (ADR-002).

## 3. Sistema de Grid (MVP)
- [ ] **Grid Canvas/SVG:** Implementar a renderização do grid baseada em coordenadas.
- [ ] **Optimistic Movement:** Implementar o arraste de tokens com atualização instantânea local e reconciliação via servidor (ADR-011).
- [ ] **Soft Locking:** Implementar sinais visuais quando um token está sendo manipulado por outro jogador.

## 4. Mecânicas Core (Backend Support)
- [ ] **Server-side Roller:** Implementar gerador de dados no `backend` usando `crypto/rand` (ADR-016).
- [ ] **Room Isolation:** Finalizar o isolamento de salas por `campaign_id` no `ws-service` via Redis (ADR-019).
- [ ] **Asset Proxy:** Implementar a entrega de imagens via Cloudflare CDN com suporte a Content Hashing (ADR-023).

## 5. Persistência e Infra
- [ ] **Migrations Sprint 2:** Criar tabelas de `campaigns` e `scenes` com suporte a snapshots JSONB (ADR-017).
- [ ] **sqlc CRUD:** Implementar os repositórios básicos para carregar o estado inicial da mesa.

---
**Critério de Aceite da Sprint:** O desenvolvedor deve conseguir abrir o `apex20-web`, ver a interface seguindo o novo padrão visual definido, conectar-se ao WebSocket e mover um token que seja sincronizado com outra aba do navegador.
