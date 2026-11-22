# Atividades individuais

A [página de apresentação](https://insper.github.io/platform/pt/2026.2/exercises/project/presentation/) pede, além do projeto, a documentação e o código das atividades realizadas. As atividades da seção *Exercícios → Individuais* do site e onde cada uma está:

| Atividade | Situação | Onde |
|---|---|---|
| [API de Câmbio](https://insper.github.io/platform/pt/2026.2/exercises/exchange/) | feita por mim, dentro do projeto | [Exchange](exchange.md) · `pma-exchange-service` |
| [Gargalos](https://insper.github.io/platform/pt/2026.2/exercises/bottlenecks/) | 2 feitos por mim: cache com Redis e observabilidade | [Bottlenecks](bottlenecks.md) |
| [API de Produtos](https://insper.github.io/platform/pt/2026.2/exercises/product/) | não feita | o projeto do grupo não tem serviço de produtos; o catálogo da loja fica no front-end ([Front-end](web.md)) |
| [API de Pedidos](https://insper.github.io/platform/pt/2026.2/exercises/order/) | não feita | as compras da loja ficam no navegador ([Front-end](web.md)) |
| [Jenkins](https://insper.github.io/platform/pt/2026.2/exercises/jenkins/) | feito pelo grupo (Douglas) | [CI/CD](cicd.md); ainda sem pipeline do `account-service` e do `exchange-service` |
| [MiniKube](https://insper.github.io/platform/pt/2026.2/exercises/minikube/) | feito pelo grupo (Douglas), direto no EKS | [EKS e load test](eks.md): os manifestos de `k8s/` incluem o Account e o Exchange |

## Hands-on usados no projeto

| Hands-on | Onde entrou |
|---|---|
| 1. CRUD | Account: interface + implementação, camadas Resource, Service, Repository, Parser e Model, PostgreSQL com Flyway |
| 3. Segurança | login no Account (`POST /accounts/login`), chamado pelo Auth |
| 6. Observabilidade | actuator no Account, métricas no Exchange, Prometheus e Grafana |
| 7. Balanceador de carga | nginx com 2 réplicas do Gateway (Douglas) |
| 8. Cache em memória | Redis no Account (`@Cacheable`, `@CacheEvict`, TTL) |

## O que eu fiz

- **Account:** interface `store:account:1.0.0` (contrato `AccountController` + DTOs) e implementação (Spring Boot 4.1, JPA, Flyway), com cadastro, consulta e login. Ver [Account](account.md).
- **Exchange:** serviço em Python com FastAPI sobre a AwesomeAPI, 8 testes com `pytest`, Dockerfile e métricas. Ver [Exchange](exchange.md).
- **Front-end:** a loja (`web/`), com login, a aba Conta que mostra o cache funcionando e a aba Loja com preços em BRL e USD pelo Exchange. Ver [Front-end](web.md).
- **Bottlenecks:** cache com Redis no Account e observabilidade com Prometheus e Grafana, com evidências. Ver [Bottlenecks](bottlenecks.md).
- **Integração:** submódulos, entradas no `compose.yaml` e duas correções de CORS encontradas ao ligar a loja no load balancer.
