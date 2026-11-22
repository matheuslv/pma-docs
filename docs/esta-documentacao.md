# Esta documentação

Feita com [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), a partir do [template de entrega](https://hsandmann.github.io/documentation.template/) da disciplina e da documentação do grupo no repositório `pma`. As páginas do Douglas (Gateway, Auth, CI/CD, EKS e Custos) foram reproduzidas com indicação no topo; as páginas de Account, Exchange, Front-end, Atividades, Apresentação e Uso de IA são minhas.

Repositório: [matheuslv/pma-docs](https://github.com/matheuslv/pma-docs).

## Rodar localmente

``` bash
python -m venv .venv
.venv/Scripts/activate            # Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve                      # http://127.0.0.1:8000
```

## Publicar

O workflow `.github/workflows/main.yaml` roda `mkdocs gh-deploy` a cada push na `main` e publica na branch `gh-pages`.

Configuração feita uma vez no repositório:

- **Settings → Actions → General → Workflow permissions:** *Read and write permissions*.
- **Settings → Pages:** *Deploy from a branch*, `gh-pages` / `root`.
