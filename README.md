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

# Principais Alterações  

## 1. Leitura dos Arquivos CSV  
Foram carregados arquivos **CSV** contendo dados de exportação e tabelas auxiliares, como **países**, **unidades de referência fiscal (URF)** e **códigos NCM**.  
- O **encoding** foi definido como `latin1` para lidar com caracteres especiais.  
- O **separador** dos arquivos foi definido como `;`.  

## 2. Padronização de Tipos de Dados  
As colunas-chave utilizadas para junção, como:  
- `cd_pais`, `CO_PAIS`, `cd_urf`, `CO_URF`, `cd_ncm`, `CO_NCM`  

foram convertidas para **string** para garantir compatibilidade durante os merges.  

## 3. Junção de Dados  
Realizou-se o **merge** entre os datasets principais e as tabelas auxiliares para substituir códigos numéricos por **nomes mais descritivos**.  

## 4. Renomeação e Padronização de Colunas  
As colunas foram padronizadas para:  
- Converter os nomes para **lowercase**.  
- Remover espaços extras utilizando `.strip()`.  
- Substituir espaços internos por **_**.  

Exemplo de mudanças de nomes:  
- `'NO_PAIS'` → `'cd_pais'`  
- `'NO_NCM_POR'` → `'cd_ncm'`  

## 5. Limpeza de Dados  
A limpeza de dados incluiu a remoção de valores **nulos** e **duplicados**, além da correção de inconsistências de formato.  

## 6. Remoção de Colunas Desnecessárias  
Após a junção dos dados, foram removidas as colunas que não eram mais necessárias para a análise, como por exemplo:  
`drop(columns=['cd_pais', 'CO_PAIS_ISON3', ...])`.  

## 7. Exportação do Arquivo Final  
O dataset tratado foi **salvo** na pasta **silver** no formato **CSV**, sem o índice, garantindo compatibilidade para uso posterior.  
