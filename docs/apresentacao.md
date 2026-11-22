# Apresentação

A [página de apresentação](https://insper.github.io/platform/pt/2026.2/exercises/project/presentation/) pede um vídeo de 1 a 2 minutos com introdução, arquitetura, demonstração, desafios (bottlenecks) e conclusão.

## Vídeo de apresentação

!!! warning "Pendente"
    O vídeo ainda não foi gravado. O link entra aqui.

## Roteiro (1 a 2 minutos)

| Tempo | Parte | Conteúdo |
|---|---|---|
| 0:00 | Introdução | Store: loja em microsserviços, grupo 11 |
| 0:10 | Arquitetura | load balancer → 2 Gateways → Auth, Account, Exchange; Postgres, Redis, Prometheus e Grafana ([Arquitetura](arquitetura.md)) |
| 0:30 | Demonstração | tela de login; criar conta; aba **Loja** com preços em BRL e USD (vindos do Exchange), carrinho e pedido; **Sair** e a página volta para o login |
| 1:00 | Bottleneck: cache | aba **Conta**: 5 consultas HIT no Redis e a conta inexistente MISS com 404 |
| 1:15 | Bottleneck: observabilidade | dashboard do Grafana com os serviços UP, requisições, latência e a taxa de acerto do cache |
| 1:30 | Bottlenecks do Douglas | load balancer com 2 réplicas e o Trivy no pipeline; HPA no EKS |
| 1:45 | Conclusão | o que funcionou e próximos passos (pipeline do Account e do Exchange, BCrypt no lugar do SHA-256) |

## Vídeo do teste de carga (Douglas)

O teste de carga no EKS com o HPA escalando o Gateway de 1 a 10 pods está na página [EKS e load test](eks.md#hpa-e-load-test).
