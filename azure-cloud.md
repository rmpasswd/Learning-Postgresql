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



