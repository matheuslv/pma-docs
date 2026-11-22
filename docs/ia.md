# Uso de IA

A [página do projeto](https://insper.github.io/platform/pt/2026.2/exercises/project/) permite o uso parcial de IA, desde que declarado. Esta declaração cobre a minha parte (Account, Exchange, front-end, cache e observabilidade). O Douglas declara o uso dele na documentação do grupo. Ferramenta: Claude Code (Anthropic).

## Como foi usada

Todas as partes foram feitas em conjunto, a partir do conteúdo da disciplina (aulas, hands-on e o repositório de referência `repo-classes/pma262`) e sob minha supervisão. Eu defini o escopo e as decisões de cada entrega, acompanhei a implementação, revisei cada mudança, executei os testes e publiquei tudo por pull request.

Por causa do prazo curto, a IA foi usada para acelerar o processo: trechos de código, de configuração e desta documentação foram escritos por ela e, em seguida, revisados, testados e ajustados por mim.

## Onde ajudou

- Consulta à documentação da disciplina, para manter cada entrega no padrão dos hands-on.
- Escrita de trechos de código e configuração do Account, do Exchange, do cache, da observabilidade e do front-end.
- Diagnóstico de erros durante a integração com o Gateway e o load balancer, como os dois problemas de CORS descritos em [Front-end](web.md).
- Redação desta documentação.

## O que foi verificado

Cada entrega foi executada antes do merge: login, Exchange (testes com `pytest` e chamadas reais), cache (hit/miss, TTL e evict), alvos UP no Prometheus e dashboard no Grafana, e a loja inteira no navegador pelo load balancer. Os resultados estão nas páginas de cada entrega.

## O que sei defender

- Interface (`account`) versus implementação (`account-service`) e como o Auth usa o Feign para chamar o Account.
- Por que login com e-mail inexistente e com senha errada respondem o mesmo `401`, e o limite do SHA-256 sem salt.
- Como o Exchange confia no `id-account` do Gateway e por que o cliente não consegue forjá-lo.
- O ciclo do cache: miss, gravação com TTL, hits, evict; por que `404` não é cacheado e por que não há `@CachePut`.
- Por que o Prometheus puxa (*pull*) as métricas, por que o `metrics_path` precisa bater com o `base-path` e como ler a taxa de acerto no Grafana.
- Os dois bugs de CORS: o `Host` sem porta no nginx e a anotação antiga no Account.
