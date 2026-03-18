# ADR-010: Estratégia de Testes (TDD Rigoroso e Testcontainers) 🧪🛡️

**Data:** 2026-03-14
**Status:** Aceito (Atualizado)

## Contexto
O Apex20 exige alta confiabilidade e resiliência a refatorações. Erros em lógica de combate ou sincronização de dados podem arruinar a experiência do usuário. Para garantir a qualidade "Elite Standard", precisamos de uma metodologia de desenvolvimento que minimize bugs desde a concepção e uma infraestrutura de testes que replique fielmente o ambiente de produção, evitando falsos positivos causados por bancos de dados em memória ou mocks excessivos.

## Decisão
Adotar o **TDD (Test-Driven Development)** como metodologia obrigatória e o uso de **Testcontainers** para validação de infraestrutura real.

### 1. Metodologia TDD (Red-Green-Refactor)
- **Mandato:** Nenhuma funcionalidade ou correção de bug deve ser iniciada sem a escrita prévia de um teste que falhe (RED).
- **Ciclo:** O código deve ser escrito estritamente para fazer o teste passar (GREEN) e, em seguida, limpo e otimizado (REFACTOR).
- **Cobertura:** Alvo de **100% de cobertura de requisitos** e **> 90% de cobertura de código**. Exceções a este rigor devem ser raras e formalmente justificadas.
- **TDD para UI (Story-first):** Para componentes de interface em `apex20-web/src/ui/`, o ciclo de TDD inicia-se com a criação da história no **Storybook** (ADR-031) definindo os estados visuais, seguida pela implementação do componente.

### 2. Infraestrutura de Testes de Integração (Testcontainers)
- **Proibição:** É terminantemente proibido o uso de SQLite ou bancos em memória para testar camadas de persistência que utilizam PostgreSQL em produção.
- **Fidelidade:** Utilizar a biblioteca **Testcontainers** para subir instâncias reais e efêmeras de:
    - **PostgreSQL:** Para validar consultas `sqlc`, migrações `tern` e integridade referencial.
    - **Redis:** Para validar a lógica de Pub/Sub e cache do `ws-service`.
    - **RabbitMQ:** Para validar o roteamento e processamento de mensagens assíncronas.
- **Isolamento:** Cada suíte de teste deve operar em containers limpos para evitar poluição de estado entre execuções.

### 3. Modelo Híbrido: Pirâmide + Troféu de Testes
- **Unitários (Base):** Testes de lógica pura, rápidos e independentes (Vitest / Go Test).
- **Integração (Corpo):** Foco massivo em validar a interação entre o código e os serviços reais (Postgres/Redis).
- **E2E (Topo):** Fluxos críticos de usuário validados via **Playwright**, simulando a experiência real de ponta a ponta.

### 4. Automação e Pre-commit
- Os testes unitários e de integração leves devem ser executados localmente via Git Hooks antes de qualquer commit (ADR-006).
- O pipeline de CI/CD (GitHub Actions) executará a suíte completa de testes antes de qualquer merge para as branches `dev` ou `master`.

## Justificativa
- **Confiabilidade Máxima:** Testcontainers elimina o "funciona na minha máquina", garantindo que o código seja validado contra o mesmo motor de banco de dados de produção.
- **Design de Código:** O TDD força o desacoplamento e a criação de interfaces limpas, facilitando a manutenção futura.
- **Segurança em Refatorações:** A alta cobertura permite realizar mudanças profundas na arquitetura com a certeza de que as regras de negócio permanecem íntegras.

## Consequências
- **Positivas:** Redução drástica de bugs em produção; documentação técnica sempre atualizada via testes; arquitetura modular e testável por padrão.
- **Negativas:** Curva de aprendizado inicial para TDD rigoroso; tempo de execução da suíte de testes é superior ao uso de mocks simples (compensado pelo cache de CI por repositório).

## Referências
- **ADR-004:** Migrações (Tern e PostgreSQL).
- **ADR-011:** Sincronização e Resolução de Conflitos.
- **ADR-019:** Escalonamento de WebSockets (Redis).
- **ADR-031:** Documentação de Design System e Testes Visuais.
- **ADR-034:** Resiliência de Dados (UUIDv7).
