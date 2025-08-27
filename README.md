# Test_Tools — Qualidade de Código Python (Ruff, Black, isort, mypy, pre-commit)

Projeto de exemplo com **padronização, linting, formatação e tipagem**:
- **Ruff** (lint, pyflakes/pycodestyle/bugbear/pyupgrade, isort embutido se quiser)
- **Black** (formatação)
- **isort** (organização de imports, opcional se usar Ruff-isort)
- **mypy** (checagem de tipos)
- **pre-commit** (bloqueia commits/pushes fora do padrão)
- **pytest** (testes)

Layout em `src/` (padrão recomendado).

---

## Requisitos
- Python **3.11+** (ok com 3.13)
- `git` e `pip`
- (Opcional) **Make** para atalhos de comando

---

## Estrutura
```bash
test_tools/
├─ src/
│ └─ test_tools/
│ ├─ init.py
│ └─ math_ops.py
├─ tests/
│ ├─ conftest.py
│ └─ test_math_ops.py
├─ pyproject.toml
├─ .pre-commit-config.yaml
├─ requirements-dev.txt
├─ .editorconfig
├─ .gitignore
└─ README.md
```


> **Opção B**: em vez do `tests/conftest.py`, você pode instalar o pacote em modo editável:  
> `pip install -e .` (veja `[tool.setuptools]` no `pyproject.toml`).

---

## Instalação (dev)
```bash
python -m venv .venv
# Linux/Mac
source .venv/bin/activate
# Windows PowerShell
.\.venv\Scripts\Activate.ps1

pip install -U pip wheel
pip install -r requirements-dev.txt

# Hooks do pre-commit (commit e push protegidos)
pre-commit install
pre-commit install --hook-type pre-push
```

## Comandos úteis

### Lint
```bash
ruff check .            # checa
ruff check . --fix      # checa + corrige o que for seguro
```

### Formatação
```bash
black .
black --check .         # só verifica (falha se houver diferenças)
```

### Imports
```bash
isort .
isort --check-only .
```

### Tipos
```bash
mypy .
```

### Testes
```bash
pytest
pytest -v
pytest --maxfail=1 -q
```

### Cobertura
```bash
pip install pytest-cov
pytest --cov=src/test_tools --cov-report=term-missing
```

>Se optar por Ruff organizando imports, pode remover o isort e usar apenas:
>ruff check . --fix (ele cuida de lint + imports).

## Automação local (pre-commit)

.pre-commit-config.yaml roda automaticamente:

- commit: ruff, black, isort, mypy
- push: pytest (hook local de pre-push)

Rodar manualmente:
```bash
pre-commit run --all-files
```

## CI (GitHub Actions)

Exemplo de workflow (crie em .github/workflows/ci.yaml):

```bash
name: ci
on:
  pull_request:
  push:
    branches: [main]
jobs:
  quality:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: ${{ matrix.python-version }} }
      - run: |
          python -m pip install -U pip
          pip install -r requirements-dev.txt
          pip install -e .
      - run: ruff check .
      - run: black --check .
      - run: isort --check-only .
      - run: mypy .
      - run: pytest -q
```

### Protegendo a main

- Settings → Branches → Branch protection rules

- - Require pull request before merging

- - Require status checks to pass (selecione o job do Actions)

- - (Opcional) Require branches to be up to date

- - (Opcional) Require approvals

## Convenções / Configurações-chave
Ruff

- select = ["E","F","I","B","UP"] (pycodestyle, pyflakes, isort, bugbear, pyupgrade)

- ignore = ["E501"] (Black já cuida de linhas longas)

- preview = true para recursos novos

Black

- line-length = 88, target-version = "py311"

isort

- profile = "black", known_first_party = ["test_tools"], src_paths = ["src", "tests"]

mypy

- strict = true (pode relaxar conforme a necessidade)

- ignore_missing_imports = true para evitar ruído de libs sem stubs

- Ajuste python_version = "3.13" se estiver usando Python 3.13 localmente

## Troubleshooting

- ImportError em testes
Use uma das opções:

 - - tests/conftest.py adicionando src/ ao sys.path; ou

- - pip install -e . para instalar o pacote ([tool.setuptools] aponta src/).

- mypy lê config errada
Se existir mypy.ini / .mypy.ini / setup.cfg, ele ignora o pyproject.toml. Use apenas um.

- Conflito Black x isort
Garanta profile = "black" no isort (ou deixe o Ruff cuidar dos imports) e mesma line-length.

- Windows + pre-push
Prefira entry: pytest -q com language: system no hook local (evite bash -c ...).