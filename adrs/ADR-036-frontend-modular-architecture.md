# ADR-036: Arquitetura Frontend Modular Baseada em Funcionalidades (Feature-based) 🧩

**Data:** 2026-03-16
**Atualizado:** 2026-03-25
**Status:** Aceito (Atualizado)

## Contexto
O `apex20-web` é um repositório frontend independente que exige alta manutenibilidade e escalabilidade. Estruturas de pastas puramente técnicas (ex: colocar todos os componentes em uma única pasta `components/`) tornam-se caóticas à medida que o projeto cresce, dificultando o isolamento de lógica, testes e a navegação de novos desenvolvedores. Precisamos de uma estrutura que reflita as funcionalidades do domínio (VTT, Chat, Fichas) e minimize o acoplamento.

## Decisão
Adotar uma **Arquitetura Baseada em Funcionalidades (Feature-based Architecture)** para as aplicações Frontend (`apex20-web` e, futuramente, o app mobile independente).

### 1. Estrutura de Diretórios (`src/`)
A lógica principal será organizada no diretório `modules/`, onde cada subpasta representa uma funcionalidade autônoma do domínio.

```text
src/
├── routes/             # TanStack Router — Definições de Rotas e Layouts (substitui src/app/ do Next.js)
│   ├── __root.tsx      # Root layout (html, body, HeadContent, ScriptOnce, Scripts)
│   ├── index.tsx       # Rota / (homepage)
│   ├── login.tsx       # Rota /login
│   ├── signup.tsx      # Rota /signup
│   └── _protected.tsx  # Layout para rotas autenticadas (/dashboard, /campaigns, /vtt)
├── modules/            # Funcionalidades Isoladas (O "Cérebro" do App)
│   ├── <feature>/      # Nome da funcionalidade (ex: grid, chat, auth)
│   │   ├── components/ # Componentes exclusivos desta funcionalidade
│   │   ├── hooks/      # Hooks reativos específicos
│   │   ├── actions/    # Chamadas de API/RPC (ConnectRPC)
│   │   ├── store/      # Estado local (Zustand/XState)
│   │   └── types.ts    # Definições de tipos locais
├── components/         # Componentes globais e UI (Layout, Nav, Modais)
├── hooks/              # Hooks de infraestrutura (useSocket, useTheme)
├── lib/                # Configurações de terceiros (ConnectRPC Client, Auth, i18n helpers)
├── styles/             # globals.css e tokens de estilo locais
├── test/               # Utilitários de teste compartilhados (renderWithRouter, etc.)
├── router.ts           # Factory getRouter() — nova instância por request (crítico para SSR)
├── client.tsx          # Entry point do cliente (hydrateRoot + RouterProvider com context)
└── ssr.tsx             # Entry point do servidor SSR (createStartHandler)
```

### 2. Regras de Dependência
- Um módulo em `modules/` pode importar de `components/` globais e `hooks/` globais.
- Um módulo **não deve** importar diretamente de outro módulo em `modules/`. Comunicações entre funcionalidades devem ocorrer via **Estado Global (Zustand)** ou **Eventos de Sistema**.
- O diretório `routes/` deve atuar apenas como uma "casca" que importa os componentes dos módulos para compor as páginas. Não deve conter lógica de negócio.
- A lógica de guarda de autenticação e detecção de i18n reside em `beforeLoad` nas rotas ou em helpers em `lib/` — não nos módulos.

## Justificativa
- **Isolamento:** Facilita a modificação ou substituição de uma funcionalidade inteira sem afetar o restante do sistema.
- **Cognição:** Reduz a carga cognitiva ao permitir que o desenvolvedor foque apenas nos arquivos relevantes para a tarefa atual (ex: mexer no Grid envolve apenas a pasta `modules/grid`).
- **Testabilidade:** Permite a criação de testes de integração e unitários localizados dentro da própria pasta da funcionalidade.
- **Escalabilidade:** Prepara o terreno para o compartilhamento de lógica entre Web e Mobile na Sprint 3.

## Consequências
- **Positivas:** Separação clara de responsabilidades; facilidade de manutenção; arquitetura previsível e profissional.
- **Negativas:** Leve aumento no número de pastas e arquivos iniciais; exige disciplina rigorosa para evitar importações cruzadas proibidas.

## Referências
- **ADR-001:** Escolha da Stack Tecnológica.
- **ADR-025:** Gestão de Estados de Jogo (Zustand/XState).
- **ADR-034:** Resiliência de Dados e Isolamento por Tenant.
- **ADR-038:** Migração Next.js → TanStack (motivo da mudança de `app/` para `routes/`).
- **Contexto Visual:** [VTT Wireframes & Walkthrough](../wireframes/vtt-walkthrough.md).
