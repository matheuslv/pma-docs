# Gateway

!!! info "Parte do Douglas"
    O Gateway é do Douglas Celestino. Esta página é a da documentação do grupo, reproduzida aqui para a documentação ficar completa.

Repositório: [pma-gateway-service](https://github.com/insper-plat-projeto/pma-gateway-service). Spring Cloud Gateway (WebFlux), Java 25.

O Gateway é o único ponto de entrada da API. Ele roteia cada pedido para o serviço certo, aplica CORS e exige um JWT válido nas rotas protegidas.

## Rotas

| Rota | Destino | Aberta? |
|---|---|---|
| `POST /auth/register`, `POST /auth/login`, `GET /auth/logout` | `http://auth:8080` | sim |
| demais `/auth/**` | `http://auth:8080` | não |
| `/accounts/**` | `http://account:8080` | não |
| `/exchanges/**` | `http://exchange:8080` | não |
| `GET /health-check` | o próprio Gateway (devolve o hostname do container) | sim |

Os nomes `auth`, `account` e `exchange` são resolvidos pelo DNS do Docker Compose e, no EKS, pelos Services do Kubernetes com os mesmos nomes.

## Autorização

``` mermaid
sequenceDiagram
    participant C as Navegador
    participant G as Gateway
    participant A as Auth
    participant S as Serviço
    C->>G: GET /accounts (cookie __store_jwt_token)
    G->>G: remove id-account enviado pelo cliente
    G->>G: RouterValidator: rota protegida?
    G->>A: POST /auth/solve {token}
    A-->>G: {idAccount} ou 401
    G->>S: GET /accounts + id-account
    S-->>C: resposta
```

- `RouterValidator` mantém a lista de rotas abertas (método + caminho).
- `AuthorizationFilter` é um `GlobalFilter`: lê o cookie, chama `/auth/solve` com `WebClient` (o Gateway é reativo, por isso não usa Feign) e acrescenta o header `id-account`.
- `RemoveRequestHeader=id-account` em `default-filters` descarta o header vindo do cliente. Sem isso, qualquer um poderia se passar por outra conta.
- Token ausente, inválido ou expirado resulta em `401`.

## Testes de ponta a ponta (Docker Compose, 2026-11-20)

| Teste | Resultado |
|---|---|
| `/health-check` 4 vezes | alterna entre 2 hostnames (2 réplicas atrás do load balancer) |
| registro e login | `200` + `Set-Cookie: __store_jwt_token=...; HttpOnly; Secure; SameSite=None` |
| `/auth/me`, `/accounts`, `/exchanges/USD/BRL` com cookie | `200`, o Exchange recebe o `id-account` correto |
| sem cookie, `id-account` forjado, senha errada, token falso | `401` em todos |
