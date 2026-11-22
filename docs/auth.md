# Auth

!!! info "Parte do Douglas"
    O Auth é do Douglas Celestino. Esta página é a da documentação do grupo, reproduzida aqui para a documentação ficar completa.

Repositórios: [pma-auth](https://github.com/insper-plat-projeto/pma-auth) (interface) e [pma-auth-service](https://github.com/insper-plat-projeto/pma-auth-service) (implementação). Spring Boot 4, Java 25, JJWT.

## Interface e implementação

O módulo `auth` é uma biblioteca: contém só o contrato `AuthController` e os DTOs `RegisterIn`, `LoginIn` e `TokenOut`. Ele é instalado no Maven (`mvn clean install`) e usado de dois lados:

- pelo `auth-service`, que implementa a interface (`AuthResource implements AuthController`);
- por qualquer serviço que precise chamar o Auth, via `@FeignClient`, que gera o cliente HTTP a partir da mesma interface.

O `auth-service` depende da interface `account` (`store:account:1.0.0`) para chamar o Account por Feign.

## Endpoints

| Endpoint | O que faz |
|---|---|
| `POST /auth/register` | cria a conta no Account |
| `POST /auth/login` | valida as credenciais no Account (`POST /accounts/login`) e devolve o JWT em cookie |
| `GET /auth/me` | devolve a conta do `id-account` informado pelo Gateway |
| `POST /auth/solve` | valida o JWT e devolve `{idAccount}`; usado pelo Gateway |
| `GET /auth/logout` | apaga o cookie (`Max-Age=0`) |

## JWT

- Assinatura HS256 com `JWT_SECRET_KEY` (256 bits em Base64). No EKS, a chave é gerada no deploy e fica num `Secret` do Kubernetes; nunca vai para o repositório.
- Claims: `jti` = id da conta, `sub` = nome, `email`, `iss = Insper::PMA`, `nbf` e `exp` (1 hora).
- `JwtService.getId` verifica assinatura, `nbf` e `exp`; qualquer falha vira `401`.

## Por que cookie e não corpo da resposta

`login` retorna `ResponseEntity<Void>`: o token vai no header `Set-Cookie`, não no corpo.

- `HttpOnly`: o JavaScript da página não lê o cookie, então um XSS não rouba o token.
- `Secure` e `SameSite=None`: o cookie só trafega por HTTPS e pode ser enviado pelo front-end em outra origem (`credentials: 'include'`). Navegadores tratam `localhost` como contexto seguro, por isso funciona no ambiente local.
- Contrapartida: com `SameSite=None`, a proteção contra CSRF depende do CORS restrito no Gateway.
