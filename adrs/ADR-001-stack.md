# ADR-001: Escolha da Stack Tecnológica 🛠️

**Data:** 2023-10-27
**Atualizado:** 2026-03-22
**Status:** Aceito (Atualizado)

## Contexto
O projeto Apex20 exige alta performance, escalabilidade e suporte a recursos avançados (WebXR, Computer Vision). Precisamos de uma stack que permita desenvolvimento ágil em um polyrepo de repositórios independentes, com forte tipagem e baixo overhead de execução.

## Decisão
A stack escolhida para o Apex20 é composta por:

1.  **Polyrepo:** Repositórios Git independentes com submodules (`apex20-docs`, `apex20-contracts`).
2.  **Backend Core:** Go com framework Chi (Arquitetura Hexagonal).
3.  **Real-time:** Serviço Go WebSocket customizado.
4.  **Frontend:** Next.js (App Router) + shadcn/ui.
5.  **Mobile:** Expo + NativeWind.
6.  **Banco de Dados:** PostgreSQL (sqlc para consultas tipadas).
7.  **Cache/PubSub:** Redis.
8.  **Contratos:** Protobuf para comunicação entre serviços e frontend.

## Arquitetura Hexagonal — Regras de Dependência (Backend Go)

O `apex20-backend` e o `apex20-ws` seguem o padrão **Ports and Adapters** com a seguinte estrutura de camadas e regras de dependência obrigatórias:

### Camadas

```
internal/
  domain/          # Entidades puras e regras de negócio. Sem dependências externas.
  application/
    port/          # Interfaces (contratos) definidas pela camada de aplicação.
    usecase/       # Orquestração de casos de uso. Depende apenas de domain/ e port/.
  infrastructure/
    adapter/
      inbound/     # HTTP handlers, WS handlers. Depende de usecase/.
      outbound/    # Repositórios (sqlc), Redis, R2. Implementa interfaces de port/.
```

### Fluxo de dependência obrigatório

```
Inbound Adapter → UseCase → Port ← Outbound Adapter
                                 ↑
                            (implementação injetada via DI no cmd/)
```

### Regras

1. **Handlers nunca acessam ports diretamente.** Um adapter inbound (HTTP handler, WS handler) só pode depender de um use case — jamais de um `port.Repository` diretamente.
2. **Lógica de negócio pertence aos use cases.** Geração de IDs (`uuid.NewV7`), timestamps (`time.Now`), construção de entidades de domínio — tudo isso é responsabilidade do use case, não do handler.
3. **Ports residem em `application/port/`.** Um port é um contrato exigido pela camada de aplicação. Nunca deve ser definido dentro de `infrastructure/`.
4. **O domínio não importa nada externo ao próprio domínio**, exceto o `apex20-contracts` (fonte única de verdade para tipos compartilhados entre serviços, conforme ADR-007).
5. **A injeção de dependência ocorre exclusivamente no `cmd/`.** O ponto de entrada (`cmd/api/main.go`, `cmd/seed/main.go`) é o único lugar onde implementações concretas de infrastructure são instanciadas e injetadas nos use cases.

## Justificativa

- **Go:** Escolhido pela excelente performance em concorrência, simplicidade de manutenção e binários leves. A arquitetura hexagonal no Go garante isolamento total das regras de negócio.
- **Git Submodules:** Permitem compartilhar `apex20-contracts` e `apex20-docs` entre todos os repositórios de forma explícita, versionada e sem overhead de tooling de workspace.
- **Next.js:** Framework React líder para Web, oferecendo SSR/ISR para carregamento rápido e uma excelente experiência de desenvolvedor.
- **Expo:** Permite desenvolvimento mobile rápido com React Native e acesso simplificado a APIs nativas para recursos futuros.
- **sqlc:** Diferente de ORMs tradicionais, o sqlc gera código Go tipado a partir de SQL puro, garantindo máxima performance e segurança em tempo de compilação.

## Consequências
- **Positivas:** Performance otimizada, forte tipagem em todo o fluxo (Type-safety), reaproveitamento de componentes e contratos.
- **Negativas:** Curva de aprendizado inicial maior devido à Arquitetura Hexagonal e Protobuf. Necessidade de processos de geração de código (sqlc, protobuf) durante o desenvolvimento.
