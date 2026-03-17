# Apex20 — Plano de Migração: Monorepo → Repositórios Independentes

**Status:** Pendente
**Decisão:** ADR-005 (package manager) revisada — migrar de pnpm workspaces/Turborepo para repositórios Git independentes.
**Motivação:** Complexidade de build excessiva para um único mantenedor. Bugs de Next.js 16 canary + Turbopack bloqueando tarefas mundanas. Manutenção de tooling (eslint, vitest, tsconfig) consumindo mais tempo que o produto.

---

## Visão Geral dos Repositórios

| Repositório | Conteúdo | Stack | Gerenciador |
|---|---|---|---|
| `apex20-docs` | ADRs, sprints, arquitetura, wireframes | Markdown | — |
| `apex20-contracts` | `.proto` + código gerado (TS e Go) | Protobuf / Buf CLI | — |
| `apex20-web` | Frontend (landing, auth, VTT app) | Next.js 16, TypeScript | npm |
| `apex20-backend` | API HTTP/gRPC | Go | Go modules |
| `apex20-ws` | WebSocket service | Go | Go modules |

Todos os repositórios incluem `apex20-docs` e `apex20-contracts` como **git submodules**.

---

## Repositório: `apex20-docs`

Fonte única de verdade para documentação. Todos os outros repos o consomem como submodule em `./docs/`.

```
apex20-docs/
  adrs/
    ADR-001-stack.md
    ADR-002-auth.md
    ... (todos os 36 ADRs existentes)
    ADR-037-polyrepo-migration.md   ← novo, documenta esta decisão
  tasks/
    sprint-1.md
    sprint-2.md
    sprint-3.md
    ...
  wireframes/
    assets/
    vtt-walkthrough.md
    vtt-wireframe.html
  ARCHITECTURE.md
  PRD.md
  STANDARDS.md
  TASKS.md
  README.md
```

**Workflow de atualização:**
```bash
# Editar docs no repo apex20-docs, commitar e pushar
# Em qualquer outro repo, para puxar as atualizações:
git submodule update --remote docs
git add docs && git commit -m "chore(docs): sync submodule"
```

---

## Repositório: `apex20-contracts`

Fonte única de verdade para os contratos entre serviços. Contém os `.proto` e o **código gerado commitado** — consumidores não precisam rodar `buf generate`, apenas fazem `git submodule update`.

```
apex20-contracts/
  proto/
    apex20/
      v1/
        chat.proto
        grid_events.proto
        handshake.proto
        health.proto
        auth.proto          ← a adicionar
        session.proto       ← a adicionar
  gen/
    ts/                     ← gerado via @bufbuild/protoc-gen-es
      proto/apex20/v1/
        chat_pb.ts
        chat_connect.ts
        grid_events_pb.ts
        grid_events_connect.ts
        handshake_pb.ts
        handshake_connect.ts
        health_pb.ts
        health_connect.ts
    go/                     ← gerado via protoc-gen-go + protoc-gen-connect-go
      proto/apex20/v1/
        chat.pb.go
        grid_events.pb.go
        handshake.pb.go
        health.pb.go
      proto/apex20/v1/apex20v1connect/
        chat.connect.go
        grid_events.connect.go
        handshake.connect.go
        health.connect.go
  buf.yaml
  buf.gen.yaml
  README.md
```

**`buf.yaml`** (sem alterações em relação ao atual):
```yaml
version: v1
name: buf.build/apex20/contracts
lint:
  use:
    - STANDARD
  except:
    - PACKAGE_DIRECTORY_MATCH
breaking:
  use:
    - FILE
```

**`buf.gen.yaml`** (sem alterações em relação ao atual):
```yaml
version: v1
plugins:
  - plugin: go
    out: gen/go
    opt: paths=source_relative
  - plugin: connect-go
    out: gen/go
    opt: paths=source_relative
  - plugin: es
    out: gen/ts
    opt: target=ts
  - plugin: connect-es
    out: gen/ts
    opt: target=ts
```

**Workflow ao alterar um contrato:**
```bash
# 1. Editar o .proto em apex20-contracts
# 2. Gerar o código
buf generate

# 3. Commitar tudo (proto + gerado)
git add proto/ gen/
git commit -m "feat(contracts): add auth service"
git push

# 4. Em cada repo consumidor, atualizar o submodule
git submodule update --remote contracts
git add contracts && git commit -m "chore(contracts): sync auth service"
```

