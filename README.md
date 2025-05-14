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

https://www.gov.br/mdic/pt-br/assuntos/comercio-exterior/estatisticas/base-de-dados-bruta

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

## 💡 Automatização de carregagamento de arquivos
- Fiz um script que automatiza o processo de carregamento de arquivos .csv de uma pasta local para um banco de dados PostgresSQL.


### 🔹 Listagem e Leitura dos Arquivos CSV

A automação inicia com a varredura da pasta `datalake/gold`, onde ficam armazenados os arquivos `.csv` prontos para carga. Utilizamos a biblioteca `os` para identificar os arquivos e garantir que apenas arquivos válidos fossem processados.

A leitura dos dados é feita com a biblioteca `pandas`, com foco inicial em uma amostra parcial de cada arquivo (cerca de 100 linhas), apenas para detectar a estrutura das colunas. Isso permitiu criar a tabela no banco de dados antes de inserir os dados por completo. Para garantir a correta interpretação de caracteres especiais, foi utilizado o encoding `latin1`.

---

### 🔹 Criação das Tabelas no Banco de Dados

Antes da carga completa, é necessário garantir que o banco esteja preparado para receber os dados. A biblioteca `SQLAlchemy` foi empregada para criar as tabelas no PostgreSQL de forma dinâmica, baseando-se no cabeçalho de cada CSV.

As colunas foram padronizadas como texto (`string`) para evitar conflitos de tipo e permitir maior flexibilidade nos dados. O nome de cada tabela é gerado automaticamente com base no nome do arquivo correspondente. Se a tabela já existir, ela é substituída — uma abordagem simples e eficaz para manter os dados atualizados.

---

### 🔹 Carga Otimizada dos Dados com COPY

Para a etapa de inserção dos dados, foi utilizada a biblioteca `psycopg2`, que permite acesso direto ao banco PostgreSQL via Python. O destaque aqui é o uso do comando nativo `COPY`, uma das formas mais rápidas e eficientes de carregar grandes volumes de dados em uma tabela.

Essa abordagem foi escolhida por sua performance superior em comparação a inserções tradicionais linha a linha. O comando `COPY` lê diretamente o conteúdo do arquivo `.csv` e insere no banco, respeitando o cabeçalho e a estrutura da tabela previamente criada.

---

### 🔹 Controle de Transações e Tratamento de Erros

Cada carga de arquivo é envolvida em uma transação. Caso ocorra qualquer erro durante o processo, a operação é revertida automaticamente com `rollback`, mantendo a integridade do banco de dados. 

Com isso, mesmo que um arquivo apresente problemas, os demais continuam sendo processados normalmente — uma estratégia importante para ambientes de dados em produção.

---

### 🔹 Encerramento e Feedback

Após o processamento de todos os arquivos, a conexão com o banco de dados é encerrada corretamente. O script fornece mensagens no terminal indicando o andamento e a conclusão de cada etapa, facilitando o acompanhamento da carga.

---

Este processo garante uma automação robusta, segura e escalável para inserção de dados no PostgreSQL, aproveitando o melhor de ferramentas como `pandas`, `SQLAlchemy` e `psycopg2`.

Com esse processo foi possível conectar o banco de dado com a ferramenta de Business Intelligence (BI) open-source para exploração, visualização e compartilhamento dos dados.

---

## 💡 Melhorias Futuras
- Implementação de dashboards interativos
- Conexão com ferramentas de BI (Power BI e Metabase)

---

## 📌 Status Atual
- ✅ Raw, Landing, Silver, Gold concluídos e automatição com banco de dados.

---

Sinta-se à vontade para explorar, usar ou contribuir com o projeto!
