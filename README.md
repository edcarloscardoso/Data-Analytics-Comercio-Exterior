# Data Analytics Comercio Exterior

## Objetivo  
Este projeto tem como objetivo processar e organizar dados de exportacao, seguindo a **Arquitetura de Medalhao** para estruturar as informacoes em diferentes camadas de qualidade. O processamento envolve **limpeza, padronizacao e juncao de dados** provenientes de diversas fontes, garantindo maior confiabilidade para analises futuras.  

## Configuracao do Ambiente  
Para a execucao do projeto, utilizei o **WSL2** para criar um ambiente de trabalho, configurando um ambiente virtual Python com o seguinte fluxo:  

1. Criar e ativar o ambiente virtual:  
   ```sh
   python -m venv .venv
   source .venv/bin/activate  # Linux/WSL2
   .venv\Scripts\activate  # Windows
   ```  
2. Abrir o projeto no **VS Code**:  
   ```sh
   code .
   ```  
3. Instalar as dependencias necessarias, incluindo a biblioteca `requests` para importacao de arquivos:  
   ```sh
   pip install requests pandas
   ```  

## Fonte de Dados  
Os dados utilizados neste projeto foram obtidos da **base publica do Ministerio da Industria, Comercio Exterior e Servicos (MDIC)**, disponivel no site oficial do governo brasileiro.  

### **Bases utilizadas:**  
- **NCM (Nomenclatura Comum do Mercosul):** dados de 2023 e 2024.  
- **SH4 (Sistema Harmonizado de Designacao e Codificacao de Mercadorias):** dados de 2023 e 2024.  

Essas bases contem informacoes detalhadas sobre o comercio exterior brasileiro, incluindo categorias de mercadorias e volumes de exportacao/importacao.  

Para mais detalhes, acesse o site oficial: **[Base de Dados Bruta do MDIC](#)**.  

## Processamento de Dados  
A estruturacao e limpeza dos dados seguiram as seguintes etapas:  

### **1: Leitura dos Arquivos CSV**  
- Os dados foram carregados a partir de arquivos CSV contendo informacoes de exportacao e tabelas auxiliares (paises, unidades de referencia fiscal - URF, codigos NCM).  
- Configuracoes aplicadas:  
  - **Encoding:** `latin1` (para evitar problemas com caracteres especiais).  
  - **Separador:** `;` (padrao dos arquivos originais).  

### **2: Padronizacao de Tipos de Dados**  
Para evitar inconsistencias nas juncoes, as colunas-chave foram convertidas para `string` usando `astype()`. Exemplos:  
```python
df["cd_pais"] = df["cd_pais"].astype(str)
df["cd_urf"] = df["cd_urf"].astype(str)
df["cd_ncm"] = df["cd_ncm"].astype(str)
```  

### **3: Juncao de Dados**  
Os datasets foram combinados utilizando `merge()`, substituindo codigos numericos por descricoes compreensiveis.  
- A juncao foi realizada com **left join**, preservando os registros do dataset principal.  
```python
df_merged = df1.merge(df2, on="cd_pais", how="left")
```  

### **4: Padronizacao de Nomes de Colunas**  
- **Conversao para minusculas:** `str.lower()`  
- **Remocao de espacos extras:** `str.strip()`  
- **Substituicao de espacos internos por `_`:**  
```python
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
```  

Exemplo de renomeacao:  
```python
df.rename(columns={"NO_PAIS": "cd_pais", "NO_NCM_POR": "cd_ncm"}, inplace=True)
```  

### **5: Limpeza de Dados**  
- Remocao de valores nulos: `dropna()`.  
- Exclusao de registros duplicados: `drop_duplicates()`.  

### **6: Remocao de Colunas Desnecessarias**  
Apos a juncao dos dados, foram removidas colunas que nao eram mais relevantes:  
```python
df.drop(columns=["CO_PAIS", "CO_URF"], inplace=True)
```  

### **7: Exportacao do Arquivo Final**  
O dataset tratado foi salvo no formato CSV, sem indice, garantindo a organizacao dos dados:  
```python
df.to_csv("dados_processados.csv", index=False, sep=";")
```  
A exportacao segue os principios da **Arquitetura de Medalhao**, organizando os dados em camadas de qualidade. A proxima etapa do projeto sera a implementacao da **camada Gold**, consolidando as informacoes para analises avancadas e dashboards.  

