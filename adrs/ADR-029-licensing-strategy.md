# ADR-029: Estratégia de Licenciamento de Conteúdo (OGL/ORC/SRD) 📜⚖️

**Data:** 2026-03-14
**Status:** Proposto

## Contexto
Um Virtual Tabletop (VTT) é alimentado por dados de sistemas de jogo (regras, monstros, itens, magias). Muitos desses dados são protegidos por direitos autorais, mas disponibilizados para terceiros através de licenças abertas como a **OGL 1.0a (Open Game License)** da Wizards of the Coast e a mais recente **ORC (Open RPG Creative License)** da Paizo. Para que o Apex20 seja comercialmente viável e legalmente seguro, precisamos de uma estratégia clara de como armazenar, exibir e atribuir esses conteúdos.

## Decisão
Adotar uma arquitetura de dados que suporte nativamente conteúdos sob licenças abertas e facilite a importação privada de conteúdos proprietários pelos usuários.

### 1. Suporte Nativo a SRDs (System Reference Documents)
- **Implementação:** O Apex20 incluirá "Compêndios" base de dados pré-carregados para sistemas populares (ex: SRD de D&D 5e e PF2e).
- **Atribuição:** Cada entrada de dado (monstro, magia, etc.) deverá conter metadados de atribuição obrigatórios, citando a fonte e a licença correspondente, conforme exigido pelas licenças OGL/ORC.
- **Imutabilidade:** Os dados oficiais das SRDs serão tratados como "Read-only" para garantir a integridade da licença original.

### 2. Modelo "Bring Your Own Content" (BYOC)
- **Privacidade:** Conteúdos protegidos por direitos autorais que não possuem licença aberta (ex: aventuras oficiais, livros de expansão) não serão distribuídos nativamente pelo Apex20.
- **Importação:** Fornecer ferramentas (como o **Foundry VTT Importer**) que permitam ao usuário importar seus próprios arquivos JSON/módulos legalmente adquiridos. Esses dados serão armazenados de forma privada no `campaign_id` do usuário (ADR-034), garantindo que o Apex20 atue apenas como o software de visualização ("Fair Use").

### 3. Separação de Camadas de Dados
- **Core Data:** Dados fundamentais do sistema sob licença aberta.
- **Community Data:** Dados criados por usuários ou convertidos via plugins (ADR-014).
- **User Private Data:** Dados importados de fontes proprietárias, restritos à conta do proprietário.

### 4. Gestão de Notificações de Violação (DMCA)
- Implementar um fluxo claro para denúncia e remoção de conteúdos que violem direitos autorais em módulos públicos ou compartilhados da comunidade.

## Justificativa
- **Segurança Jurídica:** Protege os desenvolvedores e a empresa contra processos de violação de copyright.
- **Interoperabilidade:** Facilita a migração de jogadores de outras plataformas (Roll20, Foundry) para o Apex20.
- **Respeito à Propriedade Intelectual:** Valoriza o trabalho dos criadores de conteúdo originais, garantindo a atribuição correta.

## Consequências
- **Positivas:** Plataforma legalmente robusta; ecossistema de dados organizado; facilidade em obter parcerias com editoras no futuro.
- **Negativas:** Exige esforço técnico para mapear e normalizar diferentes esquemas de SRD (JSON mapping); limita a oferta inicial de conteúdo "pronto para jogar" apenas a licenças abertas.

## Referências
- **ADR-014:** Arquitetura de Plugins.
- **ADR-015:** Privacidade e Segurança.
- **ADR-034:** Resiliência de Dados (Isolamento por Tenant).
