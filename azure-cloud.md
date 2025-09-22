### Overview of Azure Database for PostgreSQL

jargons:
Cluster - a single PostgreSQL server can host multiple user databases. PostgreSQL refers to this collection of databases as a cluster  
postgres - the default database. After your server creates, you connect to the postgres database.
azure_maintenance - the database that manages service processes. You don't have direct access to this database.
azure_sys - the Query Store database. You must not modify the azure_sys database or its schemas. Changing anything in azure_sys prevents Query Store and other performance features from functioning correctly.
Schemas - A schema is a named grouping of database objects

"The amount of storage you provision defines the I/O capacity available to your Azure Database for PostgreSQL server"  
Azure Database for PostgreSQL doesn't support tablespaces: all tables are created in the main storage area i.e. PGDATA  
find out which extensions support Azure Database for PostgreSQL: `SELECT * FROM pg_available_extensions;`  

`SELECT create_extension('postgis');` to load the extension.  
- for tracking execution statistics of SQL statements: [pg_stat_statements extension](pg_stat_statements extension)

Query optimizer: parser parses the sql syntax and crates a parse tree. Then the rewriter applies rules to the query. The planner figures out best way to execute the query.  
Backend Process: PostgreSQL authenticates a new user and creates a backend server process, the client(user) then interacts only with the process when submitting the queryies and receiving result.  

**PostgreSQL shared memory**:  
1. Local memory is allocated to each process. Following are some relevant server paramters :
    a. work_mem defines memory required for sorting tuples for ORDER BY and DISTINCT operations and basically internal sort operations and hash tables. Total RAM * 0.25 / max_connections as initial value.
    b. maintenance_work_mem is memory required by vacuum and reindex
    c. autovacuum_work_mem
    d. temp_buffers defines memory for storing temporary tables.
    e. effective_cache_size defines the amount of available memory for disk caching by the operating system. Set effective_cache_size to 50% of the machine's total RAM.
2. Shared memory is used by all process and allocated at startup
   a. _shared_buffers_ defines the shared memory buffers used by the server. PostgreSQL loads pages of tables and indexes from persistent storage to a shared buffer pool, and then works on them in memory. 128MB is the default. Need to restart server if it is increased.
   b. wal_buffers : Before writing to persistent storage, PostgreSQL will store them in buffer as WAL data (Write ahead loggin). this wal_buffers paramter defines the number of disk page buffers in shared memory for write ahead logging (WAL) before writing it to persistent storage.

- important server parameters relating to memory that you could want to tune are:** shared_buffers   work_mem   effective_cache_size**

   
   
   


Now that you completed this module, you're able to:

Describe Azure Database for PostgreSQL.
Explore PostgreSQL architecture.
Understand PostgreSQL shared memory.
To learn more information about the topics covered in this module, see:

