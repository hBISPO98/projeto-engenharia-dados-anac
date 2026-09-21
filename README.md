# ✈️ Jornada de Engenharia de Dados: Análise da Malha Aérea Brasileira com Databricks

[![Databricks](https://img.shields.io/badge/Databricks-Red?style=flat&logo=databricks&logoColor=white)](https://databricks.com/)
[![Python](https://img.shields.io/badge/Python-3670A0?style=flat&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![SQL](https://img.shields.io/badge/SQL-003B57?style=flat&logo=sqlite&logoColor=white)](https://en.wikipedia.org/wiki/SQL)
[![Apache Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?style=flat&logo=apachespark&logoColor=white)](https://spark.apache.org/)

## 📋 Sobre o Projeto
Este projeto foi desenvolvido durante a **Imersão Engenharia de Dados da Alura**. Consiste na construção de um *Data Lakehouse* para a **VoeBem Analytics**, utilizando dados abertos da **ANAC** (VRA - Voos Regulares Ativos) entre Agosto de 2025 e Julho de 2026.

O objetivo principal foi construir uma plataforma de dados robusta para analisar a pontualidade, atrasos e cancelamentos de voos na malha aérea brasileira, disponibilizando uma camada otimizada para consultas em linguagem natural via IA Generativa.

---

## 🛠️ Ferramentas Utilizadas no Databricks

* **Workspace:** Gestão, organização de pastas e versionamento dos notebooks e scripts.
* **Notebooks:** Desenvolvimento das rotinas de ingestão, tratamento e governança em PySpark e SQL.
* **Catalog (Unity Catalog):** Governança centralizada para gerenciamento de schemas, tabelas Delta e rastreabilidade automatizada de linhagem (*data lineage*).
* **Pipelines (Jobs & Workflows):** Orquestração sequencial e controle de expectativas de qualidade dos dados.
* **Genie Agents:** Camada de consumo inteligente configurada com regras de negócio para permitir perguntas em linguagem natural via IA.

---

## 📁 Estrutura do Projetos no Workspace

A organização das pastas dentro do Workspace do Databricks foi estruturada por camadas de maturidade dos dados:

```text
projeto-engenharia-dados-anac/
├── bronze/
│   ├── 03_bronze_vra
│   └── 04_bronze_referencias
├── silver/
│   ├── 05_silver_espelho
│   └── 06_silver_qualidade
└── gold/
    ├── 09_governanca_gold
    ├── 01_dim_aeroporto.sql
    ├── 02_fato_voos.sql
    └── 03_obt_voos.sql
```

---

## 🏛️ Arquitetura Medalhão e Linhagem de Dados

A Arquitetura Medalhão foi adotada para garantir isolamento de responsabilidades, auditabilidade e governança ao longo do fluxo de dados:

```text
[ Dados Abertos ANAC (CSV) ] ──> 🥉 Bronze (Raw) ──> 🥈 Silver (Tratada) ──> 🥇 Gold (OBT) ──> 🤖 Genie Agent (IA)
```

### 1. Ingestão e Camada Bronze
* **Notebooks:** `03_bronze_vra` e `04_bronze_referencias`.
* **Descrição:** Carga dos dados brutos exatamente como disponibilizados pela ANAC.
 **💡 Boas Práticas Aplicadas:**
  * **Schema-on-Read / Carga Rígida Flexível:** Leitura dos arquivos mantendo colunas como `STRING` para evitar quebras no pipeline caso o schema de origem mude.
  * **Metadados de Auditoria:** Inclusão das colunas `_dh_ingestao` e `_nome_arquivo` para rastreabilidade de origem.
  * **Idempotência:** Gravação em formato Delta Lake com sobreescrita controlada ou merge, garantindo que reexecuções não dupliquem registros.

### 2. Tratamento e Camada Silver (Governança & Qualidade)
* **Notebooks:** `05_silver_espelho` e `06_silver_qualidade`.
* **Descrição:** Padronização, limpeza e cálculo de métricas essenciais de negócio.
 **💡 Boas Práticas Aplicadas:**
  * **Tipagem Forte:** Conversão explícita de datas para `TIMESTAMP` e valores numéricos para `INT/DOUBLE`.
  * **Auditoria Zero Loss:** Mapeamento e checagem contínua para assegurar que nenhum registro válido seja descartado indevidamente entre a Bronze e a Silver.
  * **Catalogação e Dicionário de Dados:** Adição de descrições ricas em todas as colunas no Unity Catalog para facilitar a governança.

### 3. Modelagem e Camada Gold (Consumo & OBT)
* **Arquivos:** `09_governanca_gold`, `01_dim_aeroporto.sql`, `02_fato_voos.sql` e `03_obt_voos.sql`.
* **Descrição:** Estruturação dimensional evoluindo para uma visão única desnormalizada.
 **💡 Boas Práticas Aplicadas:**
  * **One Big Table (OBT):** Consolidação das tabelas de fatos e dimensões na tabela `voebem.gold.obt_voos`. A OBT elimina a necessidade de `JOINs` complexos, prevenindo alucinações de LLMs ao serem consultadas por Agentes de IA.
  * **Regras de Negócio Padronizadas:** Criação de flags binárias (ex: `partida_pontual` para atrasos $\le 15$ min) para padronizar o cálculo de métricas em qualquer ferramenta de consumo.

### 🔗 Linhagem de Dados no Unity Catalog
A imagem abaixo demonstra a linhagem automatizada pelo Unity Catalog, mapeando a origem da `obt_voos` a partir das tabelas `dim_aeroporto` e `fato_voos`:

| :---: |
| <img src="https://github.com/user-attachments/assets/ae7878ef-a1bd-4464-a06a-6621a74385fc" /> |

---

## ⚙️ Orquestração e Pipeline de Qualidade Silver

A automação do fluxo e a verificação das regras de qualidade foram configuradas utilizando os **Databricks Pipelines**, garantindo o monitoramento contínuo da quarentena e auditoria de dados:

| :---: |
| <img src="https://github.com/user-attachments/assets/b62f1c83-4486-4f51-a632-a4f0f939ae28" /> |

---

## 🤖 Consumo via Genie Agent e Exemplos de Prompts

O **Genie Agent** foi instrumentado com exemplos práticos de consultas SQL pré-definidas e instruções explícitas de contexto para permitir análises em linguagem natural.

| :---: |
| <img src="https://github.com/user-attachments/assets/1a97a291-8fd4-4f0d-946e-29be51804846" /> |

| :---: |
| <img src="https://github.com/user-attachments/assets/e4c6348c-fbf5-496a-a4d3-3fa1e7bdcc17" /> |


### 📈 Insights Extraídos pelo Agente de IA:

#### 1. Doméstico vs. Internacional
* **Voos Domésticos:** Taxa de pontualidade de **84,14%** com atraso médio de **5,13 minutos**.
* **Voos Internacionais:** Taxa de pontualidade de **74,63%** com atraso médio de **19,36 minutos** (quase 4x maior).

| :---: |
| <img src="https://github.com/user-attachments/assets/cafc1f4d-d755-4433-a76e-3a4660228f3f" /> |

| :---: |
| <img src="https://github.com/user-attachments/assets/a060da4f-77e5-49cb-96d2-bfbb94408a10" /> |

#### 2. Padrão de Degradado ao Longo do Dia (Efeito Cascata)
* **Madrugada (0h-6h):** Período mais pontual do dia, com pico de **94,08%** às 5h.
* **Noite (17h-23h):** Pior desempenho, caindo para **73,79%** às 23h devido ao acúmulo de atrasos ao longo da malha aérea.

| :---: |
| <img src="https://github.com/user-attachments/assets/999bedcc-eb08-4bc3-bb76-ca74600b5916" /> |

| :---: |
| <img src="https://github.com/user-attachments/assets/2c37f659-5e56-4aba-802b-e50e73a93cad" /> |

---

## 📖 Glossário Técnico & Decisões de Arquitetura

* **Arquitetura Medalhão:** Padrão de design de dados que organiza a informação em três camadas de qualidade crescente: Bronze (bruto/raw), Silver (limpo/tratado) e Gold (agregado/negócio).
* **Lakehouse:** Arquitetura que combina a flexibilidade e baixo custo de armazenamento dos Data Lakes com os recursos de governança, transações ACID e desempenho dos Data Warehouses.
* **Delta Lake:** Camada de armazenamento de código aberto que traz transações ACID, versionamento de dados (*Time Travel*) e alto desempenho sobre arquivos Parquet no Data Lake.
* **OBT (One Big Table):** Técnica de modelagem onde tabelas fato e dimensão são consolidadas em uma única grande tabela desnormalizada. Excelente para ferramentas de BI e consultas via IA por eliminar `JOINs`.
* **Idempotência:** Propriedade de uma rotina que permite que ela seja executada múltiplas vezes com os mesmos parâmetros de entrada sem alterar o resultado final ou gerar dados duplicados.
* **Data Lineage (Linhagem de Dados):** Mapeamento do ciclo de vida dos dados que rastreia desde a sua origem na camada Bronze até o seu destino final e consumo na camada Gold.
* **Unity Catalog:** Solução de governança unificada do Databricks que gerencia permissões de acesso, catálogo de metadados, controle de qualidade e linhagem de dados.
* **Genie Agent:** Recurso do Databricks que utiliza Inteligência Artificial Generativa para converter perguntas em linguagem natural em queries SQL precisas sobre a camada Gold.

---

## 👥 Agradecimentos

Projeto desenvolvido na **Imersão Engenharia de Dados da Alura**, sob orientação dos instrutores:
* **Lucas Mata**
* **Agnes Ruescas**
* **Oscar Meyer**
