# ADR-001: Escolha da Stack Tecnológica 🛠️

**Data:** 2023-10-27
**Status:** Aceito

## Contexto
O projeto Apex20 exige alta performance, escalabilidade e suporte a recursos avançados (WebXR, Computer Vision). Precisamos de uma stack que permita desenvolvimento ágil em um monorepo, com forte tipagem e baixo overhead de execução.

## Decisão
A stack escolhida para o Apex20 é composta por:

1.  **Monorepo:** Turborepo com pnpm.
2.  **Backend Core:** Go com framework Chi (Arquitetura Hexagonal).
3.  **Real-time:** Serviço Go WebSocket customizado.
4.  **Frontend:** Next.js (App Router) + shadcn/ui.
5.  **Mobile:** Expo + NativeWind.
6.  **Banco de Dados:** PostgreSQL (sqlc para consultas tipadas).
7.  **Cache/PubSub:** Redis.
8.  **Contratos:** Protobuf para comunicação entre serviços e frontend.

## Justificativa

- **Go:** Escolhido pela excelente performance em concorrência, simplicidade de manutenção e binários leves. A arquitetura hexagonal no Go garante isolamento total das regras de negócio.
- **Turborepo:** Essencial para gerenciar múltiplas aplicações compartilhando pacotes (`contracts`, `ui`, `i18n`) de forma eficiente e rápida (caching de builds).
- **Next.js:** Framework React líder para Web, oferecendo SSR/ISR para carregamento rápido e uma excelente experiência de desenvolvedor.
- **Expo:** Permite desenvolvimento mobile rápido com React Native e acesso simplificado a APIs nativas para recursos futuros.
- **sqlc:** Diferente de ORMs tradicionais, o sqlc gera código Go tipado a partir de SQL puro, garantindo máxima performance e segurança em tempo de compilação.

## Consequências
- **Positivas:** Performance otimizada, forte tipagem em todo o fluxo (Type-safety), reaproveitamento de componentes e contratos.
- **Negativas:** Curva de aprendizado inicial maior devido à Arquitetura Hexagonal e Protobuf. Necessidade de processos de geração de código (sqlc, protobuf) durante o desenvolvimento.
