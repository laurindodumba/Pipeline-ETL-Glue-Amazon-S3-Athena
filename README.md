# Pipeline ETL com AWS Glue, Amazon S3 e Athena

Pipeline de dados na AWS que ingere um arquivo CSV de consumo de energia, transforma os dados com **AWS Glue (PySpark)**, grava o resultado em **Parquet particionado** no **Amazon S3** e disponibiliza a consulta via **Amazon Athena**.

<img width="1908" height="861" alt="Visão geral do job no AWS Glue Studio" src="https://github.com/user-attachments/assets/d195a471-2d9b-4036-98a5-614e4def0cf0" />


## Visão geral

| Item | Descrição |
|---|---|
| **Objetivo** | Automatizar a ingestão, transformação e disponibilização de dados de consumo de energia |
| **Fonte** | Arquivo `Consumo_Energia.csv` (camada *landing*) |
| **Processamento** | Job AWS Glue (PySpark) |
| **Destino** | Parquet com compressão Snappy, particionado por data (camada *Gold*) |
| **Consulta** | Amazon Athena |
| **Região** | `us-east-1` |

## Arquitetura

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐     ┌──────────┐
│  CSV bruto   │ ──▶ │  AWS Glue    │ ──▶ │  Parquet (Gold)  │ ──▶ │  Athena  │
│  S3 landing  │     │  Job PySpark │     │  S3 bk-processed │     │  (SQL)   │
└──────────────┘     └──────────────┘     └──────────────────┘     └──────────┘
```

1. O CSV é depositado no bucket de origem, pasta `landing/`.
2. O job do Glue lê o arquivo, aplica as transformações e converte os tipos.
3. O resultado é gravado em Parquet, particionado por `DATE`, no bucket de destino.
4. O Glue Data Catalog expõe a tabela para consulta SQL no Athena.

## Tecnologias

- **AWS Glue** (Glue Studio e PySpark)
- **Amazon S3** (armazenamento em camadas)
- **Amazon Athena** (consultas SQL)
- **AWS IAM** (controle de acesso)
- **Amazon CloudWatch** (logs de execução)
- **Formato Parquet + Snappy**

## Estrutura dos dados no S3

```
s3://integracaodatabrick/
└── landing/
    └── Consumo_Energia.csv            # dado bruto (origem)

s3://bk-processed/
└── Gold/
    └── DATE=AAAA-MM-DD/
        └── part-*.snappy.parquet      # dado tratado (destino)
```

## Pré-requisitos

- Conta AWS com acesso aos serviços Glue, S3, Athena e IAM
- Dois buckets S3 criados na mesma região do job (`us-east-1`): origem e destino
- Uma IAM Role para o Glue (veja a próxima seção)
- Arquivo `Consumo_Energia.csv` carregado em `landing/`

## Configuração de permissões (IAM)

O Glue assume uma **role** para acessar os buckets. Ela precisa de:

1. A política gerenciada **`AWSGlueServiceRole`**. Ela só libera S3 em buckets `aws-glue-*`, então não basta sozinha.
2. Uma política própria com acesso aos dois buckets do projeto:


3. Relação de confiança com `glue.amazonaws.com`.

> **Boas práticas:** `glue:*` em `Resource: "*"` é amplo e serve para estudo. Em produção, restrinja às ações e recursos necessários (princípio do menor privilégio).

## Como executar

1. Faça upload de `Consumo_Energia.csv` em `s3://integracaodatabrick/landing/`.
2. No console **AWS Glue → ETL jobs**, abra o job do pipeline.
3. Em **Job details**, confirme a IAM Role usada.
4. Clique em **Run** e acompanhe o status na aba **Runs**.
5. Ao finalizar, confira os arquivos em `s3://bk-processed/Gold/`.
6. Execute o crawler (ou crie a tabela manualmente) para registrar o dado no Glue Data Catalog.

## Consultando no Athena

Defina o local de resultados das consultas e rode, por exemplo:

```sql
SELECT *
FROM nome_do_banco.nome_da_tabela
LIMIT 10;
```


## Autor

**Laurindo Dumba**
