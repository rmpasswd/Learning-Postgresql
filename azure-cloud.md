**Azure Database for PostgreSQL**

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

   
   
   
