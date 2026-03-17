# ADR-016: Estratégia de Geração de Números Aleatórios (RNG Verificável) 🎲

**Data:** 2026-03-12
**Status:** Aceito

## Contexto
Em um Virtual Tabletop (VTT), a confiança na aleatoriedade das rolagens de dados é um dos pilares da experiência do jogador. Se houver qualquer suspeita de que os resultados são previsíveis ou manipuláveis, a integridade do jogo é comprometida. Precisamos de um sistema que gere números verdadeiramente imprevisíveis e que possa ser auditado para garantir a imparcialidade do servidor.

## Decisão
Adotar o uso de **CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)** como a fonte primária de aleatoriedade para todas as mecânicas de jogo.

1.  **Tecnologia (Go crypto/rand)**: Utilizar o pacote `crypto/rand` do Go, que consome entropia do sistema operacional (`/dev/urandom` no Linux), garantindo que os resultados sejam imprevisíveis mesmo que o estado interno do gerador seja conhecido.
2.  **Autoridade do Servidor (Server-Side Rolling)**: O resultado final do dado é **sempre** gerado no Backend Go. O cliente (Frontend/Mobile) apenas recebe o resultado e executa a animação visual correspondente.
3.  **Auditabilidade (Seed Logging)**: Manter um log técnico (opcional para o usuário ativá-lo) que registre a semente e o algoritmo utilizado em cada rolagem para verificações de integridade em cenários de disputa ou auditoria.
4.  **Cláusula de Performance (Fallback Híbrido)**: Caso o monitoramento (ADR-009) indique latência excessiva ou consumo de CPU proibitivo decorrente de syscalls do kernel para entropia, o sistema poderá migrar para uma abordagem **Híbrida**: usar `crypto/rand` para gerar sementes (seeds) e alimentar um PRNG de alta performance e estatisticamente robusto (como **ChaCha20** ou **PCG**).

## Justificativa
- **Imprevisibilidade Absoluta**: O CSPRNG impede ataques de previsão de estado, onde um jogador técnico poderia tentar adivinhar os próximos resultados baseando-se em rolagens anteriores.
- **Confiança do Jogador**: Estabelece o Apex20 como uma plataforma justa e profissional, adequada para cenários competitivos ou de alta fidelidade.
- **Segurança (Anti-Exploit)**: Elimina o risco de "seeds previsíveis" baseadas em horário do sistema, um erro comum em implementações simples de RNG.

## Consequências
- **Positivas**: Máxima segurança e integridade nas rolagens; eliminação de bugs de previsibilidade; base sólida para recursos futuros de "Fair Play".
- **Negativas**: Leve overhead de CPU devido a chamadas de sistema (syscalls) para obter entropia do kernel (estimado em menos de 1% do processamento total, mas monitorado).

## Alternativas Consideradas
- **math/rand (PRNG comum)**: Rejeitado por ser previsível e não seguro para ambientes multiusuário onde a aleatoriedade é crítica.
- **Client-Side Rolling**: Rejeitado pelo risco extremo de manipulação de código no navegador (injeção de scripts para forçar 20 naturais).
