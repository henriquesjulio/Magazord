# Pipeline de ETL de uma empresa fictícia DataMart.

[![PySpark](https://img.shields.io/badge/PySpark-3.5.0-red)](https://spark.apache.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-336791)](https://www.postgresql.org/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)

Pipeline ETL para processamento de dados de transações comerciais, integrando dados externos com um DataMart em PostgreSQL.

## 📋 Visão Geral do Pipeline

1. **Extração**:
   - Dados de clientes, produtos e transações de fontes externas
   - Dados existentes do DataMart PostgreSQL
2. **Transformação**:
   - Normalização de nomes
   - Validação de tipos de dados
   - Cálculo de métricas agregadas
   - Tratamento de valores ausentes
3. **Carregamento**:
   - Atualização das tabelas dimensionais
   - Armazenamento particionado de transações
   - Tabela agregada para análise

## ⚙️ Configuração Necessária

### Banco de Dados PostgreSQL
```sql
CREATE DATABASE DataMart;
CREATE TABLE clientes(id_cliente INT PRIMARY KEY, nome_cliente VARCHAR(100), email VARCHAR(100), telefone VARCHAR(15));
CREATE TABLE produtos(id_produto INT PRIMARY KEY, nome_produto VARCHAR(100), categoria VARCHAR(50), preco DECIMAL(10,2));
CREATE TABLE transacoes(id_transacao INT PRIMARY KEY, id_cliente INT, id_produto INT, quantidade INT, data_transacao DATE);
CREATE TABLE agregada(id_cliente INT PRIMARY KEY, receita_total DECIMAL(18,2), numero_total_transacoes INT, mais_comprado INT);
```

# 🛠️ Instalação de Dependências

Instalar JDK 8 no mínimo.

Instalar Hadoop Spark

Instalar JDBC Driver do PostgreSQL:

Baixar postgresql-42.7.5.jar

Colocar em C:/Program Files/PostgreSQL/Driver/

Instalar pacotes Python:
```
bash
Copy
pip install pyspark
```

# 🚀 Execução do Pipeline
```
bash
Copy
spark-submit \
--driver-class-path "C:/Program Files/PostgreSQL/Driver/postgresql-42.7.5.jar" \
--master local[*] \
pipeline.py
```

# 🔄 Fluxo de Transformações
# 1. Normalização de Dados
python
Copy
# Capitalização de nomes
```spark.sql('''
SELECT
    id,
    CONCAT(UPPER(SUBSTRING(nome,1,1)), 
    LOWER(SUBSTRING(nome,2))) AS nome
FROM clientes''')
```

# 2. Tratamento de Dados
python
Copy
# Preenchimento de preços ausentes
```spark.sql('''
SELECT
    COALESCE(preco, AVG(preco)) AS preco
FROM produtos''')
```

# 3. Agregação de Métricas
```
python
Copy
# Cálculo da receita total
spark.sql('''
SELECT
    t.id_cliente,
    SUM(t.quantidade * p.preco) AS receita_total
FROM transacoes t
JOIN produtos p ON t.id_produto = p.id_produto''')
```

# 📊 Estrutura Final do DataMart
Tabela	Descrição	Particionamento
clientes	Informações de clientes	-
produtos	Catálogo de produtos	-
transacoes	Registros de transações diárias	Por data_referencia (YYYYMM)
agregada	Métricas consolidadas por cliente	-


# ⚠️ Configurações Especiais
```
python
Copy
spark = SparkSession.builder \
    .config("spark.sql.shuffle.partitions", "1000") \
    .config("spark.executor.memory", "4g") \
    .config("spark.driver.memory", "8g") \
    .getOrCreate()
```

# 🔍 Monitoramento
Verificar dados no PostgreSQL após execução:
```
sql
Copy
SELECT * FROM agregada ORDER BY receita_total DESC LIMIT 10;
```

# 🛑 Solução de Problemas Comuns
Erro de Memory Overflow: Aumentar configurações de memória no SparkSession

Problemas de Conexão: Verificar se o serviço PostgreSQL está ativo

Formato de Datas: Validar formato das datas nas transações (YYYY-MM-DD)

# 📌 Notas Importantes
Particionamento automático por data de referência nas transações

Merge de dados novos com existentes via UNION

Otimização de escritas usando repartição:
```
python
Copy
tbl_clientes.repartition(100).write.mode('append')...
```

OBS: Os dados de exemplo estão disponíveis no repositório magazord-plataforma/data_engineer_test
