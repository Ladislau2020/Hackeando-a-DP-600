# 🎯 Hackeando a DP-600

> Série prática de preparação para a certificação **Microsoft Certified: Fabric Analytics Engineer Associate (DP-600)**, com notebooks prontos a correr no **Microsoft Fabric** para replicares tudo o que mostro nos vídeos.

![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Lakehouse%20%7C%20Warehouse%20%7C%20Eventhouse-0078D4)
![PySpark](https://img.shields.io/badge/PySpark-Notebook-E25A1C)
![T-SQL](https://img.shields.io/badge/T--SQL-Warehouse-CC2927)
![KQL](https://img.shields.io/badge/KQL-Eventhouse-5C2D91)
![DP-600](https://img.shields.io/badge/Exame-DP--600-F9BF04)

---

## 📌 Sobre a série

Na série **Hackeando a DP-600** trabalho os tópicos do exame que considero **fundamentos**: conceitos que saem na certificação e que servem em qualquer plataforma de dados, seja o Microsoft Fabric, o Azure Databricks ou o Snowflake.

Cada episódio tem um vídeo no YouTube e um notebook neste repositório. Assim podes ver a explicação e repetir os passos no teu próprio workspace.

| # | Episódio | Notebook | Vídeo |
|---|----------|----------|-------|
| 01 | Preparar dados com **PySpark**: arquitetura medalhão, limpeza, star schema e SCD Type 2 | [`NB01_Lakehouse_PySpark_Prepare_Data`](notebooks/NB01_Lakehouse_PySpark_Prepare_Data.ipynb) | [▶️ YouTube](LINK_DO_VIDEO_01) |
| 02 | **T-SQL** no Warehouse: views, functions e stored procedures | [`NB02_Warehouse_TSQL_Views_Functions_Procedures`](notebooks/NB02_Warehouse_TSQL_Views_Functions_Procedures.ipynb) | 🔜 Em breve |
| 03 | Consultar dados com **KQL** e DAX | [`NB03_Query_KQL_and_DAX`](notebooks/NB03_Query_KQL_and_DAX.ipynb) | 🔜 Em breve |

---

## 🧱 Episódio 01: Preparar dados com PySpark

Num Lakehouse do Microsoft Fabric levamos **dados sujos da camada bronze até um star schema na camada gold**, sempre em PySpark.

### O que vais aprender

- **Arquitetura medalhão**: criar os esquemas `bronze`, `silver` e `gold`
- **Ingerir dados sujos** na bronze (customers, products, categories, subcategories, stores, vendas 2025 e 2026)
- **Limpar dados** em PySpark:
  | Problema | PySpark |
  |----------|---------|
  | Duplicados | `dropDuplicates()` |
  | Nulos | `fillna()`, `dropna()`, `coalesce()` |
  | Tipos errados | `cast()`, `try_cast` |
  | Linhas indesejadas | `filter()` |
- **Juntar tabelas com o mesmo esquema** usando `unionByName()` (append)
- **Manter a última linha por cliente** com Window Functions (`partitionBy` + ordenação pela data de modificação)
- **Denormalizar**: passar de *snowflake schema* para *star schema* com joins
- **Tipos de join**: inner, left, full e left anti
- **Agregações na camada gold** para queries usadas com frequência
- **Criar colunas novas** com `withColumn()` e construir a `dim date`
- **Slowly Changing Dimension Type 1 vs Type 2** com Delta `MERGE`, *surrogate key* vs *business key*
- **Histórico das Delta Tables**: consultar e restaurar versões anteriores

### Estrutura do notebook

| Secção | Conteúdo |
|--------|----------|
| 0 · Setup | Criar os esquemas bronze, silver e gold |
| 1 · Generate raw (bronze) data | Gerar e carregar os dados sujos |
| 2 · Clean | Duplicados, nulos, tipos errados e linhas indesejadas |
| 3 · Keep the latest row per customer | Window Functions em PySpark |
| 4 · Reshape | Denormalize, join e aggregate |
| 5 · Add the columns the business needs | `withColumn` e escrita em Delta |
| 6 · Build the star schema | Dimensões, tabela de factos e `dim date` |
| 7 · Apply new changes | SCD Type 1 vs Type 2 com Delta `MERGE` |
| 8 · Check the result | Validação final e histórico das Delta Tables |

### Do bronze ao gold

```text
BRONZE (dados como chegam)      SILVER (dados limpos)        GOLD (pronto para análise)
──────────────────────────      ─────────────────────        ──────────────────────────
customers_raw              ──►  customers              ──►   DimCustomer (SCD Type 2)
products / categories /    ──►  produtos limpos        ──►   DimProduct (denormalizada)
subcategories
stores                     ──►  stores                 ──►   DimStore
sales_2025 + sales_2026    ──►  sales (unionByName)    ──►   FactSales
                                                             DimDate
                                                             MonthlyStoreSales (agregada)
```

> ℹ️ Os nomes exatos das tabelas estão no notebook.

---

## 🚀 Como usar

### Pré-requisitos

- Um **workspace do Microsoft Fabric** com capacidade ativa (pode ser uma [trial do Fabric](https://learn.microsoft.com/fabric/fundamentals/fabric-trial)).
- Permissões para criar **Lakehouse** e **Notebooks**.

### Passo a passo

1. **Clona ou descarrega** este repositório.
2. No teu workspace do Fabric, cria um **Lakehouse** (ex.: `lh_dp600`) com **esquemas ativados** (*Lakehouse schemas*).
3. Vai a **Import → Notebook → From this computer** e importa o ficheiro `.ipynb` do episódio.
4. Abre o notebook e associa-o ao Lakehouse como **Lakehouse por defeito**.
5. Corre as células **por ordem**, a começar pelo `0 · Setup`.
6. Faz **refresh** no Explorer para ver as tabelas a aparecer em `bronze`, `silver` e `gold`.

> 💡 **Dica:** se aparecer o aviso *"compute session is open elsewhere"*, fecha os outros notebooks abertos ou volta a ligar a sessão antes de correr.

---

## 📝 Dicas para o exame DP-600

- 🧩 **Sintaxe de PySpark:** podem aparecer perguntas em que tens de **completar o código que falta**. Não precisas de decorar tudo, mas tens de perceber o que cada linha faz.
- 🔗 **Joins:** sabe escolher o join certo para cada cenário (inner, left, right, full, anti).
- 🕰️ **SCD Type 1 vs Type 2:**
  - **Type 1:** *overwrite*, só interessa o valor atual (ex.: telemóvel ou e-mail num retalhista).
  - **Type 2:** guarda o histórico com `StartDate`, `EndDate` (`9999-12-31` na versão atual) e `IsCurrent`, e usa uma *surrogate key* por versão.
- 🥇 **Camada gold:** é aqui que ficam os dados prontos para análise e as agregações mais usadas.
- 🔺 **Delta Tables:** cada operação (`CREATE OR REPLACE`, `MERGE`, `WRITE`) cria uma versão que podes consultar ou restaurar.

---

## 📂 Estrutura do repositório

```text
hackeando-a-dp-600/
├── README.md
├── notebooks/
│   ├── NB01_Lakehouse_PySpark_Prepare_Data.ipynb
│   ├── NB02_Warehouse_TSQL_Views_Functions_Procedures.ipynb
│   └── NB03_Query_KQL_and_DAX.ipynb
└── slides/            # (opcional) slides de apoio referidos nos notebooks
```

---

## 📚 Recursos oficiais

- [Página do exame DP-600 (Microsoft Learn)](https://learn.microsoft.com/credentials/certifications/fabric-analytics-engineer-associate/)
- [Guia de estudo do DP-600](https://learn.microsoft.com/credentials/certifications/resources/study-guides/dp-600)
- [Documentação do Microsoft Fabric](https://learn.microsoft.com/fabric/)
- [Lakehouse e Delta Lake no Fabric](https://learn.microsoft.com/fabric/data-engineering/lakehouse-and-delta-tables)

---

## 👤 Autor

**Ladislau André**, Microsoft Certified Trainer (MCT) e arquiteto de soluções de Data & Analytics.

- 💼 LinkedIn: [LINK_DO_LINKEDIN](LINK_DO_LINKEDIN) (a DM está sempre aberta)
- ▶️ YouTube: [LINK_DO_CANAL](LINK_DO_CANAL)

Se este repositório te ajudar na preparação, deixa uma ⭐ e partilha com quem está a estudar para a DP-600.

---

## 🤝 Contribuições

Encontraste um erro ou tens uma sugestão? Abre uma **issue** ou um **pull request**.

## ⚖️ Licença

Distribuído sob a licença [MIT](LICENSE). Os dados usados nos notebooks são **fictícios** e servem apenas para fins educativos.
