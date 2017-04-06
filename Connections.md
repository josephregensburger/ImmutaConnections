# Connecting Immuta to Analytics Tools

## Introduction

This document will serve as a living primer to help analysts and data scientists a
guide on how Immuta can be connected to various tools used by data scientists,
business analysts, and other analytics professionals.  In this document we will
attempt to enumerate the steps required to connect to data science tools to
expose Immuta data stores.

Immuta enables remote connection to the Immuta datastore through a PostgreSQL queries.
In principle any tool which can connect to a PostgreSQL database can access the Immuta data.

## Getting Started
In order to connect to the Immuta datastore, we must first identify the connection string.

1. Sign into the Immuta Console, shown below
![Immuta Console](ImmutaFrontScreen.png)

2. Select your initials on the upper left hand corner of the GUI.  In the case here **JR**

3. Select **SQL Credentials**
![SQL Credentials](SQLCredentials.png)

4. If you have not set a password, select **Change Password**.  

5. Copy the connection string and remember the password you set.

Now we have the basic connection information needed to connect to a tool.

## Python 3.x Using SQLAlchemy and Pandas Dataframes

**SQLAlchemy** is the Python SQL toolkit to connect and query SQL databases.  This package
will form the primary interface to query the source data.  **Psycopg2** is a popular
PostgreSQL adapter for Python.  This is will provide the drivers needed to connect to the
Immuta data store.  Finally **Pandas** is a high performance data structurs and data
analysis toolkit which will hold the results of any query on the Immuta data store.
Connecting to the Immuta data store will require installing all of these packages:

1. Install the required packages.  If you are using pip execute the following command:

`pip install sqlalchemy psycopg2 pandas`

2. Now launch python 3.x
3. We now need to create an SQLAlchemy Engine.  The engine package
defines the basic components used to interface DB-API modules with
higher-level statement construction, connection-management, execution,
 and result contexts.  The following code can be used to
 create the SQLAlchemy Engine:

 ```
import sqlalchemy as sq
import psycopg2 as pg
import getpass
user_name = 'joe' # Enter your username here
host_name = 'db.innovation.immuta.limited'
port_number = 5432
db_name = 'immuta'

def MakeConnectionString(user_name, host_name,
                         port_number, db_name):
    password = getpass.getpass()
    connection_string = "postgresql://%s:%s@%s:%i/%s?sslmode=require"%(user_name,
                                                                      password,
                                                                      host_name,
                                                                      port_number,
                                                                      db_name)
    return(connection_string)

engine = sq.create_engine(MakeConnectionString(user_name, host_name, port_number,
                                              db_name))
```
4. You can now examine the tables which have been exposed to you using the
SQLAlchemy inspector object:

```
inspector = sq.inspect(engine)
## Look at the available source tables
inspector.get_foreign_table_names()
```

The list of tables returned from the ``get_foreign_table_names``
will match the list of tables which are exposed to you in Immuta.

5. Read a table into a Pandas data frame.  This can be done
several ways but here are a few methods:

```
# create a connection
conn = engine.connect()

# Query the table through Pandas
df_pandas = pd.read_sql("SELECT * FROM credit_card_merged_data", conn)

# Use SQLAlchemy to query the table directory and fold
# the results into pandas
query = conn.execute('SELECT * FROM credit_card_merged_data')
df_sql = pd.DataFrame(query.fetchall(), columns=query.keys())

# Close the connection
conn.close()
```
