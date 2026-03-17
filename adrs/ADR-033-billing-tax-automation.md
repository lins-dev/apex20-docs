# ADR-033: Automação de Faturamento e Emissão de Notas Fiscais (NF-e) 🧾🏛️

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
O Apex20 opera em um cenário tributário complexo. No Brasil, a confirmação de um pagamento (ADR-032) exige a emissão de uma **NFS-e (Nota Fiscal de Serviço Eletrônica)**. Internacionalmente, o foco é o cálculo e reporte de **Sales Tax** (EUA) e **VAT** (Europa). A arquitetura deve suportar a transição futura de uma única entidade nacional para uma estrutura multi-entidade (Offshore), tratando o faturamento de forma agnóstica à localização da sede fiscal.

## Decisão
Implementar uma arquitetura de faturamento baseada na **Localização do Usuário**, separando os fluxos fiscais em "Nacional" (Brasil) e "Global" (Resto do Mundo).

### 1. Identificação de Região Fiscal
- **Mecanismo:** O sistema utilizará uma combinação de **Geo-IP** e **Billing Address** (fornecido pelo Stripe) para classificar o usuário.
- **Branching Lógico:** 
    - **Se Usuário = Brasil:** Direcionar para o fluxo de NFS-e Brasileira.
    - **Se Usuário = Internacional:** Direcionar para o fluxo de Invoice/Tax Global.

### 2. Fluxo Nacional (NFS-e via Gateway)
- **Integração:** Utilizar um gateway de API fiscal (ex: **e-notas** ou **Focus NFE**) integrado assincronamente ao Stripe.
- **Processo:** Confirmação de Pagamento -> Webhook -> Fila de Emissão -> Gateway Fiscal -> Registro na Prefeitura -> Envio de PDF/XML ao usuário.
- **Certificação:** Uso de certificado digital A1 para assinatura automática.

### 3. Fluxo Global (Tax Compliance & Invoicing)
- **Cálculo de Impostos:** Utilizar o **Stripe Tax** para calcular Sales Tax/VAT em tempo real no checkout.
- **Faturamento:** Gerar **Invoices** multilíngues compatíveis com as regras locais (ex: VAT MOSS na UE).
- **Relatórios:** Exportação consolidada para contabilidade, preparando o terreno para uma futura sede internacional (Rota B).

### 4. Arquitetura de Resiliência
- **Assincronismo:** A falha na emissão de um documento fiscal nunca deve bloquear o acesso do jogador ao serviço (ADR-018).
- **Fila de Retentativa (Retry Queue):** Implementar retentativas automáticas no `apps/backend` para lidar com a alta instabilidade dos webservices de prefeituras no Brasil.

## Justificativa
- **Conformidade "Elite":** Garante que o projeto esteja legalmente seguro desde o primeiro dia, evitando passivos tributários.
- **Pronto para o Futuro:** A separação lógica permite migrar para uma estrutura Offshore sem alterar o código-fonte, apenas configurando novos adaptadores de faturamento.
- **Automação Total:** Elimina a necessidade de processos manuais de emissão de notas, permitindo que a equipe foque no desenvolvimento do produto.

## Consequências
- **Positivas:** Redução de riscos fiscais; escalabilidade internacional simplificada; automação de fluxos burocráticos.
- **Negativas:** Custo por nota emitida nos gateways brasileiros; complexidade na gestão de impostos variáveis por estado/país.

## Referências
- **ADR-015:** Privacidade e Segurança (Dados Fiscais).
- **ADR-018:** Gestão de Cotas de Recursos.
- **ADR-032:** Estratégia de Monetização e Pagamentos (Polymorphic Mapping).
