# VoeBem Analytics — Pipeline de Dados com Arquitetura Medalhão

Pipeline completo de Engenharia de Dados construído com dados públicos de voos do Brasil, disponibilizados pela ANAC através do VRA (Voos Regulares Aéreos). O projeto cobre o período de agosto de 2025 a julho de 2026 e foi desenvolvido na plataforma Databricks durante a Imersão de Engenharia de Dados com IA da Alura.

---

## Arquitetura

O pipeline segue a arquitetura Medalhão (Bronze, Silver, Gold), organizada no Unity Catalog do Databricks.

```
Bronze → Silver → Gold → Genie Agent
```

### Bronze
Ingestão dos dados brutos do VRA/ANAC sem transformações. Preserva o dado original como fonte de verdade.

### Silver
Camada de tratamento e qualidade:
- Tipagem das colunas
- Tratamento de valores nulos
- Cálculo do atraso real de partida e chegada
- Cálculo de minutos recuperados em voo
- Metadados de governança (data de carga, versão do pipeline)
- **Data Quality com Spark Declarative Pipelines**: registros inválidos (como códigos ICAO vazios) são isolados em quarentena em vez de contaminar a camada seguinte

### Gold
Tabela analítica no formato **One Big Table (OBT)**: `voebem.gold.obt_voos`

Desnormalizada e pronta para consumo, reunindo fatos e dimensões em uma única tabela. Cobre métricas de pontualidade, atraso, cancelamento e recuperação em voo por companhia, aeroporto, rota, hora do dia e escopo (doméstico/internacional).

### Genie Agent
Agente de IA configurado em cima da camada Gold. Recebe as regras de negócio, definições de métricas e exemplos de consultas SQL. Responde perguntas sobre os dados em linguagem natural, sem necessidade de escrever queries manualmente.

Exemplos de perguntas respondidas pelo agente:
- *Quais aeroportos concentram os maiores atrasos de partida no Brasil?*
- *Como o atraso evolui ao longo do dia?*
- *Qual companhia entrega melhor pontualidade e menor taxa de cancelamento?*
- *Voos internacionais atrasam mais que domésticos?*
- *Quanto atraso as companhias recuperam em voo?*

---

## Estrutura do Repositório

```
├── notebooks/          # Notebooks de cada camada
├── pipelines/          # Spark Declarative Pipelines (Data Quality)
│   └── gold_pipeline/
│       └── transformations/
├── sql/                # Consultas SQL auxiliares
├── scripts/            # Scripts de apoio
├── genie/              # Configuração do Genie Agent
├── docs/               # Documentação adicional
└── dados/              # Referências e metadados dos dados fonte
```

---

## Como Executar

O projeto roda no ambiente Databricks. Os notebooks dependem do Spark e não devem ser executados como scripts Python locais.

**Pré-requisito:** ter permissão para criar catálogo, schemas e volume no seu workspace Databricks. O código usa o catálogo `voebem`.

1. Execute `sql/00_preparar_ambiente.sql` para criar o catálogo, schemas e volumes
2. Envie os CSVs de `dados/vra/` para `/Volumes/voebem/bronze/arquivos/vra/` e os de `dados/referencias/` para `/Volumes/voebem/bronze/arquivos/referencias/`
3. Importe os arquivos de `notebooks/` como notebooks Databricks
4. Execute na ordem:
   - `notebooks/03_bronze_vra.py`
   - `notebooks/04_bronze_referencias.py`
   - `notebooks/05_silver_espelho.py`
   - Pipeline de qualidade com os arquivos de `pipelines/qualidade/` (schema `voebem.silver`)
   - `sql/gold/01_dim_aeroporto.sql` → `02_fato_voos.sql` → `03_obt_voos.sql`
   - `notebooks/09_governanca_gold.py`
5. Opcional: configure o Genie Agent com `voebem.gold.obt_voos` usando os exemplos em `genie/`

---

## Tecnologias

- **Databricks** (Free Edition)
- **Apache Spark / PySpark**
- **Spark Declarative Pipelines**
- **Delta Lake**
- **Unity Catalog**
- **SQL**
- **Genie Agents**

---

## Fonte dos Dados

Dados públicos do VRA (Voos Regulares Aéreos) disponibilizados pela ANAC:
[gov.br/anac — Histórico de Voos](https://www.gov.br/anac/pt-br/assuntos/dados-e-estatisticas/historico-de-voos)

Período coberto: agosto de 2025 a julho de 2026.
