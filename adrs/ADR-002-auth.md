# ADR-002: Estratégia de Autenticação Cross-Service (JWT/RS256) 🔐

**Data:** 2026-03-12
**Status:** Proposto

## Contexto
O Apex20 é composto por múltiplos serviços (`apex20-backend`, `apex20-ws`) que precisam validar a identidade do usuário de forma consistente e performática. Em um ambiente de alta concorrência, como o serviço de WebSockets, consultar o banco de dados ou um serviço central de sessão para cada mensagem/conexão é inviável.

## Decisão
Adotar **JWT (JSON Web Tokens)** utilizando o algoritmo de assinatura **RS256 (RSA Signature com SHA-256)**.

1.  **Assinatura Assimétrica**:
    - O `apex20-backend` possui a **Chave Privada** para gerar os tokens.
    - O `apex20-ws` e demais serviços possuem apenas a **Chave Pública** para validar — sem IO de banco.
2.  **Claims do Token**: O token contém `sub` (User ID), `exp`, `iat` e `is_admin` (boolean para acesso administrativo de plataforma).
    - **Sem claim `role`**: A role de um usuário é sempre relativa a uma campanha específica e **não é transportada no JWT**. Ela é resolvida dinamicamente a partir da tabela `campaign_members` usando o `campaign_id` do contexto da requisição ou do handshake WebSocket.
3.  **Distribuição de Chaves**: Implementar um endpoint `.well-known/jwks.json` no serviço de autenticação para rotação automática de chaves (futuro).

### Modelagem de Roles (campaign-scoped)

A role de um usuário é um **atributo da relação entre usuário e campanha**, não do usuário em si. A tabela `campaign_members` é a fonte de verdade:

```
campaign_members (id, campaign_id, user_id, role, created_at, updated_at)
  role: ENUM('gm', 'player', 'trusted')
  UNIQUE(campaign_id, user_id)
```

- **Criação de campanha**: O criador é automaticamente inserido em `campaign_members` como `gm`.
- **Convite**: O usuário convidado é inserido como `player` ou `trusted`.
- **Resolução de permissões**: Ao receber uma requisição com `campaign_id`, o serviço consulta `campaign_members` para determinar a role e aplicar as permissões via `role_permissions`.
- **Admin de plataforma**: Gerenciado via flag `is_admin` na tabela `users` — independente de campanhas.

## Justificativa

- **Performance**: A validação RS256 é feita em memória pelos serviços satélites, sem chamadas de rede ou IO de banco.
- **Segurança**: Mesmo que a chave pública de um serviço (como o WS) seja exposta, o atacante não pode gerar novos tokens válidos.
- **Escalabilidade**: Facilita o escalonamento horizontal, pois os serviços são *stateless* em relação à sessão.

## Consequências
- **Positivas**: Desacoplamento total entre os serviços para validação de identidade; facilidade de integração com Mobile (Expo) e Web (Next.js).
- **Negativas**: Tokens não podem ser facilmente revogados antes da expiração (requer Blacklist em Redis se a revogação imediata for crítica); payload do token aumenta o overhead de cada requisição (manter o JWT enxuto).

## Integração Frontend (apex20-web)

### Armazenamento do Token

O `access_token` JWT é armazenado em **cookie HTTP** (`apex20-token`) com `SameSite=Lax` e `max-age=86400` (24h). Cookie é preferido a `localStorage` por ser automaticamente enviado em requisições SSR e por permitir leitura server-side em `beforeLoad` hooks futuros.

### Estado Global de Autenticação (Zustand)

O estado de auth é um **Zustand store** — não `useState` local. Isso é obrigatório para o padrão de RouterContext do TanStack Router (ADR-038): o store precisa ser compartilhado entre o componente que injeta o contexto (`App`) e os componentes que fazem `signIn`/`signOut` (formulários). Com `useState`, cada chamador teria sua própria instância independente.

```ts
// src/modules/auth/hooks/use-auth.ts
export const useAuth = create<AuthState>((set) => ({
  ...buildState(getToken()), // inicializa lendo o cookie
  signIn(token) { setToken(token); set(buildState(token)); },
  signOut()     { clearToken(); set(buildState(null)); },
}));
```

### Guard de Rotas (RouterContext Pattern)

O store é injetado no `RouterProvider` via `context`, disponibilizando `auth` com type-safety em todos os `beforeLoad` da árvore de rotas:

```tsx
// src/client.tsx
function App() {
  const auth = useAuth();
  return <RouterProvider router={router} context={{ auth }} />;
}

// src/routes/_protected.tsx — rotas que exigem autenticação
export const Route = createFileRoute("/_protected")({
  beforeLoad: ({ context }) => {
    if (!context.auth.isAuthenticated) throw redirect({ to: "/login" });
  },
});

// src/routes/login.tsx — redireciona usuário já autenticado
export const Route = createFileRoute("/login")({
  beforeLoad: ({ context }) => {
    if (context.auth.isAuthenticated) throw redirect({ to: "/" });
  },
});
```

### Decode do Payload (Client-side)

Claims `sub` (userId), `is_admin` e `exp` são extraídos do JWT no cliente via decode Base64 do payload (sem verificação de assinatura — a verificação ocorre no servidor). Isso é seguro pois o cliente não toma decisões de segurança baseado no claim; serve apenas para exibição de UI e pré-checagem de expiração.

## Alternativas Consideradas
- **HS256 (Symmetric)**: Rejeitado porque exige que todos os serviços compartilhem o mesmo segredo, aumentando o risco de vazamento.
- **Opaque Tokens**: Rejeitado pelo custo de latência de consultar um store central (Redis/DB) em cada validação.
- **useState para auth no frontend**: Rejeitado — cada componente teria sua própria instância do estado. Impossibilita o padrão de RouterContext onde o `App` (fora do router) e os formulários de auth precisam do mesmo estado global.
