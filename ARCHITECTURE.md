# Arquitetura do Apex20 🏗️

Este documento descreve as decisões arquiteturais e o fluxo de dados do ecossistema Apex20.

## 🏗️ Visão Geral

O sistema é construído sobre uma base **Local-first**, priorizando a experiência do usuário e a performance em tempo real. A sincronização global é orquestrada por um serviço dedicado de WebSockets.

## 🔄 Fluxo de Dados Principal

O fluxo de comunicação segue o padrão:
`Client <-> API (HTTP/gRPC) <-> Redis Pub/Sub <-> WS Service <-> Client`

1.  **Ação do Usuário:** O cliente realiza uma alteração local (Optimistic UI).
2.  **API Core:** Uma requisição HTTP (ou gRPC) é enviada para o `apps/backend` (Go Chi).
3.  **Persistência:** A API valida a regra de negócio e persiste os dados no PostgreSQL.
4.  **Pub/Sub:** Após o sucesso, a API publica uma mensagem no **Redis Pub/Sub**.
5.  **WS Service:** O `apps/ws-service` (Go WebSocket) está inscrito nos canais do Redis e recebe o evento.
6.  **Broadcast:** O serviço de WS encaminha a atualização para todos os clientes conectados na sala de jogo.

## 📦 Arquitetura Hexagonal (Backend)

O `apps/backend` e o `apps/ws-service` seguem o padrão de **Arquitetura Hexagonal (Ports and Adapters)** para garantir testabilidade e independência de infraestrutura.

- **Domain:** Entidades puras e regras de negócio essenciais.
- **Application (Services):** Orquestração de casos de uso e lógica de aplicação.
- **Infrastructure (Adapters):**
    - **Inbound:** Handlers HTTP (Chi), Handlers WebSocket, CLI.
    - **Outbound:** Repositórios SQL (sqlc), Redis Pub/Sub, Cloudflare R2 Storage.

## 🔌 Sincronização em Tempo Real

Diferente de um backend tradicional que integra WebSockets diretamente na API principal, o Apex20 utiliza um **serviço isolado (`apps/ws-service`)**.

- **Vantagem:** Escalabilidade horizontal independente do tráfego da API REST.
- **Protocolo:** Utiliza mensagens binárias (Protobuf) para reduzir o overhead de rede.

## 🛠️ Tecnologias de Base

- **Comunicação:** Protobuf (Contracts), Redis (Pub/Sub).
- **Banco de Dados:** PostgreSQL (sqlc).
- **Armazenamento:** Cloudflare R2 (Compatível com S3).
- **Frontend/Mobile:** React Query (Cache & Optimistic UI).