---

## Repositório: `apex20-web`

Frontend standalone. Usa npm puro, sem workspace. O código gerado dos contratos é consumido diretamente do submodule via path alias no `tsconfig.json`.

```
apex20-web/
  contracts/          ← git submodule (apex20-contracts)
  docs/               ← git submodule (apex20-docs)
  public/
  src/
    app/
      [lang]/
        layout.tsx
        lang-updater.tsx
        page.tsx          ← landing page
      layout.tsx
      page.tsx            ← redirect para [lang]
      not-found.tsx
    middleware.ts
    i18n/
      index.ts            ← t(key, locale) — internalizado, sem pacote externo
      locales/
        en/
          common.json
          landing.json
        pt-BR/
          common.json
          landing.json
        es/
          common.json
          landing.json
        fr/
          common.json
          landing.json
    ui/
      components/         ← design system internalizado (Button, Input, etc.)
        button.tsx
        input.tsx
        ...
      tokens/
        colors.ts
        typography.ts
        spacing.ts
  next.config.ts
  tsconfig.json           ← path alias @contracts/* → ./contracts/gen/ts/*
  package.json
  .env.example
  .gitmodules
```

**`package.json`** (somente dependências reais — sem workspace deps):
```json
{
  "name": "apex20-web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "dependencies": {
    "next": "16.1.7",
    "react": "19.2.3",
    "react-dom": "19.2.3",
    "@connectrpc/connect": "^1.6.1",
    "@connectrpc/connect-web": "^1.6.1",
    "@bufbuild/protobuf": "^2.2.7",
    "zustand": "^5.0.3",
    "xstate": "^5.19.2"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "@testing-library/react": "^16.3.0",
    "@testing-library/user-event": "^14.6.1",
    "@vitejs/plugin-react": "^4.3.4",
    "@vitest/coverage-v8": "^3.1.1",
    "jsdom": "^26.1.0",
    "vitest": "^3.1.1",
    "tailwindcss": "^4",
    "typescript": "^5"
  }
}
```

