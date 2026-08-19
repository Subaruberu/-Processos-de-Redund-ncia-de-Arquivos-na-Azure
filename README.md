# Copiando Dados para um Data Lake com Azure Data Factory

Projeto desenvolvido durante o bootcamp de Azure da [DIO](https://www.dio.me/), com o objetivo de praticar a criação de um pipeline de ingestão de dados utilizando o **Azure Data Factory (ADF)** para copiar informações de uma API pública para um **Azure Data Lake Storage Gen2 (ADLS Gen2)**.

## 📋 Índice

- [Sobre o projeto](#sobre-o-projeto)
- [Arquitetura](#arquitetura)
- [Tecnologias utilizadas](#tecnologias-utilizadas)
- [Passo a passo](#passo-a-passo)
  - [1. Criando o Data Factory](#1-criando-o-data-factory)
  - [2. Criando a Storage Account (Data Lake Gen2)](#2-criando-a-storage-account-data-lake-gen2)
  - [3. Configurando os Linked Services](#3-configurando-os-linked-services)
  - [4. Criando os Datasets](#4-criando-os-datasets)
  - [5. Montando o Pipeline](#5-montando-o-pipeline)
  - [6. Executando e validando](#6-executando-e-validando)
- [Resultado](#resultado)
- [Aprendizados](#aprendizados)
- [Referências](#referências)

## 🎯 Sobre o projeto

Este projeto demonstra, na prática, como orquestrar a movimentação de dados dentro do ecossistema Azure. O fluxo consiste em:

1. Extrair dados da **API pública do IBGE** (lista de estados do Brasil);
2. Utilizar o **Azure Data Factory** para orquestrar a cópia desses dados via uma atividade **Copy Data**;
3. Armazenar o resultado em formato JSON dentro de um **Azure Data Lake Storage Gen2**, na camada `bronze` (dados brutos).

## 🏗️ Arquitetura

```
[API IBGE - HTTP]  --->  [Azure Data Factory (Copy Activity)]  --->  [Azure Data Lake Storage Gen2 - container "bronze"]
```

**Recursos criados:**

| Recurso | Nome |
|---|---|
| Resource Group | `rg-teste-dio` |
| Data Factory | `adf-dio-test` |
| Storage Account (Data Lake Gen2) | `stadfdiotest2026` |
| Container | `bronze` |
| Linked Service (origem) | `ls_http_ibge` |
| Linked Service (destino) | `ls_adls_destino` |
| Dataset (origem) | `ds_ibge_origem` |
| Dataset (destino) | `ds_adls_destino` |
| Pipeline | `pl_copia_ibge_datalake` |
| Arquivo final | `bronze/estados.json` |

## 🛠️ Tecnologias utilizadas

- Azure Data Factory
- Azure Data Lake Storage Gen2
- API pública do IBGE — Localidades (`https://servicodados.ibge.gov.br/api/v1/localidades/estados`)
- Portal do Azure

## 🚀 Passo a passo

### 1. Criando o Data Factory

No portal do Azure, foi criado um recurso do tipo **Data Factory** (`adf-dio-test`), vinculado ao Resource Group `rg-teste-dio`, na região East US, versão V2.

![Criar Data Factory](./imagens/01-criar-data-factory.png)

Após a implantação, o Data Factory Studio foi aberto para dar início à configuração dos componentes.

![Data Factory Studio](./imagens/02-data-factory-studio.png)

### 2. Criando a Storage Account (Data Lake Gen2)

Foi criada a Storage Account **`stadfdiotest2026`**, no mesmo Resource Group, com o **namespace hierárquico habilitado** — o que a transforma em um Data Lake Storage Gen2.

![Storage Account criada](./imagens/06-storage-account-criada.png)

Dentro dela, foi criado o container **`bronze`**, que representa a camada de dados brutos (raw) do Data Lake.

![Container bronze](./imagens/07-container-bronze.png)

### 3. Configurando os Linked Services

Dentro do Data Factory Studio (aba **Manage → Linked services**), foram criados dois serviços vinculados:

**Origem — `ls_http_ibge`** (tipo HTTP), apontando para a URL base da API do IBGE:
`https://servicodados.ibge.gov.br/api/v1/localidades/estados`

![Linked Service HTTP IBGE](./imagens/03-linked-service-http-ibge.png)

**Destino — `ls_adls_destino`** (tipo Azure Data Lake Storage Gen2), conectado via chave de conta à Storage Account `stadfdiotest2026`:

![Linked Service ADLS Gen2](./imagens/04-linked-service-adls-gen2.png)

Ambos os serviços vinculados testados e criados com sucesso:

![Linked Services criados](./imagens/05-linked-services-criados.png)

### 4. Criando os Datasets

Foram criados dois **Datasets** em formato JSON:

- **`ds_ibge_origem`**: vinculado ao `ls_http_ibge`, representando os dados de origem da API
- **`ds_adls_destino`**: vinculado ao `ls_adls_destino`, apontando para o caminho `bronze/estados.json` dentro do Data Lake

### 5. Montando o Pipeline

Foi criado o pipeline **`pl_copia_ibge_datalake`**, contendo uma atividade **Copy Data**, configurada com:

- **Origem (Source)**: `ds_ibge_origem`
- **Coletor (Sink)**: `ds_adls_destino`

### 6. Executando e validando

O pipeline foi executado via **Debug**, retornando o status **"Bem-sucedido"** em cerca de 19 segundos.

## 📊 Resultado

O arquivo `estados.json` (2.39 KiB) foi copiado com sucesso da API do IBGE para o container `bronze` do Data Lake, confirmando o funcionamento ponta a ponta do pipeline.

![Resultado final - estados.json no container bronze](./imagens/08-resultado-estados-json.png)

## 📚 Aprendizados

- Estrutura básica do Azure Data Factory: Linked Services, Datasets, Pipelines e Activities
- Diferença entre uma Storage Account comum e um Data Lake Storage Gen2 (namespace hierárquico)
- Configuração de conectores HTTP para consumo de APIs públicas como origem de dados
- Organização de dados em camadas (bronze/raw) dentro de um Data Lake
- Execução e monitoramento de pipelines via Debug e Monitor

