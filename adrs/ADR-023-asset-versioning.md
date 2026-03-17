# ADR-023: Versionamento de Assets e Invalidação de Cache (Content Hashing) 🖼️⚡

**Data:** 2026-03-13
**Status:** Aceito

## Contexto
O Apex20 armazena grandes volumes de assets (mapas em 4K, tokens, áudios) no **Cloudflare R2**. Para garantir uma UX fluida e baixo custo de tráfego, precisamos de uma estratégia de cache agressiva (CDN). No entanto, se um mestre atualizar um mapa, todos os jogadores devem ver a nova versão instantaneamente, sem que o cache do navegador ou da CDN sirva a versão antiga ("stale").

## Decisão

### 1. Imutabilidade via Content Hashing (SHA-256)
- **Regra:** Assets enviados para o R2 nunca serão sobrescritos.
- **Identificação:** O nome do arquivo no storage será o seu hash SHA-256 (ex: `assets/3a4f1...bc8.webp`).
- **Justificativa:** Se o conteúdo mudar, o hash muda, e consequentemente a URL muda. Isso torna o cache do navegador "infinito" e seguro, pois cada URL representa um conteúdo único e imutável.

### 2. Gestão de Referências no Banco de Dados
- O PostgreSQL não armazenará o arquivo, apenas a referência do hash.
- **Tabela de Assets:**
    - `id`: UUIDv7 (PK).
    - `hash`: SHA-256 (Unique Index).
    - `campaign_id`: UUIDv7 (Tenant-isolation / ADR-034).
    - `metadata`: JSONB (Dimensões, tamanho em bytes, tipo MIME).
- Se dois usuários subirem a mesma imagem idêntica, o sistema pode opcionalmente realizar a **Deduplicação**, apontando ambos para o mesmo hash no R2.

### 3. Cache Control e CDN (Cloudflare)
- **Headers:** Todos os assets servidos pelo R2/CDN terão o header `Cache-Control: public, max-age=31536000, immutable`.
- **Invalidation:** Não é necessário realizar "Purge" manual na CDN, pois novas versões de um asset terão novas URLs.

### 4. Otimização de Assets (Pipeline)
- O `apps/backend` será responsável por processar assets antes do upload final:
    - Conversão automática de imagens para **WebP/AVIF** para reduzir o tamanho.
    - Geração de **BlurHash** ou **Thumbnails** de baixa resolução para carregamento progressivo (LCP improvement).

### 5. Ciclo de Vida e Limpeza (Garbage Collection)
- **Substituição de Versão:** Quando um usuário atualiza um asset (ex: nova versão de um mapa), o sistema gera um novo Hash. A versão antiga permanece no storage até ser identificada como órfã.
- **Processo de Cleanup (GC):** Um serviço de background executará periodicamente uma varredura para identificar hashes no Cloudflare R2 que **não possuem referências** em nenhuma tabela do banco de dados (Cenas, Mapas, Personagens).
- **Segurança (Grace Period):** Para evitar a deleção de arquivos em trânsito (recém-enviados mas ainda não persistidos no banco), o GC apenas removerá objetos com data de criação superior a **12 horas**.
- **Deduplicação Global:** O arquivo físico só é removido se nenhuma outra campanha (mesmo de usuários diferentes, se a deduplicação estiver ativa) o referenciar.

## Justificativa
- **Performance:** URLs imutáveis permitem que o navegador nunca precise re-baixar um mapa que já está no cache local.
- **Custo e Sustentabilidade:** O processo de GC garante que versões antigas e inúteis de mapas 4K não ocupem espaço permanentemente, mantendo o custo de storage controlado.
- **Simplicidade:** Elimina a lógica complexa de "cache invalidation" por tempo ou tags.

## Consequências
- **Positivas:** Carregamento de mapas instantâneo para jogadores que já os acessaram; controle automático de crescimento do storage.
- **Negativas:** Adiciona complexidade na infraestrutura de background jobs para gerenciar a limpeza de forma segura e atômica.

## Referências
- **ADR-001:** Escolha da Stack (Cloudflare R2).
- **ADR-018:** Gestão de Cotas de Recursos.
- **ADR-034:** Resiliência de Dados (UUIDv7 e Tenant-isolation).
