# 🧩 Configuração do Change Data Capture (CDC) no SQL Server

O **CDC (Change Data Capture)** permite capturar alterações (INSERT, UPDATE e DELETE) realizadas nas tabelas de um banco de dados SQL Server, facilitando integrações em tempo real com ferramentas como o **Debezium**.

---

## ⚙️ 1. Habilitar o CDC no Banco de Dados

Execute o comando abaixo no **SQL Server Management Studio (SSMS)** para ativar o CDC no banco de dados:

```sql
EXEC sys.sp_cdc_enable_db;
```

🔍 **Dica:** Esse comando só precisa ser executado uma vez por banco de dados.

# 🧱 2. Habilitar o CDC em uma Tabela Específica

Habilite o CDC em uma tabela desejada (exemplo: Employee):

```sql
EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',         -- Nome do schema
    @source_name = 'Employee',      -- Nome da tabela
    @role_name = NULL;              -- NULL permite acesso padrão
```

💡 **Observação:**
Caso deseje restringir o acesso, substitua NULL por uma role existente no banco.

# 🧾 3. Verificar o Status do CDC

✅ Verificar se o CDC está habilitado no banco de dados:
To check if CDC is enabled for the database:

```sql
SELECT name, is_cdc_enabled
FROM sys.databases
WHERE name = 'YourDatabaseName';
```

✅ Verificar se o CDC está habilitado em uma tabela:

```sql
SELECT * FROM cdc.change_tables;
```

Se houver registros retornados, significa que há tabelas com CDC ativo.

# 🔗 Configurando o Conector Debezium para SQL Server

O Debezium é responsável por capturar as alterações registradas pelo CDC e publicá-las em um sistema de mensageria como o Kafka.

Para criar o conector, envie uma requisição POST para:

```bash
http://localhost:8083/connectors/
```

com o seguinte payload JSON:

```json
{
  "name": "sql-server-cdc-connector",
  "config": {
    "connector.class": "io.debezium.connector.sqlserver.SqlServerConnector",
    "tasks.max": "1",
    "database.hostname": "host.docker.internal",
    "database.port": "1433",
    "database.user": "",
    "database.password": "",
    "database.names": "testes",
    "topic.prefix": "sqlserver_local",
    "table.include.list": "dbo.pedido",
    "decimal.handling.mode": "string",
    "schema.history.internal.kafka.bootstrap.servers": "kafka:29092",
    "schema.history.internal.kafka.topic": "schemahistory.sqlserver_local",
    "schema.history.internal.kafka.recovery.poll.interval.ms": "5000",
    "schema.history.internal.kafka.recovery.attempts": "4",
    "snapshot.mode": "initial",
    "topic.creation.default.replication.factor": 1,
    "topic.creation.default.partitions": 1,
    "topic.creation.enable": true,
    "database.encrypt": "true",
    "database.trustServerCertificate": "true",
    "database.applicationName": "Debezium"
  }
}
```

## 🧠 Explicação dos Principais Campos

| 🏷️ **Campo**                          | 💬 **Descrição**                                                                             |
| ------------------------------------- | -------------------------------------------------------------------------------------------- |
| `connector.class`                     | Define o tipo de conector Debezium utilizado (SQL Server).                                   |
| `database.hostname`                   | Endereço do servidor SQL Server.                                                             |
| `database.port`                       | Porta de conexão (padrão: `1433`).                                                           |
| `database.user` / `database.password` | Credenciais de acesso ao banco de dados.                                                     |
| `database.dbname`                     | Nome do banco de dados que contém o CDC.                                                     |
| `database.server.name`                | Nome lógico usado para gerar os tópicos Kafka.                                               |
| `table.include.list`                  | Lista de tabelas monitoradas pelo conector.                                                  |
| `snapshot.mode`                       | Define como o snapshot inicial será realizado (`initial` captura todos os dados existentes). |
| `database.history.kafka.*`            | Configurações de histórico e persistência de schema no Kafka.                                |

---

## 🚀 Dica Final

Após configurar o conector, você pode validar se ele foi criado com sucesso:

```bash
curl -X GET http://localhost:8083/connectors/sql-server-cdc-connector/status

```
