# Objetivo  

Este projeto tem como **objetivo** processar e organizar os dados de exportação, realizando a **limpeza**, **padronização** e **junção** de informações provenientes de diferentes fontes.  

Primeiramente, utilizei **WSL2** para criar uma pasta de trabalho, dentro da qual criei um ambiente Python. Após ativar o ambiente virtual (.venv), executei o comando `code .` para abrir o **VSCode**.  

Utilizei a biblioteca **Requests** para importar os arquivos que necessitam de manipulação.  

---  

# Fonte de Dados  

Os dados utilizados neste projeto foram retirados da base de dados pública do **Ministério da Indústria, Comércio Exterior e Serviços (MDIC)**, disponível no site oficial do governo brasileiro.  

- **Base de Dados NCM (Nomenclatura Comum do Mercosul):** Dados referentes aos anos **2023** e **2024**.  
- **Base de Dados SH4 (Sistema Harmonizado de Designação e Codificação de Mercadorias):** Dados referentes aos anos **2023** e **2024**.  

Essas bases contêm informações detalhadas sobre o comércio exterior brasileiro, incluindo dados de **exportações** e **importações**, com categorização das mercadorias com base nos códigos **NCM** e **SH4**.  

Para mais detalhes, consulte o site oficial:  
[Base de Dados Bruta do MDIC](https://www.gov.br/mdic/pt-br/assuntos/comercio-exterior/estatisticas/base-de-dados-bruta)  

---

# Principais Alterações  / Processamento de Dados  

## 1. Leitura dos Arquivos CSV  
Os dados foram carregados a partir de arquivos **CSV**, incluindo informações de exportação e tabelas auxiliares, como **países**, **unidades de referência fiscal (URF)** e **códigos NCM**.  
- O **encoding** foi definido como `latin1` para garantir compatibilidade com caracteres especiais.  
- O **separador** adotado foi `;`, conforme o formato dos arquivos de origem.  

## 2. Padronização de Tipos de Dados  
Para evitar inconsistências durante as junções, as colunas-chave foram convertidas para **string** utilizando a função `astype()`, garantindo compatibilidade nos merges. Exemplos de colunas padronizadas:  
- `cd_pais`, `CO_PAIS`, `cd_urf`, `CO_URF`, `cd_ncm`, `CO_NCM`.  

## 3. Junção de Dados  
Os dados foram combinados utilizando a função `merge()`, permitindo a substituição de códigos numéricos por **descrições mais compreensíveis**. A junção foi realizada no formato **left join**, preservando todos os registros do dataset principal.  

## 4. Padronização de Nomes de Colunas  
As colunas foram renomeadas e padronizadas para manter uniformidade:  
- **Conversão para lowercase** com a função `str.lower()`, garantindo um padrão único.  
- **Remoção de espaços extras** com `str.strip()` para evitar inconsistências.  
- **Substituição de espaços internos por `_`** para facilitar manipulação no código.  

A padronização foi aplicada de forma automatizada em todas as colunas relevantes.  

Exemplos de renomeação:  
- `'NO_PAIS'` → `'cd_pais'`  
- `'NO_NCM_POR'` → `'cd_ncm'`  

## 5. Limpeza de Dados  
Foram removidos valores **nulos** e **duplicados** utilizando as funções `dropna()` e `drop_duplicates()`, além da correção de formatações inconsistentes.  

## 6. Remoção de Colunas Desnecessárias  
Após a junção dos dados, colunas que não eram mais relevantes foram removidas utilizando a função `drop()`. Esse processo eliminou códigos e identificadores que já haviam sido substituídos por descrições mais compreensíveis.  

## 7. Exportação do Arquivo Final  
O dataset tratado foi **salvo** no formato **CSV**, sem índice, utilizando a função `to_csv()`. O armazenamento seguiu a **Arquitetura de Medalhão**, garantindo organização em camadas de qualidade e preservação do histórico de dados para futuras análises.  