References:  
[Azure Database for PostgreSQL - Flexible Server | Microsoft Learn](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview)
[Monitoring and metrics - Azure Database for PostgreSQL - Flexible Server | Microsoft Learn  
](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-monitoring)
[Architecture and Tuning of Memory in PostgreSQL Databases | Several nines  
](https://severalnines.com/database-blog/architecture-and-tuning-memory-postgresql-databases#:%7E:text=Shared%20memory%20area%3A%20It%20is%20used%20by%20all,sorting%20tuples%20by%20ORDER%20BY%20and%20DISTINCT%20operations.)  
[Memory Tuning for workloads in PostgreSQL Flexible Server on Azure - Microsoft Tech Community
](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/memory-tuning-for-workloads-in-postgresql-flexible-server-on/ba-p/2863440)


### Client-server communication in PostgreSQL  


### Query Processing  
[link]([url](https://learn.microsoft.com/en-us/training/modules/understand-postgresql-query-process/2-identify-components))
four separate stages to executing the query:  
1. Parsing: The parser builds a query tree, seperating into identifiable parts to understand the which filters, which tables are involved.  
3. Transformation (Rewriter): Applies rules. For example: UPDATE queries don't overwrite existing rows, instead a new row is inserted, and the old row is hidden. After the transaction commits, the vacuum process can remove the hidden row.  
4. Planning: The planner creates a plan tree, with nodes representing physical operations on the data. Creates many plans, plan with lowest cost is selected. Uses statistics about tables and rows to produce accurate cost estimates. ANALYZE collects statistics about database tables and stores the results in the pg_statistic system catalog. We have to run ANALYZE if autovacuum is disabled or hasn't been run recenlty.  
5. Execution  

<img width="1532" height="391" alt="image" src="https://github.com/user-attachments/assets/99398940-e598-4774-800c-43d069eca130" />  

[**Explain Statement**](https://learn.microsoft.com/en-us/training/modules/understand-postgresql-query-process/3-understand-explain])  
We can use some parameters alongside EXPLAIN syntax: ANALYZE VERBOSE COSTS BUFFERS FORMAT  
learn about buffers [here](https://pganalyze.com/blog/5mins-explain-analyze-buffers-nested-loops).  
_Exercise:_ Look at the EXPLAIN function and how it can display the execution plan that the PostgreSQL planner generates for a supplied statement.  


### Secure Azure Database for PostgreSQL  

There are two networking options when we are setting up our azure server:  
1. Private access: the server will be under a private network with a private IP address. In network security groups, the security rules allows us to filter traffic flow _in and out of virtual network subnets and network itnerfaces_
2. Public acccess: Just like a website, server can be accessed through a public endpoint with a publicly resolvable DNS. A firewall blocks all access by default. create IP firewall rules to grant access to servers based on the originating IP.

> When you create an Azure Database for PostgreSQL flexible server you select either Private access or Public access. Once your server has been created, you cannot change your network option.


Password acces is of two types: regular password and SCRAM(Salted Challenge Response Authentication Mechanism). Two paramters, password_encryption and azure.accepted_password_auth_method, default to sram-sha-256


**Roles:**  
[official docs.](https://www.postgresql.org/docs/current/user-manag.html#:~:text=permissions%20using%20the-,concept%20of%20roles,-.%20A%20role%20can)  
Azure Database for PostgreSQL server is created with three default roles View all the server roles by executing `SELECT * FROM pg_roles;`  

To create admin users in Azure Database for PostgreSQL: 
`CREATE ROLE <new_user> WITH LOGIN NOSUPERUSER INHERIT CREATEDB CREATEROLE NOREPLICATION PASSWORD '<StrongPassword!>';
GRANT azure_pg_admin TO <new_user>;`  

`azure_pg_admin` here is a role. We can also just grant a priviledge to a user. Following does three things: creates a database, creates a new user, grants the 'connect' privileges to the database:  
```
CREATE DATABASE testdb;
CREATE ROLE <db_user> WITH LOGIN NOSUPERUSER INHERIT CREATEDB NOCREATEROLE NOREPLICATION PASSWORD '<StrongPassword!>';
GRANT CONNECT ON DATABASE testdb TO <db_user>;
```
More syntax and list of priviledges [here...](https://learn.microsoft.com/en-us/training/modules/secure-azure-database-for-postgresql/4-grant-permissions)  

**Encryption**  
- Data at rest: Encryption's always on and uses Microsoft's managed keys. The encryption uses FIPS (Federal Information Processing Standard) 140-2 validated cryptographic module and an AES (Advanced Encryption Standard) 256-bit cipher.
- Data in transit:  Transport Layer Security (TLS) and Secure Sockets Layer (SSL) by default. Flexible server supports TLS 1.2 and 1.3 and can't be disabled. ssl_min_protocol_version(default=1.2) and ssl_max_protocol_version allows to set the min and max SSL/TLS version

https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-connect-scram  
https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-connect-tls-ssl  
https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-networking


### Functions & Procedures:  

There are differences between them, however, which means there are times when you use one or the other:

Functions always return a single value, a scalar value, or a table. Stored procedures might return nothing, a single value, or multiple values.
Functions can't include DML (Data Manipulation Language) statements such as UPDATE and INSERT. Stored procedures can include any DML statement.
Functions can't include transactions, whereas stored procedures can. This restriction means that functions can't include COMMIT or ROLLBACK statements.
Functions can be used within stored procedures. A function can't call a stored procedure.
Stored procedures are a relatively new addition to PostgreSQL, whereas functions are available for some time.  


procedure name must be unique within the schema. You can however overload procedure names by giving the same name to a procedures or function with different argument types.  
Procedures take the following parameters:

name - optionally include the schema name.  
argmode - the mode of the argument. Can be IN, INOUT, or VARIADIC. The default is IN. OUT isn't supported; use INOUT instead. VARDIADIC is an undefined number of input arguments of the same type, and must be the last input arguments.  
argname - argument name.  
argtype - the argument data type.  
default_expr - a default expression (of the same type) to be used if the parameter isn't specified. Input parameters following a parameter with a default value must also have default values.  
lang_name - the language used to write the procedure. Can be sql, c, internal, or the name of a user-defined procedural language, for example, plpgsql.  

```sql
CREATE PROCEDURE myprocedure (a integer, b integer)
    LANGUAGE SQL
    AS $$
        INSERT INTO mytable VALUES (a, b);
    $$;
```
**Calling a procedure**
`CALL myprocedure(45,12);` or we can be more verbose when there are many input paramters to avoid confusion: `CALL myprocedure(a=>45, b=>12);`  

[Learn about functions](https://learn.microsoft.com/en-us/training/modules/procedures-functions-postgresql/4-create-function-azure-database-postgresql)
Official docs: https://www.postgresql.org/docs/current/functions.html

### write ahead logging, WAL  

tldr: When changes are made to the database, they are written to the log, ahead of committing them to data files. If a problem, such as loss of power, the log always holds the latest data and can be used to ensure the database is in a consistent state.  
Another benefit is that for many small updates, performance is improved as changes to tables and indexes can be written in batches, rather than individually. 

Relvent server paramters:  
commit_delay - the delay between transaction commit and flushing the log to disk.  
wal_buffers - the number of disk page buffers in shared memory for write-ahead logging (WAL).  
max_wal_size - maximum size to let the WAL grow before triggering automatic checkpoint.  
wal_writer_delay - time interval between WAL flushes performed by the WAL writer.  
wal_compression - whether full-page writes in the WAL file are compressed.  
wal_level - determines how much information is written to the WAL. The allowed values are REPLICA(default) or LOGICAL.  

The on-premises version of PostgreSQL allows the log to be used as an archive, allowing you to restore to a point in time using the configuration settings archive_mode and archive_command. Azure Database for PostgreSQL do not give access to file system but do have a Backup feature. We can also configure the retention period in days ie. how long backups should be retained for.  

**High availability**: Azure Postgres service allows us to configure high availability which provides a _standby_ server located in seperated availability zone. Data is replicated from the primary server to the standby server using PostgreSQL streaming replication in synchronous mode. High availability isn't supported for Burstable tier.  
<img width="808" height="401" alt="image" src="https://github.com/user-attachments/assets/81ba9642-251a-463f-b398-b0b9035d7d00" />

Note that this is a disaster recovery option and we cannot use the standby server in our workload(e.g. using standby server databaes as read-only). To establish a main server and read-only server  architecture, we can use a sub-pub model and configure replication between two Azure managed Postgres servers.

**Logical Decoding** is the process of extracting all persistent changes to a database's tables into a coherent, easy to understand format which can be interpreted without detailed knowledge of the database's internal state. It is used for auditing, analytics, or any other reason you might be interested in knowing what changed, and when it changed. Azure supports wal2json extension and also pglogical extension(allows logical streaming replication). [Learn More](https://learn.microsoft.com/en-us/training/modules/understand-write-ahead-logging/3-understand-replication-logical-decode)   
Exercise: [List table](https://learn.microsoft.com/en-us/training/modules/understand-write-ahead-logging/4-exercise-list-table-changes-logical-decode) changes with logical decoding.

[Learn More!](https://learn.microsoft.com/en-us/training/modules/understand-write-ahead-logging/6-summary)  











