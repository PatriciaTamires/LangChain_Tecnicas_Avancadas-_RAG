# Vende+ Data Platform — Claude Code

## Contexto do projeto

Pipeline de dados da Vende+ orquestrado com Apache Airflow, rodando em ambiente local via Docker Compose.

Dois pipelines:
- **DAG 1 — Vendas:** processa CSV de pedidos diário e gera resumo por categoria em `data/output/`
- **DAG 2 — Cotação:** busca cotação USD-BRL diária via AwesomeAPI e mantém histórico em `data/cotacao/`

## Estrutura

```
dags/           # DAGs do Airflow — detectadas automaticamente
helpers/        # Funções de negócio (vendas.py, cotacao.py)
configs/        # Configurações por ambiente (settings.py)
data/           # Arquivos de dados
docs/contexto/  # E-mails e histórias de origem dos projetos
docs/specs/     # Especificações técnicas validadas
```

## Padrões obrigatórios

- Sempre usar TaskFlow API com `@task` e `@dag` — nunca PythonOperator tradicional
- `start_date` sempre com data fixa — nunca `datetime.now()`
- `catchup=False` por padrão
- Lógica de negócio sempre em `helpers/` — DAGs só orquestram
- XCom apenas para metadados pequenos (caminhos, contagens, flags) — nunca DataFrames
- URLs e caminhos via Airflow Variables — nunca hardcoded
- Credenciais via Airflow Connections — nunca no código
- dag_id e task_id sempre em snake_case com verbo: `extrair_vendas`, `carregar_resumo`
- Comentários em português
- Docstring em todas as funções de `helpers/`
- Python 3.10+

## Configuração de imports (obrigatório)

A pasta `helpers/` fica na raiz do projeto, não dentro de `dags/`. Para que os imports funcionem no container é necessário:

**No `.env`:**
```
PYTHONPATH=/opt/airflow
```

**No `docker-compose.yaml`, dentro de `x-airflow-common`:**

Em `environment:`:
```yaml
PYTHONPATH: '/opt/airflow'
```

Em `volumes:`:
```yaml
- ${AIRFLOW_PROJ_DIR:-.}/helpers:/opt/airflow/helpers
```

Sem essas configurações o Airflow não consegue resolver `from helpers.vendas import extrair_vendas` e as DAGs falham com `ModuleNotFoundError`.
