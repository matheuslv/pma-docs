# AWS, EKS e load test

!!! info "Parte do Douglas"
    A parte de AWS, EKS e teste de carga é do Douglas Celestino. Esta página é a da documentação do grupo, reproduzida aqui para a documentação ficar completa.

Conta AWS do grupo (`pmapi-bcc-grupo11`), acesso pelo IAM Identity Center com `aws sso login`, região `us-east-1`.

## Cluster

Criado com `eksctl`, que monta a VPC (2 zonas, subnets públicas e privadas), o control plane e um node group gerenciado:

``` bash
eksctl create cluster --name store --region us-east-1 \
  --nodegroup-name store-nodes --node-type t3.medium \
  --nodes 2 --nodes-min 2 --nodes-max 4 --managed
```

O material do curso cria a VPC por um template CloudFormation no console. O `eksctl` faz o mesmo (também usa CloudFormation) num só comando, e já instala o `metrics-server` como add-on, que o HPA precisa.

## Deploy

Manifestos em `k8s/`:

| Arquivo | Conteúdo |
|---|---|
| `data.yaml` | PostgreSQL e Redis (Deployment + Service), credenciais do banco em `Secret` |
| `services.yaml` | Account, Auth, Exchange e Gateway; Service `gateway` do tipo `LoadBalancer`; HPA do Gateway |
| `deploy.sh` | configura o kubeconfig, cria o `Secret` do JWT com chave aleatória e aplica os manifestos |
| `load-test.sh` | sobe pods geradores de carga dentro do cluster |

Os Services têm os mesmos nomes dos hosts do Compose (`account`, `auth`, `exchange`, `db`, `redis`), então o código roda sem mudança.

``` bash
export AWS_PROFILE=pma
./k8s/deploy.sh
```

## HPA e load test

``` yaml
minReplicas: 1
maxReplicas: 10
averageUtilization: 50   # % da CPU pedida (requests.cpu: 250m)
```

Três terminais, como no exemplo do curso:

``` bash
kubectl get hpa gateway --watch
kubectl get pods -l app=gateway --watch
./k8s/load-test.sh          # para: kubectl delete pod -l role=load-generator
```

## Evidências (2026-11-20)

Aplicação respondendo pelo load balancer da AWS (`*.us-east-1.elb.amazonaws.com`): `/health-check` 200, registro e login 200, `/auth/me` com cookie 200, `/accounts` sem cookie 401.

HPA durante o teste com 4 geradores de carga:

``` text
20:54:42  cpu: 0%/50%    réplicas: 1
20:54:54  cpu: 391%/50%  réplicas: 1
20:55:05  cpu: 316%/50%  réplicas: 4
          ... 8 pods do gateway em Running
```

A CPU subiu para quase 8 vezes a meta e o HPA criou pods até a média voltar perto de 50%. Ao parar a carga, ele volta a 1 réplica depois da janela de estabilização (5 minutos).

Vídeo (2min33s, narração com voz sintética Microsoft Maria; carga real no cluster, HPA de 1 a 10 pods):

<video controls width="100%" src="../evidencias/store-douglas.mp4"></video>

[Baixar o vídeo](evidencias/store-douglas.mp4)

## Limpeza

O EKS cobra por hora. Depois do vídeo:

``` bash
kubectl delete -f k8s/services.yaml   # remove o load balancer da AWS antes do cluster
eksctl delete cluster --name store --region us-east-1
```
