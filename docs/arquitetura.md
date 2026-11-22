# Arquitetura

Aplicação de compra e venda em microsserviços, com autenticação JWT, câmbio de moedas, cache, observabilidade, CI/CD e deploy no Amazon EKS com autoscaling.

``` mermaid
flowchart LR
    web[Front-end<br/>nginx :80] -->|fetch, cookie JWT| lb
    subgraph store [Docker Compose / EKS]
        lb[Load balancer<br/>nginx :8080] --> gw1[gateway #1]
        lb --> gw2[gateway #2]
        gw1 & gw2 -->|/auth/**| auth
        gw1 & gw2 -->|/accounts/**| account
        gw1 & gw2 -->|/exchanges/**| exchange
        gw1 & gw2 -.->|POST /auth/solve| auth
        auth -->|Feign| account
        account --> db[(PostgreSQL)]
        account --> redis[(Redis)]
        prometheus --> account & exchange
        grafana --> prometheus
    end
    exchange --> awesome[AwesomeAPI]
```

- Todo pedido externo entra pelo **load balancer** e passa pelo **Gateway**. O Gateway valida o cookie JWT com o **Auth** e só então encaminha o pedido, com o header `id-account`.
- Os serviços internos (Account, Auth, Exchange, PostgreSQL, Redis) **não têm porta publicada**: só são alcançados dentro da rede do Compose ou do cluster.
- Rotas abertas: `POST /auth/register`, `POST /auth/login` e `GET /auth/logout`. Todas as outras exigem o cookie.
- O **Account** guarda as contas no PostgreSQL e usa o **Redis** como cache de leitura.
- O **Exchange** busca a cotação na AwesomeAPI.
- O **Prometheus** coleta as métricas do Account e do Exchange, e o **Grafana** mostra o dashboard.

## Serviços

| Serviço | Tecnologia | Responsável | Página |
|---|---|---|---|
| Gateway | Spring Cloud Gateway (WebFlux), Java 25 | Douglas | [Gateway](gateway.md) |
| Auth | Spring Boot 4, JJWT | Douglas | [Auth](auth.md) |
| Account | Spring Boot 4.1, JPA, Flyway, PostgreSQL, Redis | Matheus | [Account](account.md) |
| Exchange | Python 3.12, FastAPI | Matheus | [Exchange](exchange.md) |
| Front-end | HTML, CSS e JavaScript, nginx | Matheus | [Front-end](web.md) |

## Como rodar localmente

``` bash
git clone --recurse-submodules https://github.com/insper-plat-projeto/pma.git
cd pma/api

# os Dockerfiles copiam o target/*.jar: gerar os jars antes
(cd account && mvn -DskipTests clean install)
(cd auth && mvn -DskipTests clean install)
for m in account-service auth-service gateway-service; do (cd $m && mvn -DskipTests clean package); done

docker compose up -d --build     # API em http://localhost:8080
cd ../web && docker compose up -d   # loja em http://localhost
```

| Endereço | O que é |
|---|---|
| `http://localhost` | a loja (front-end) |
| `http://localhost:8080` | a API, pelo load balancer |
| `http://localhost:9090` | Prometheus |
| `http://localhost:3000` | Grafana (`admin` / `admin`) |