**`tsconfig.json`** — path alias para contratos e src:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@contracts/*": ["./contracts/gen/ts/*"]
    }
  }
}
```

**Nota sobre `packages/ui` e `packages/i18n`:**
Com um único frontend, não há motivo para pacotes externos. O design system (`ui/`) e o i18n (`i18n/`) são internalizados diretamente em `src/`. Se no futuro houver um app mobile separado, extrair o UI para um pacote naquele momento.

---

## Repositório: `apex20-backend`

Sem alterações na estrutura interna. O Go module muda para refletir o novo repositório. O código gerado dos contratos é consumido via submodule.

```
apex20-backend/
  contracts/          ← git submodule (apex20-contracts)
  docs/               ← git submodule (apex20-docs)
  internal/
    application/
      port/
        event_publisher.go
    infrastructure/
      adapter/
        inbound/
          http/
            server.go
            server_test.go
        outbound/
          redis/
            publisher.go
            publisher_test.go
            ping_pong_integration_test.go
          repository/
            migrations/
              001_create_users.sql
              002_create_campaigns.sql
              tern.conf
      port/
        http_server.go
  cmd/
    api/
      main.go
  go.mod
  go.sum
  sqlc.yaml
  .env.example
  Makefile
```

**`go.mod`** — module path atualizado:
```
module github.com/apex20/backend

go 1.26.1

require (
  ...dependências existentes...
)

// Contratos via submodule local
replace github.com/apex20/contracts => ./contracts/gen/go
```

**Alternativa para contratos no Go** (mais simples que replace):
O submodule `contracts/gen/go/` pode ser importado diretamente como caminho relativo nos arquivos Go, já que `go.mod` com `replace` aponta para o diretório local. Ao atualizar o submodule, os tipos são atualizados automaticamente sem publicar nada no pkg.go.dev.

---

## Repositório: `apex20-ws`

Mesma estrutura do backend. WebSocket service independente.

```
apex20-ws/
  contracts/          ← git submodule (apex20-contracts)
  docs/               ← git submodule (apex20-docs)
  internal/
    application/
      port/
        redis_subscriber.go
    infrastructure/
      adapter/
        inbound/
          websocket/
            server.go
            server_test.go
        outbound/
          redis/
            subscriber.go
            subscriber_test.go
      port/
        ws_server.go
  cmd/
    ws/
      main.go
  go.mod
  go.sum
  .env.example
  Makefile
```

**`go.mod`:**
```
module github.com/apex20/ws

go 1.26.1

replace github.com/apex20/contracts => ./contracts/gen/go
```

---

## Passos de Migração

### Passo 1 — Criar e popular `apex20-docs`
```bash
# Criar repositório apex20-docs no GitHub (privado)
# Copiar conteúdo
cp -r apex20/docs/* apex20-docs/
cd apex20-docs
git init && git add . && git commit -m "feat(docs): initial migration from monorepo"
git remote add origin git@github.com:apex20/apex20-docs.git
git push -u origin main
```

### Passo 2 — Criar e popular `apex20-contracts`
```bash
# Criar repositório apex20-contracts no GitHub (privado)
cp -r apex20/packages/contracts/* apex20-contracts/
# Remover node_modules, .turbo, package.json (não são necessários standalone)
cd apex20-contracts
git init && git add . && git commit -m "feat(contracts): initial migration from monorepo"
git remote add origin git@github.com:apex20/apex20-contracts.git
git push -u origin main
```

### Passo 3 — Criar `apex20-web`
```bash
# Criar repositório apex20-web no GitHub (privado)
# Copiar apps/web + internalizar packages/ui e packages/i18n
cp -r apex20/apps/web/* apex20-web/
# Mover packages/ui/src → apex20-web/src/ui/
# Mover packages/i18n/src → apex20-web/src/i18n/
# Atualizar imports (de @apex20/ui → @/ui, de @apex20/i18n → @/i18n)
# Adicionar submodules
cd apex20-web
git submodule add git@github.com:apex20/apex20-contracts.git contracts
git submodule add git@github.com:apex20/apex20-docs.git docs
npm install
npm run build  # deve passar sem erros
git add . && git commit -m "feat(web): initial migration from monorepo"
git push -u origin main
```

### Passo 4 — Criar `apex20-backend`
```bash
cp -r apex20/apps/backend/* apex20-backend/
cd apex20-backend
# Atualizar go.mod: module github.com/apex20/backend
# Adicionar replace para contratos
git submodule add git@github.com:apex20/apex20-contracts.git contracts
git submodule add git@github.com:apex20/apex20-docs.git docs
go build ./...
go test ./...
git add . && git commit -m "feat(backend): initial migration from monorepo"
git push -u origin main
```

### Passo 5 — Criar `apex20-ws`
```bash
cp -r apex20/apps/ws-service/* apex20-ws/
cd apex20-ws
# Atualizar go.mod: module github.com/apex20/ws
git submodule add git@github.com:apex20/apex20-contracts.git contracts
git submodule add git@github.com:apex20/apex20-docs.git docs
go build ./...
go test ./...
git add . && git commit -m "feat(ws): initial migration from monorepo"
git push -u origin main
```

### Passo 6 — Clonar qualquer repositório do zero
```bash
# Qualquer desenvolvedor que clonar um repositório:
git clone --recurse-submodules git@github.com:apex20/apex20-web.git
# ou, após um clone normal:
git submodule update --init --recursive
```

### Passo 7 — Arquivar monorepo
```bash
# No GitHub: Settings → Archive repository
# Manter disponível como referência histórica
```

---

## O que NÃO migrar

| Item | Destino |
|---|---|
| `packages/ui` | Internalizado em `apex20-web/src/ui/` |
| `packages/i18n` | Internalizado em `apex20-web/src/i18n/` |
| `turbo.json` | Descartado |
| `pnpm-workspace.yaml` | Descartado |
| `eslint.config.mjs` (raiz) | Cada repo tem o seu próprio |
| `sandbox/web` | Descartado (era apenas teste) |
| `.turbo/` (qualquer) | Descartado |

---

## Ordem de Desenvolvimento Pós-Migração

Com os repositórios separados, o fluxo de trabalho se torna linear:

1. **Sprint atual** → concluir landing page em `apex20-web`
2. **Contratos** → adicionar `auth.proto` e `session.proto` em `apex20-contracts`
3. **Backend** → implementar auth API em `apex20-backend` (ConnectRPC, JWT/RS256)
4. **Web** → implementar auth UI consumindo os tipos gerados via submodule
5. **WS** → implementar sessão de jogo em `apex20-ws`

Cada repositório evolui de forma independente. A sincronização entre eles acontece apenas quando o contrato muda — explícita, versionada, e auditável via git.
