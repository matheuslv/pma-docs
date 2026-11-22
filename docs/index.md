# Store: documentação individual

Disciplina **Plataformas, Microsserviços, DevOps e APIs**, Insper, edição **2026.2**.

## Aluno e grupo

**Matheus** ([matheuslv](https://github.com/matheuslv)), grupo 11, responsável pelo **Account**, pelo **Exchange**, pelo front-end (a loja) e por dois bottlenecks: **cache com Redis** e **observabilidade com Prometheus e Grafana**.

| Integrante | GitHub | O que fez no projeto |
|---|---|---|
| Matheus | [matheuslv](https://github.com/matheuslv) | Account (interface e serviço), Exchange (Python), front-end, cache com Redis, Prometheus e Grafana |
| Douglas Celestino | [DouglasCelestino](https://github.com/DouglasCelestino) | Gateway, Auth, load balancer, varredura de vulnerabilidades (Trivy), Jenkins, EKS, teste de carga com HPA, custos |

## Entregas do projeto

Tabela de notas da [página do projeto](https://insper.github.io/platform/pt/2026.2/exercises/project/):

| Item | Peso | Status | Onde |
|---|---|---|---|
| Gateway | 5% | :material-check: feito (Douglas) | [Gateway](gateway.md) |
| Auth | 5% | :material-check: feito (Douglas) | [Auth](auth.md) |
| Account | 5% | :material-check: feito (Matheus) | [Account](account.md) |
| Exchange | 5% | :material-check: feito (Matheus) | [Exchange](exchange.md) |
| Bottlenecks (≥ 2 por integrante) | 20% | :material-check: 2 do Matheus + 2 do Douglas | [Bottlenecks](bottlenecks.md) |
| AWS | 5% | :material-check: feito (Douglas) | [EKS e load test](eks.md) |
| EKS | 10% | :material-check: feito (Douglas) | [EKS e load test](eks.md) |
| Jenkins (CI/CD) | 10% | :material-alert: parcial: sem pipeline do `account-service` e do `exchange-service` | [CI/CD](cicd.md) |
| Teste de carga | 15% | :material-check: feito (Douglas), com vídeo | [EKS e load test](eks.md) |
| PaaS, custos e SLA | 10% | :material-check: feito (Douglas) | [Custos](custos.md) |
| MkDocs | 10% | :material-check: esta documentação | [Esta documentação](esta-documentacao.md) |

Além da tabela: o [front-end](web.md) (a loja que usa todos os serviços), a [apresentação](apresentacao.md), as [atividades individuais](atividades.md) e a [declaração de uso de IA](ia.md).

## Repositórios

| Repositório | O que tem |
|---|---|
| [insper-plat-projeto/pma](https://github.com/insper-plat-projeto/pma) | raiz: submódulos, `compose.yaml`, load balancer, Prometheus e Grafana, front-end (`web/`), Jenkins, `k8s/` e a documentação do grupo |
| [pma-account](https://github.com/insper-plat-projeto/pma-account) · [pma-account-service](https://github.com/insper-plat-projeto/pma-account-service) | interface e implementação do Account (meu) |
| [pma-exchange-service](https://github.com/insper-plat-projeto/pma-exchange-service) | Exchange em Python (meu) |
| [pma-gateway-service](https://github.com/insper-plat-projeto/pma-gateway-service) | Gateway (Douglas) |
| [pma-auth](https://github.com/insper-plat-projeto/pma-auth) · [pma-auth-service](https://github.com/insper-plat-projeto/pma-auth-service) | interface e implementação do Auth (Douglas) |
| [matheuslv/pma-docs](https://github.com/matheuslv/pma-docs) | esta documentação |

!!! note "Repositórios privados"
    Os repositórios da organização `insper-plat-projeto` são privados. O código das minhas partes está descrito, com trechos, nas páginas [Account](account.md), [Exchange](exchange.md), [Front-end](web.md) e [Bottlenecks](bottlenecks.md).
