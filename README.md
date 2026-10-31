# NovaDrive: Azure → Snowflake → dbt → Looker

An end-to-end ELT pipeline for a fictional car dealership network (**NovaDrive**). Raw CSV extracts land in Azure Blob Storage, are loaded into Snowflake through an external stage, are modeled with dbt into a tested star schema, and are served to a Looker dashboard.

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?logo=snowflake&logoColor=white)
![dbt](https://img.shields.io/badge/dbt-FF694B?logo=dbt&logoColor=white)
![Azure](https://img.shields.io/badge/Azure%20Blob%20Storage-0078D4?logo=microsoftazure&logoColor=white)
![Looker](https://img.shields.io/badge/Looker-4285F4?logo=looker&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-336791?logo=postgresql&logoColor=white)

## Architecture

```mermaid
flowchart LR
    A[CSV extracts] --> B[Azure Blob Storage<br/>container]
    B -- storage integration +<br/>external stage --> C[(Snowflake<br/>raw tables)]
    C --> D[dbt staging<br/>views]
    D --> E[dbt marts<br/>dims + incremental fact]
    E --> F[dbt analytics<br/>aggregates]
    F --> G[Looker dashboard]
```

| Layer | Tool | What happens |
|---|---|---|
| Landing | Azure Blob Storage | Raw CSVs (sales, customers, vehicles, dealerships, salespeople, cities, states) stored in a container |
| Ingestion | Snowflake | Storage integration + external stage + `COPY INTO` load raw tables ([`snowflake/setup.sql`](snowflake/setup.sql)) |
| Staging | dbt (views) | Renaming, typing (`DECIMAL(10,2)`), `INITCAP` cleanup, `COALESCE` for missing update timestamps |
| Marts | dbt (tables) | Star schema: 6 dimensions + `fct_vendas`, built **incrementally** on `data_atualizacao` |
| Analytics | dbt (tables) | Pre-aggregated tables by dealership, salesperson, vehicle and month |
| Serving | Looker | Dashboard on top of the analytics layer |

## Data model

```mermaid
erDiagram
    fct_vendas }o--|| dim_veiculos : veiculo_id
    fct_vendas }o--|| dim_clientes : cliente_id
    fct_vendas }o--|| dim_vendedores : vendedor_id
    fct_vendas }o--|| dim_concessionarias : concessionaria_id
    dim_concessionarias }o--|| dim_cidades : cidade_id
    dim_cidades }o--|| dim_estados : estado_id
```

## Data quality

- **Generic tests** on every model: `unique` / `not_null` primary keys and `relationships` from the fact to each dimension ([`_staging.yml`](models/staging/_staging.yml), [`_marts.yml`](models/marts/_marts.yml)).
- **Business-rule test**: [`assert_sale_price_within_suggested_range`](tests/assert_sale_price_within_suggested_range.sql) flags any sale above the suggested price or more than 5% below it.
- **Referential integrity by design**: the fact uses inner joins to every dimension, so orphan sales never reach reporting.

## Project structure

```
├── dbt_project.yml
├── models/
│   ├── staging/            # stg_* views + sources
│   └── marts/
│       ├── dimensions/     # dim_* tables
│       ├── facts/          # fct_vendas (incremental)
│       └── analysis/       # analise_vendas_* aggregates
├── tests/                  # singular business-rule tests
├── snowflake/setup.sql     # storage integration, stages, raw tables, COPY INTO
└── docs/images/
```

## Running it

```bash
pip install dbt-snowflake
cp profiles.example.yml ~/.dbt/profiles.yml   # then set the SNOWFLAKE_* env vars
dbt build                                      # runs models and tests in DAG order
```

Run [`snowflake/setup.sql`](snowflake/setup.sql) first to create the storage integration, stages and raw tables (replace the `<placeholders>` with your Azure tenant and storage account).

## Screenshots

| Azure container | dbt project | Snowflake objects |
|---|---|---|
| ![Azure Blob container with raw CSV files](docs/images/azure-blob-container.png) | ![dbt project tree](docs/images/dbt-project-tree.png) | ![Snowflake database objects](docs/images/snowflake-objects.png) |

### Dashboard

![Sales performance dashboard: revenue, units, average ticket and price-policy KPIs, monthly revenue, revenue by state, top dealerships, vehicle mix and top salespeople](docs/images/dashboard.png)

> **Design mockup.** Each panel maps to one dbt analytics model (`analise_vendas_temporal`, `_concessionaria`, `_veiculo`, `_vendedor`), and the "sales within price policy" KPI comes from the business-rule test. Headline totals match the sample dataset; the breakdowns are illustrative. Source: [`docs/mockup/novadrive-dashboard.html`](docs/mockup/novadrive-dashboard.html).

## Related

- [CI-CD](https://github.com/GianottiLeo/CI-CD): GitHub Actions pipeline that tests and deploys dbt models to Snowflake dev/prod.

---
*Sample CSVs come from a Udemy course dataset; all names are fictional. The dbt project was originally developed in [NovaDrive](https://github.com/GianottiLeo/NovaDrive) and consolidated here.*
