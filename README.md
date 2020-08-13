# Learning-Postgresql


---
Yello! This is me learning and 'documenting' what I learn about *PostgreSQL*. Take it as a daily ~~vlog~~ diary.

### Setting up in LinuxMint19:

Linux mint officially supports postgres v10. But postgres has v13 in beta !
[itsfoss.com](https://itsfoss.com/install-postgresql-ubuntu/) has a beginner friendly article about setting up postgres.

> Postgres (or all database softwares) has two components client and server. Also to interact with the database there are two ways: GUI (via *pgadmin*) and CommandLine (via *psql*)

![itsfoss](./images/itsfoss.png)

To use `psql` and work with the databases, you need to login as postgresql user. By default there is a superuser name *postgres*, so you need to login as *postgres* before working with `psql`.

![psql.error](./images/psql.error.png)

To make this easier, create another user with superuser privileges named $USER (your computer username). that way you are already logged in as a db user:
`sudo -u postgres createuser --superuser $USER`

PostgreSQL's official term is *roles*. More details in their [man page](https://www.postgresql.org/docs/8.1/user-manag.html).

Also there is database named after each user. Found a reason in the [internet](https://help.ubuntu.com/community/PostgreSQL) :
"Client programs, by default, connect to the local host using your Ubuntu login name and *expect* to find a database with that name too. So to make things REALLY easy, use your new superuser privileges granted above to **create a database with the same name as your login name**:"

So create a db named after $USER: `sudo -u postgres createdb $USER`

![nameddb.png](./images/usernamedb.png)

#### All (database related) commands must end with a semicolon(;) as statement terminator.

To create a database, type `CREATE DATABASE` followed by the database name of your choice.
these two words are capitalised to make the  distinction that they are SQL commands.
I'm going to create a database named... yello!

`CREATE DATABASE yello;`

( If you perfrom this command within *postgres* then it's owner will be set to *postgres*. )
As you can see I have created the yello table but didn't give semicolon, then I had to!

![yellodb](./images/yello0.png)

Yello=Yo+Hello! I'm planning to populate this database with 'greetings' of different language and also other things about the corresponding languages...

To manipulate/work with a database you have to first *connect* to it. Go to `psql --help` and read the short manual. You can also type `\?` inside psql and learn about the *backslash commands*!
![psql.help](./images/psql.help.connect.png)

Notice the default values of hostnames and  portnumber


From *inside* psql: \c yello

![](./images/cyello.png)

or,
From *outside* psql (also used to connect to remote databases): psql -h localhost -p 1234 -U mintt yello

![outside.psql](./images/outsidepsql.png)

Put the corresponding default values to successfully connect to the database.

Create table command: CREATE TABLE table_name(
							Column_name + [data type](https://www.postgresql.org/docs/9.5/datatype.html) + constraints
							)
I am creating a table with 4 columns ( id,language name, greeting and greeting phonetics):

![](./images/create.table.png)

type `\d` to see the list of tables or `\d table_name` to see inside a particualar table:

![](./images/see.table.png)

OK there are couple of things wrong in this.Looks like i did misspellings and I didn't put any [**constraints**](https://www.postgresql.org/docs/10/ddl-constraints.html). Also the id number is supposed to be incremental from 1 to n! So we should use SERIAL data type(click the 'data type' link above!).

Changing table name and column name:
`ALTER TABE IF EXISTS old_table_name
RENAME TO new_table_name
RENAME old_column_name  TO new_column_name`
the 'if exists' is to not mandatory.
And yes. you have to change table name and column name in seperate commands! look *closely*:
![](./images/renaming.png)

id column should be of *serial* data type. But changing a data type to serial is .....['hard!'](https://www.postgresql-archive.org/ALTER-TABLE-with-TYPE-serial-does-not-work-td1914514.html). So just drop the column and create again!:
`ALTER TABLE greetings ADD COLUMN id serial not null;ALTER TABLE greetings ADD PRIMARY KEYS(id);`

![](./images/fixed.type.png)

**Lesson Learned**: Plan you damn database before you implement:


![](./images/amigoscode.create.table.png)

 Also here [are](https://dataguide.prisma.io/postgresql/column-and-table-constraints) two [links](https://kb.objectrocket.com/postgresql/alter-table-add-constraint-how-to-use-constraints-sql-621) about *constraints*.





