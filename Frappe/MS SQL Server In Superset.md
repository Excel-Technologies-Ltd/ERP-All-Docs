# Connect MS SQL Server In Superset 

* First Install pyodbc

```
pip install pyodbc
```

* Check if the ODBC driver is installed

```bash
find /opt -name "libmsodbcsql*.so*"
```

* ODBC Driver 17 for SQL Server

```
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list | tee /etc/apt/sources.list.d/msprod.list
apt-get update
ACCEPT_EULA=Y apt-get install -y mssql-tools unixodbc-dev
```
​
* Verify the ODBC driver is installed

```
odbcinst -q -d
​```

* Connection string should like:

```
mssql+pyodbc://SA:server123%23%40%21@202.91.42.205/Reward?driver=ODBC+Driver+17+for+SQL+Server&timeout=30
```
