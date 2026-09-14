# Atividade 2 - Pipeline Data Lake

## Sobre a atividade

Nesta atividade foi desenvolvido um pipeline de Data Lake para trabalhar com dados de vendas de ingressos utilizando serviços da AWS.

O pipeline foi desenvolvido em Python e executado no Google Colab, utilizando o Amazon S3 para armazenamento dos dados e o Amazon Athena para consulta e validação.

A estrutura foi organizada seguindo as camadas Raw, Silver e Gold, além de uma área de quarentena para os registros que apresentaram problemas nas validações.

## Tecnologias utilizadas

- Python
- Google Colab
- Amazon S3
- Amazon Athena
- Pandas
- Boto3
- Parquet

## O que foi realizado

O pipeline contempla as seguintes etapas:

1. Geração dos dados de compradores, eventos e vendas;
2. Inserção de algumas anomalias nos dados de vendas;
3. Ingestão dos dados na camada Raw do S3;
4. Particionamento dos arquivos por data de ingestão;
5. Aplicação das regras de Data Quality;
6. Envio dos registros inválidos para a área de quarentena;
7. Enriquecimento dos dados válidos na camada Silver;
8. Criação da camada Gold com os dados agregados;
9. Criação das tabelas externas no Athena;
10. Registro das partições;
11. Realização das consultas de auditoria e validação.

## Dados gerados

Foram utilizados:
- 500 compradores;
- 50 eventos;
- 2.000 vendas de ingressos.

Foram inseridas anomalias nas vendas para testar as regras de qualidade dos dados.

Após as validações:
- 1.694 registros foram considerados válidos;
- 306 registros foram enviados para a quarentena.

## Regras de Data Quality

Foram utilizadas as seguintes validações:
- A quantidade de ingressos deve ser maior que zero;
- O `comprador_id` deve existir na tabela de compradores;
- O `evento_id` deve existir na tabela de eventos.

Os registros que não atenderam às regras foram armazenados em formato JSON na área de quarentena, junto com o motivo da rejeição.

## Estrutura do Data Lake

A estrutura criada no Amazon S3 foi organizada da seguinte forma:
```text
s3://datalake-amanda-10781761/
│
├── raw/
│   ├── compradores/
│   │   └── ingest_date=2026-09-13/
│   ├── eventos/
│   │   └── ingest_date=2026-09-13/
│   └── vendas_ingressos/
│       └── ingest_date=2026-09-13/
│
├── quarantine/
│   └── vendas_rejeitadas/
│       └── data=2026-09-13/
│
├── processed/
│   └── fato_vendas_ingressos/
│       └── ingest_date=2026-09-13/
│
├── gold/
│   └── vendas_uf_categoria/
│       └── ingest_date=2026-09-13/
│
└── athena-results/
```

Camada Raw

Os dados originais foram armazenados em formato CSV no S3 e organizados por data de ingestão utilizando o padrão de particionamento Hive:
ingest_date=YYYY-MM-DD

Camada Silver

Na camada Silver foram utilizados somente os registros válidos.
Os dados de vendas foram enriquecidos com as informações dos compradores e dos eventos por meio de JOIN.
Também foi calculado o campo:
valor_total = quantidade * preco_ingresso

Os dados foram armazenados em formato Parquet.

Camada Gold

Na camada Gold os dados foram agregados por estado e categoria do evento.

Foram calculadas as seguintes métricas:

total de vendas;
total de ingressos;
valor total vendido;
ticket médio.

Os dados também foram armazenados em formato Parquet.

Amazon Athena

Foi criado o banco de dados:

datalake_db_datalake_amanda_10781761

E as seguintes tabelas:

raw_compradores
raw_eventos
raw_vendas_ingressos
quarentena_vendas
silver_fato_vendas_ingressos
gold_vendas_uf_categoria

As partições foram registradas utilizando o comando MSCK REPAIR TABLE.

Auditoria no Athena

Para validar os arquivos da camada Raw, foi utilizada a consulta com as pseudo-colunas $path e $file_size:

SELECT
    "$path" AS arquivo,
    "$file_size" AS tamanho_bytes
FROM raw_vendas_ingressos
LIMIT 10;

O resultado apresentou o caminho do arquivo no S3 e seu tamanho em bytes.

Evidência - metadados

Conciliação dos dados

Também foi realizada uma consulta para verificar se a quantidade de registros da Raw correspondia à soma dos registros processados na Silver e enviados para a quarentena.

Resultado:

Raw: 2.000
Silver: 1.694
Quarentena: 306

1.694 + 306 = 2.000

Integridade: OK
Evidência - conciliação

Como executar
Abrir o notebook no Google Colab;
Configurar as credenciais temporárias da AWS;
Informar a região utilizada;
Executar as células do notebook em ordem;
Conferir a criação dos arquivos e partições no S3;
Conferir as tabelas e consultas no Athena.

As credenciais da AWS não fazem parte deste repositório.
