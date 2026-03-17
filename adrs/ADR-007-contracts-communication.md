# ADR-007: Estratégia de Contratos e Serialização (Protobuf + ConnectRPC) 🧬

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
O Apex20 exige comunicação eficiente e fortemente tipada entre múltiplos serviços e plataformas (Backend Go, Frontend Next.js, Mobile Expo). O uso de JSON puro via REST pode levar a inconsistências de tipos, falta de documentação viva e overhead de rede. Precisamos de um sistema que garanta que todos os serviços "falem a mesma língua" de forma performática.

## Decisão
Adotar **Protobuf (Protocol Buffers)** como a linguagem de definição de interface (IDL) e o **ConnectRPC** como o framework de comunicação.

1.  **Single Source of Truth**: Todos os contratos (.proto) residirão no pacote compartilhado `packages/contracts`.
2.  **Protocolo (ConnectRPC)**: Utilizar o Connect da Buf (buf.build) para as comunicações entre o Frontend/Mobile e o Backend.
    - **Vantagem**: Funciona nativamente sobre HTTP/1.1 e HTTP/2, suporta chamadas unárias e streaming, e não exige proxies complexos (como o Envoy) para funcionar no navegador.
3.  **Geração de Código**: 
    - **Backend**: Gerar código Go utilizando `protoc-gen-go` e `protoc-gen-connect-go`.
    - **Frontend/Mobile**: Gerar código TypeScript utilizando `@bufbuild/protoc-gen-es` e `@connectrpc/protoc-gen-connect-es`.
4.  **Serialização Binária**: Utilizar Protobuf binário para comunicações server-to-server e WebSocket (WS-Service) para reduzir a latência e o consumo de banda.

## Justificativa
- **Type-Safety End-to-End**: Mudanças no contrato do backend quebram o build do frontend instantaneamente, prevenindo bugs de produção.
- **Performance**: Protobuf é significativamente mais rápido e compacto que JSON, essencial para sincronização em tempo real de grid e combate.
- **Ecossistema Moderno**: O ConnectRPC resolve as dores de cabeça históricas do gRPC-Web, sendo compatível com as ferramentas de inspeção do navegador.
- **Documentação Automática**: O arquivo `.proto` serve como a documentação técnica definitiva da API.

## Consequências
- **Positivas**: Redução drástica de erros de contrato; melhor performance de rede; facilidade de manutenção de múltiplos clientes (Web/Mobile).
- **Negativas**: Requer um passo extra de geração de código durante o desenvolvimento (`pnpm run generate`); curva de aprendizado para desenvolvedores familiarizados apenas com JSON/REST tradicional.

## Alternativas Consideradas
- **REST + OpenAPI (Swagger)**: Rejeitado pela dificuldade em manter os tipos TypeScript sincronizados com o Go de forma automática e performática.
- **gRPC puro**: Rejeitado pela complexidade de infraestrutura necessária para suportar gRPC nativo no navegador (exige Envoy Proxy).
- **GraphQL**: Rejeitado pelo overhead de processamento no servidor e falta de suporte nativo a streaming binário para o WS-Service.
