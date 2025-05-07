# 📦 Data Analytics Comércio Exterior

## 🎯 Objetivo

Este projeto tem como foco o tratamento e organização de dados de exportação do Brasil, utilizando a **Arquitetura de Medalhão** (Medallion Architecture) para estruturar os dados em camadas: **raw**, **landing**, **silver** e **gold**. A proposta é garantir integridade, qualidade e disponibilidade dos dados para análises futuras.

---

## ⚙️ Configuração do Ambiente

O ambiente foi montado utilizando **WSL2 (Ubuntu)** e **VS Code**, com o seguinte setup:

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/WSL2
code .
pip install requests pandas
```

---

## 📅 Fonte de Dados

Os dados utilizados neste projeto foram obtidos da **base pública do Ministério da Indústria, Comércio Exterior e Serviços (MDIC)**.

### Bases Utilizadas:
- **NCM** (Nomenclatura Comum do Mercosul): 2023 e 2024
- **SH4** (Sistema Harmonizado): 2023 e 2024
- **URF**, **País**, **Unidade**, **Via de Transporte**, **NCM Unidade**

Essas bases contêm informações detalhadas sobre o comércio exterior brasileiro, como categorias de mercadorias, unidades de medida, e destinos.

---

## 🧪 Processamento e Transformações

O processo de transformação dos dados seguiu as seguintes etapas, aplicando funções importantes do `pandas`:

### 🔹 Leitura dos Arquivos CSV
- Utilização de `pandas.read_csv()`
- Configurações:
  - `encoding='latin1'` para caracteres especiais
  - `sep=';'` para arquivos do MDIC

### 🔹 Padronização dos Dados
- Conversão de colunas-chave para `string`:
```python
df["cd_pais"] = df["cd_pais"].astype(str)
df["cd_urf"] = df["cd_urf"].astype(str)
df["cd_ncm"] = df["cd_ncm"].astype(str)
```
- Renomeação e limpeza de colunas:
```python
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
df.rename(columns={"NO_PAIS": "cd_pais", "NO_NCM_POR": "cd_ncm"}, inplace=True)
```

### 🔹 Substituição de Códigos por Descrições (MERGE)
- Para tornar os dados mais legíveis, foram feitas junções usando `merge()`:
```python
df_merge = df_export.merge(df_paises, left_on='cd_pais', right_on='CO_PAIS', how='left')
```
- Essa técnica foi aplicada também para unir informações de URF, unidade de medida, via de transporte e NCM.

### 🔹 Extração e Separação de Informações
- A coluna `NO_URF`, por exemplo, foi tratada para separar tipo e nome com `str.extract()` e `str.replace()`:
```python
df['urf_codigo'] = df['NO_URF'].str.extract(r'^(\d+)\s*-\s*')
df['urf_info'] = df['NO_URF'].str.replace(r'^\d+\s*-\s*', '', regex=True)
```

### 🔹 Padronização de Valores Textuais
- Foi criada uma função customizada para padronizar textos:
```python
def padronizar_colunas(df, colunas):
    df[colunas] = df[colunas].apply(lambda x: x.str.strip().str.lower())
    return df
```

### 🔹 Remoção de Colunas e Limpeza
```python
df.drop(columns=['coluna_irrelevante'], inplace=True)
df.dropna(inplace=True)
df.drop_duplicates(inplace=True)
```

---

## 🧱 Arquitetura de Medalhão

### 🟤 RAW
- Download automático dos arquivos com `requests` e salvamento inicial em `raw_data`.

### 🟡 LANDING
- Padronização básica: renomeação de colunas, tipos e criação da coluna `data_exportacao` a partir de ano e mês.

### ⚪ SILVER
- Realização de merges com tabelas auxiliares para substituir códigos por nomes legíveis.
- Padronização geral das colunas e eliminação de colunas desnecessárias.

### 🟢 GOLD
- Consolidação dos dados: Junção dos dados de exportação e importação de diferentes anos e fontes, usando pd.concat(), resultando em tabelas mestre consolidadas para cada fluxo.
- Enriquecimento e Agregação: Aplicação de transformações finais e agregações para gerar conjuntos de dados prontos para análise e visualização. Isso inclui:
- Agrupamento por entidades chave: Agregações por país, NCM, via de transporte, etc.
- Cálculo de métricas: Soma de valores FOB, peso líquido, e outras medidas relevantes.
- Criação de indicadores: Cálculo de médias percentuais (ex: frete/seguro sobre valor FOB).

Geração de Insights Chave:

Exportação:
- Top Produtos por Valor (FOB): Identificação dos produtos (NCM) que geraram maior receita em dólares FOB.
- Evolução Mensal das Exportações: Análise do volume (kg) e valor (USD) total das exportações ao longo do tempo.
- Modal de Transporte: Distribuição e importância das diferentes vias de transporte utilizadas.
- Exportações por Localidade: Soma do valor e peso das exportações originadas de diferentes municípios e estados.
- Volume Exportado por NCM: Total de peso (kg) exportado para cada categoria de produto.

Importação:
- Importação por Estado e Ano: Análise do valor total importado por unidade federativa ao longo dos anos.
- Top Produtos Importados por Valor (FOB): Identificação dos produtos (NCM) com maior valor de importação.
- Percentual de Frete e Seguro: Cálculo da média percentual dos custos de frete e seguro em relação ao valor FOB da importação.
- Principais Países Fornecedores: Identificação dos países com maior valor total de importação.
- Evolução Mensal das Importações: Análise do volume e valor total das importações ao longo do tempo.
- Os dados nesta camada estão limpos, estruturados e agregados, prontos para consumo por ferramentas de BI, dashboards e análises mais aprofundadas.

---

## 🗂️ Estrutura de Pastas

```
Data-Analytics-Comercio-Exterior/
├── datalake/
│   ├── gold/
│   ├── landing/
│   ├── raw_data/
│   └── silver/
├── workflow/
│   ├── ingestion/
│   └── process/
│       ├── bronze/
│       │   ├── exportacao/
│       │   └── importacao/
│       ├── gold/
│       └── silver/
│           ├── exportacao_silver/
│           ├── importacao_silver/
│           └── transformacao_data_gold/
├── ingestao.md
└── README.md
```

---

## 💡 Melhorias Futuras
- Implementação de dashboards interativos
- Conexão com ferramentas de BI (Power BI, Metabase, Plotly)

---

## 📌 Status Atual
- ✅ Raw, Landing, Silver e Gold concluídos

---

Sinta-se à vontade para explorar, usar ou contribuir com o projeto!