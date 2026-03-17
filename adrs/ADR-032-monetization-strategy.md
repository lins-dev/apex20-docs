# ADR-032: Estratégia de Monetização e Pagamentos (Polymorphic Payment Mapping) 💸💳

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
Para garantir a sustentabilidade do Apex20, precisamos de um sistema de monetização flexível. O mercado global exige o Stripe, mas o mercado brasileiro pode demandar gateways locais por questões de taxas ou métodos específicos (Pix). Um acoplamento direto com o Stripe (ex: campo `stripe_customer_id`) no banco de dados principal dificultaria a troca de provedores ou a expansão para novos mercados.

## Decisão
Adotar o **Stripe** como provedor inicial, mas implementar uma arquitetura de **Relação Polimórfica** para a gestão de contas de pagamento.

### 1. Relação Polimórfica (Payment Profiles)
- **Estrutura:** Utilizar uma tabela `payment_profiles` vinculada ao `User` via `user_id` (UUIDv7).
- **Campos:** `provider` (enum: stripe, pagarme, etc), `external_id` (ID no provedor), `status` e `metadata` (JSONB).
- **Vantagem:** Desacoplamento total da tabela `Users`. Permite migrar de provedor ou suportar múltiplos gateways simultaneamente apenas adicionando registros.

### 2. Abstração via Portas e Adaptadores (Hexagonal)
- O Domínio do Apex20 não conhecerá o Stripe. Ele interagirá com uma interface `PaymentProvider`.
- O `StripeAdapter` será a primeira implementação. O sistema de Tiers e Cotas (ADR-018) consumirá esta abstração.

### 3. Escopo de Responsabilidade
- Este ADR foca estritamente no **fluxo financeiro** (cobrança, planos, webhooks de pagamento).
- A responsabilidade de conformidade fiscal e emissão de documentos legais (NF-e, VAT) é delegada ao **ADR-033**.

## Justificativa
- **Isolamento de Infraestrutura:** Mantém o core do sistema limpo de detalhes de APIs de terceiros.
- **Flexibilidade de Negócio:** Permite otimizar taxas de conversão trocando o provedor por região sem refatoração de banco de dados.
- **Escalabilidade Global:** Facilita a entrada em mercados com restrições financeiras específicas.

## Consequências
- **Positivas:** Arquitetura "Future-proof"; facilidade de testes com mocks; suporte multi-gateway.
- **Negativas:** Pequeno aumento na complexidade de consultas (Join adicional para obter status de pagamento); exige mapeamento rigoroso de status entre diferentes provedores.

## Referências
- **ADR-015:** Privacidade e Segurança (Dados Financeiros).
- **ADR-018:** Gestão de Cotas de Recursos.
- **ADR-033:** Automação de Faturamento e NF-e.
- **ADR-034:** Resiliência de Dados (UUIDv7).
