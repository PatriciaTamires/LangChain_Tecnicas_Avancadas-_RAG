# Vende+ Data Platform

Plataforma de dados da Vende+ orquestrada com Apache Airflow.

## O que esse projeto faz

Dois pipelines em produção:

- **Pipeline de Vendas:** processa o arquivo de pedidos diário do time comercial e gera um resumo por categoria para a área de Operações
- **Pipeline de Cotação:** mantém um histórico atualizado da cotação USD-BRL para a área Financeira

## Estrutura do projeto

```
.
├── .cursorrules                    # Regras e skills do projeto para a IA
├── docker-compose.yaml             # Ambiente Airflow local
├── dags/                           # DAGs do Airflow
├── helpers/                        # Funções de negócio
│   ├── vendas.py
│   └── cotacao.py
├── configs/
│   └── settings.py                 # Configurações por ambiente
├── data/                           # Arquivos de dados (não versionados)
│   ├── input/
│   ├── output/
│   └── cotacao/
│       ├── raw/
│       └── consolidated/
├── docs/
│   ├── contexto/                   # Documentos de origem (e-mails, histórias)
│   └── specs/                      # Especificações técnicas validadas
└── scripts/
    └── gerar_dados.py              # Geração de dados de teste
```

## Como subir o ambiente

```bash
# Configurar o UID do usuário
echo -e "AIRFLOW_UID=$(id -u)" > .env

# Inicializar
docker compose up airflow-init

# Subir
docker compose up -d
```

Acesso: http://localhost:8080 (airflow/airflow)

## Como gerar dados de teste

```bash
python3 scripts/gerar_dados.py
```

Gera 3 dias de arquivos de vendas em `data/`.

## Fluxo de trabalho do time

1. **Contexto** chega como e-mail, história de sprint ou ata de reunião → `docs/contexto/`
2. **Especificação** é gerada a partir do contexto e revisada pelo time → `docs/specs/`
3. **Implementação** é feita a partir da spec validada
4. **Revisão** garante que o código segue os padrões do `.cursorrules`

## Ambientes

A variável `ENV` controla qual configuração está ativa:

```bash
# Desenvolvimento (padrão)
export ENV=dev

# Produção
export ENV=prod
```

## Contato

Time de Dados — dados@vendemais.com.br

## Configuração de imports

A pasta `helpers/` fica na raiz do projeto e é importada pelas DAGs. Para que os imports funcionem no container são necessárias duas configurações:

**1. No `.env`** — adicionar junto com o `AIRFLOW_UID`:
```
AIRFLOW_UID=1000
PYTHONPATH=/opt/airflow
```

**2. No `docker-compose.yaml`** — dentro do bloco `x-airflow-common`:

Em `environment:` (junto com `AIRFLOW_CONFIG`):
```yaml
PYTHONPATH: '/opt/airflow'
```

Em `volumes:` (entre `dags` e `logs`):
```yaml
- ${AIRFLOW_PROJ_DIR:-.}/helpers:/opt/airflow/helpers
```

Sem isso o Airflow não consegue resolver `from helpers.vendas import extrair_vendas` e as DAGs falham com `ModuleNotFoundError`.
