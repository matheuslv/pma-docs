# Account

Repositórios: [pma-account](https://github.com/insper-plat-projeto/pma-account) (interface) e [pma-account-service](https://github.com/insper-plat-projeto/pma-account-service) (implementação). Spring Boot 4.1, Java 25, JPA, Flyway, PostgreSQL 17, Redis.

O Account guarda as contas dos usuários: cadastro, consulta e validação de credenciais. Ele não é chamado pelo navegador direto: o Auth fala com ele por Feign, e o Gateway encaminha `/accounts/**` só com um JWT válido.

## Interface e implementação

Mesmo padrão do Auth:

- `account` é uma biblioteca com o contrato `AccountController` (anotado com `@FeignClient(name = "account", url = "http://account:8080")`) e os DTOs `AccountIn` e `AccountOut`. É instalada com `mvn clean install`.
- `account-service` implementa o contrato (`AccountResource implements AccountController`).
- O `auth-service` usa a mesma interface para gerar o cliente HTTP: `accountController.create(...)`, `login(...)` e `findById(...)` viram chamadas para `http://account:8080`.

## Endpoints

| Endpoint | Resposta |
|---|---|
| `POST /accounts` | `201` + `Location: /accounts/{id}`; `400` se a senha tem menos de 6 caracteres ou o e-mail está vazio |
| `POST /accounts/login` | `200` com a conta; `401` para e-mail ou senha errados; `400` para senha vazia |
| `GET /accounts/{id}` | `200` com a conta (**cacheado no Redis**); `404` se não existe |
| `GET /accounts` | `200` com todas as contas |
| `DELETE /accounts/{id}` | `204`; remove também a entrada do cache |
| `GET /accounts/health-check` | `200` |
| `GET /accounts/actuator/prometheus` | métricas para o Prometheus (ver [Bottlenecks](bottlenecks.md#observability-matheus)) |

A resposta nunca traz a senha nem o hash: `AccountOut` tem só `id`, `name` e `email`.

## Camadas

``` mermaid
flowchart LR
    R[AccountResource<br/>HTTP] -->|AccountParser| S[AccountService<br/>regras + cache]
    S --> P[AccountRepository<br/>Spring Data JPA]
    P --> M[(accounts.accounts)]
    S -.->|"@Cacheable"| C[(Redis)]
```

- `AccountResource`: recebe e devolve DTOs; não tem regra de negócio.
- `AccountParser`: converte `AccountIn`/`AccountOut` no objeto de domínio `Account`.
- `AccountService`: valida a entrada, calcula o hash da senha, lança `ResponseStatusException` com o status certo e aplica o cache.
- `AccountRepository`: `JpaRepository<AccountModel, String>` + `findByEmailAndHashPassword`.
- `AccountModel`: entidade JPA da tabela.

## Banco de dados

O schema é criado pelo Flyway na subida do serviço (`src/main/resources/db/migration`):

``` sql
CREATE SCHEMA IF NOT EXISTS accounts;

CREATE TABLE accounts.accounts (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(256) NOT NULL,
    email VARCHAR(256) NOT NULL UNIQUE,
    hash_password VARCHAR(256) NOT NULL
);
```

O `id` é um UUID gerado pela aplicação. A conexão vem de variáveis de ambiente (`DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_DB`, `DATABASE_USERNAME`, `DATABASE_PASSWORD`), definidas no `compose.yaml` e, no EKS, no `Secret` do banco.

## Login

`POST /accounts/login` recebe `{email, password}`, calcula o hash da senha e procura por `email` + `hash_password`:

- O cadastro guarda o hash da senha **sem espaços nas pontas** (`trim`), então o login faz o mesmo antes de comparar. Sem isso, uma senha cadastrada como `"senha123 "` nunca bateria.
- E-mail inexistente e senha errada dão a mesma resposta (`401 Invalid credentials`), para não revelar quais e-mails têm conta.
- O token não é gerado aqui: quem chama é o Auth, que cria o JWT a partir da conta devolvida.

!!! warning "Limitação conhecida"
    O hash é SHA-256 sem salt, como no material do curso. Em produção o certo seria um algoritmo lento e com salt, como BCrypt (`spring-security-crypto`), para resistir a ataques de dicionário se o banco vazar.

## CORS

O Account não trata CORS: isso é papel do Gateway (`globalcors`). Um `@CrossOrigin` antigo, de quando o serviço era acessado direto pela porta 8080, rejeitava com `403 Invalid CORS request` a web servida em `http://localhost`, porque o Gateway repassa o header `Origin`. Ele foi removido ([pma-account-service#4](https://github.com/insper-plat-projeto/pma-account-service/pull/4)).

## Testes de ponta a ponta (2026-11-19 e 2026-11-20)

| Teste | Resultado |
|---|---|
| login com credenciais corretas | `200` com `id`, `name`, `email` |
| senha errada / e-mail inexistente | `401` / `401` |
| senha vazia | `400` |
| `GET /accounts/{id}` inexistente | `404`, e nada é gravado no Redis |
| `GET /accounts/{id}` repetido | 1ª leitura no Postgres, as seguintes no Redis (ver [Bottlenecks](bottlenecks.md#caching-matheus)) |
| `DELETE /accounts/{id}` | a chave `accounts::<id>` some do Redis |
