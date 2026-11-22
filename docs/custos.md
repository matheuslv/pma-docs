# Custos, PaaS e SLA

!!! info "Parte do Douglas"
    A análise de custos, PaaS e SLA é do Douglas Celestino. Esta página é a da documentação do grupo, reproduzida aqui para a documentação ficar completa.

## Custo mensal estimado (us-east-1, sob demanda)

Valores de lista da AWS para `us-east-1`, preço sob demanda, 730 horas por mês. São estimativas para comparação: o valor exato deve ser conferido no [AWS Pricing Calculator](https://calculator.aws/).

| Recurso | Quantidade | Preço unitário | Mês (US$) |
|---|---|---|---|
| EKS control plane | 1 cluster | 0,10/h | 73,00 |
| EC2 `t3.medium` (nós) | 2 | 0,0416/h | 60,74 |
| EBS gp3 dos nós | 2 × 80 GB | 0,08/GB-mês | 12,80 |
| NAT Gateway (criado pelo eksctl) | 1 | 0,045/h + tráfego | 32,85 |
| Load balancer do Service `gateway` | 1 | ~0,025/h + LCU | ~18,00 |
| **Total aproximado** | | | **~197** |

O que mais pesa é o control plane do EKS e o NAT Gateway, que são fixos. Os nós escalam com a carga (o node group vai de 2 a 4 nós) e o HPA escala os pods do Gateway dentro deles.

**Como reduzir:** instâncias Spot ou Graviton (`t4g`) nos nós; desligar o cluster fora do horário de uso; trocar o NAT Gateway por VPC endpoints quando o tráfego de saída for só para serviços da AWS.

## IaaS, CaaS ou PaaS

| Opção | Exemplo na AWS | Prós | Contras |
|---|---|---|---|
| Kubernetes gerenciado (atual) | EKS | portabilidade, HPA, ecossistema | cobra o control plane, mais operação |
| Containers sem servidor | ECS Fargate | sem nós para gerenciar, paga por tarefa | menos portável, sem o ecossistema do Kubernetes |
| PaaS | App Runner, Elastic Beanstalk | deploy direto da imagem, escala automática | menos controle de rede e de roteamento interno |
| Banco gerenciado | RDS PostgreSQL, ElastiCache Redis | backup, réplicas e failover prontos | custo maior que um pod no cluster |

No projeto, o banco e o Redis rodam como pods com armazenamento efêmero, o que basta para a demonstração. Em produção, o PostgreSQL iria para o RDS (Multi-AZ) e o Redis para o ElastiCache, para não perder dados quando um pod reinicia.

## SLA

SLAs publicados pela AWS (compromisso mensal de disponibilidade):

| Componente | SLA |
|---|---|
| Amazon EKS (API do control plane) | 99,95% |
| Amazon EC2 (região, instâncias em 2+ zonas) | 99,99% |
| Elastic Load Balancing | 99,99% |

A disponibilidade de uma cadeia em série é o produto das partes: 0,9995 × 0,9999 × 0,9999 ≈ **99,93%**, cerca de 30 minutos de indisponibilidade por mês. O próprio sistema reduz o risco com réplicas do Gateway em zonas diferentes, HPA e o load balancer desviando de pods com falha. Os pontos únicos de falha restantes são o PostgreSQL e o Redis com uma réplica cada.
