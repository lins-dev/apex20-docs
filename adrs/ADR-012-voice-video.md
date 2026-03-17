# ADR-012: Estratégia de Voz e Vídeo Integrada (LiveKit) 🎙️📹

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
A comunicação audiovisual (AV) é um pilar fundamental da experiência de RPG de mesa. Para o Apex20, precisamos de uma solução de voz e vídeo que seja de baixíssima latência, escalável para grupos de 4 a 10 pessoas e que permita uma integração visual orgânica com o grid de jogo (ex: posicionar o vídeo do jogador próximo ao seu token ou ficha).

## Decisão
Adotar o **LiveKit** como o framework e infraestrutura de comunicação em tempo real (WebRTC).

1.  **Arquitetura SFU (Selective Forwarding Unit)**: Utilizar o LiveKit Server para gerenciar o tráfego de mídia. Diferente do P2P (Peer-to-Peer), a SFU recebe um único stream de cada participante e o distribui para os outros, reduzindo drasticamente o consumo de banda e CPU do cliente.
2.  **Tecnologia (Go-based)**: O LiveKit é escrito em Go, alinhando-se perfeitamente com a stack de backend do Apex20 e facilitando a manutenção e extensibilidade.
3.  **Headless SDKs**: Utilizar os SDKs oficiais do LiveKit para React (Web) e React Native (Mobile) para construir uma interface customizada, evitando o uso de iFrames ou componentes pré-moldados.
4.  **Integração de Metadados**: Sincronizar os IDs de participantes do LiveKit com os IDs de personagens do Apex20 para permitir automações (ex: destacar o vídeo de quem está falando ou silenciar automaticamente jogadores fora de seu turno, se configurado).

## Justificativa
- **Baixa Latência**: Focado em WebRTC puro, o LiveKit oferece uma das menores latências do mercado, essencial para a dinâmica de conversas em tempo real.
- **Customização Total**: A natureza "headless" dos SDKs permite que o vídeo seja tratado como qualquer outro componente UI do React, possibilitando efeitos visuais imersivos integrados ao tabuleiro.
- **Escalabilidade**: Suporta milhares de salas simultâneas e oferece recursos avançados como simulcast e adaptação dinâmica de rede.
- **Sinergia com Go**: Facilita a implementação de webhooks e lógica de controle de acesso dentro do nosso ecossistema existente.

## Consequências
- **Positivas**: Experiência de áudio/vídeo superior e integrada; facilidade de desenvolvimento multiplataforma (Web/Mobile); infraestrutura moderna e performática.
- **Negativas**: Exige a gestão de um serviço adicional (LiveKit Server) na infraestrutura ou o custo de uso da LiveKit Cloud em larga escala.

## Alternativas Consideradas
- **Jitsi Meet**: Rejeitado pela complexidade de customização da UI (muito focado em iFrames) e por introduzir Java/JVM na infraestrutura.
- **WebRTC P2P puro**: Rejeitado pela instabilidade em grupos com mais de 3 participantes e alto consumo de recursos no dispositivo do usuário.
- **Zoom/Discord Integrations**: Rejeitado pela falta de controle sobre a experiência do usuário e dificuldade de integração profunda com o estado do jogo.
