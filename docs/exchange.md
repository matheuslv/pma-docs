# Exchange

Repositório: [pma-exchange-service](https://github.com/insper-plat-projeto/pma-exchange-service). Python 3.12, FastAPI, `requests`, Uvicorn.

O Exchange devolve a cotação atual entre duas moedas, consultando uma API de terceiros. Como o enunciado pede, é implementado em Python e só responde a usuários autenticados.

## Endpoint

`GET /exchanges/{from}/{to}`, por exemplo `GET /exchanges/USD/BRL`:

``` json
{
    "sell": 5.9351,
    "buy": 5.9346,
    "date": "2026-11-19 18:08:12",
    "id-account": "0195ae95-5be7-7dd3-b35d-7a7d87c404fb"
}
```

| Caso | Resposta |
|---|---|
| sucesso | `200` |
| sem o header `id-account` | `401` |
| código de moeda que não tem 3 letras | `422` (validação do FastAPI, `^[A-Za-z]{3}$`) |
| par que a API não conhece (ex.: `USD/XYZ`) | `404` |
| API de terceiros fora do ar, lenta ou com resposta inesperada | `502` |
| `GET /exchanges/health-check` | `200 {"status": "up"}` |
| `GET /exchanges/metrics` | métricas para o Prometheus |

## De onde vem a cotação

A fonte é a [AwesomeAPI](https://docs.awesomeapi.com.br/api-de-moedas) (`GET https://economia.awesomeapi.com.br/json/last/USD-BRL`), gratuita e sem chave:

| AwesomeAPI | Resposta do Exchange | Significado |
|---|---|---|
| `bid` | `buy` | quanto o mercado paga pela moeda |
| `ask` | `sell` | quanto o mercado cobra pela moeda |
| `create_date` | `date` | momento da cotação |

A URL e o timeout (5 s) vêm das variáveis `AWESOMEAPI_URL` e `AWESOMEAPI_TIMEOUT`. Assim, se a AwesomeAPI bloquear requisições vindas da AWS (o curso avisa que algumas APIs gratuitas fazem isso), dá para trocar de fonte sem mexer no código. A alternativa indicada pelo curso é a PTAX do Banco Central.

## Autenticação

O Exchange não valida JWT: ele confia no header `id-account`, que só o Gateway coloca depois de resolver o token no Auth. O Gateway também remove qualquer `id-account` enviado pelo cliente, então um usuário não consegue forjá-lo. Sem o header, a resposta é `401`. O serviço não publica porta: só é alcançado em `http://exchange:8080`, dentro da rede.

## Uso na loja

A [loja](web.md) mostra os preços em reais e em dólares. O valor em dólar vem de `GET /exchanges/BRL/USD` (usa o `sell`, o lado que o cliente pagaria). Se o Exchange falhar, a loja continua funcionando e mostra só os preços em reais.

## Testes

`test_main.py` tem 8 testes com `pytest`. A AwesomeAPI é substituída por um mock, então os testes não usam a rede:

| Teste | Verifica |
|---|---|
| `test_health_check` | `200` |
| `test_exchange_returns_rate` | `bid` → `buy`, `ask` → `sell`, `date` e `id-account` |
| `test_lowercase_codes_are_accepted` | `usd/eur` funciona como `USD/EUR` |
| `test_missing_id_account_is_unauthorized` | `401` |
| `test_invalid_currency_code_is_rejected` | `422` |
| `test_unknown_pair_is_not_found` | `404` |
| `test_provider_down_is_bad_gateway` | `502` quando a API de terceiros não responde |
| `test_metrics_count_requests_by_route` | o endpoint de métricas conta por rota (`/exchanges/{from_currency}/{to_currency}`), não por par de moedas |

``` bash
cd api/exchange-service
python -m venv .venv
.venv/Scripts/activate          # Linux/Mac: source .venv/bin/activate
pip install -r requirements-dev.txt
pytest                          # 8 passed
```

## Container

``` dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY main.py .
EXPOSE 8080
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```

Porta 8080, como os serviços Java, para o Gateway e os manifestos do Kubernetes tratarem todos do mesmo jeito.
