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
- Consolidação dos dados com uso de `concat()`:
```python
df_exportacao = pd.concat([df1, df2]).reset_index().sort_values(['cd_pais'])
```
- Agrupamento final por país, pronto para uso em dashboards e análises avançadas.

---

## 🗂️ Estrutura de Pastas

```
Data-Analytics-Comercio-Exterior/
│
├── datalake/
│   ├── raw_data/
│   ├── landing/
│   ├── silver/
│   └── gold/
├── scripts/              # (Opcional)
└── README.md
```

---

## 💡 Melhorias Futuras
- Implementação de dashboards interativos
- Conexão com ferramentas de BI (Power BI, Metabase, Plotly)
- Aplicação de análises exploratórias e preditivas

---

## 📌 Status Atual
- ✅ Raw, Landing e Silver concluídos
- 🔄 Gold em desenvolvimento (exportação e importação já estruturadas)

---

Sinta-se à vontade para explorar, usar ou contribuir com o projeto!