**Mariadb Database not connect issue**

```sql
SELECT user, host FROM mysql.user WHERE user = 'root';
SHOW GRANTS FOR 'root'@'10.0.17.4';
10.0.17.4
```

```sql
SELECT user, host FROM mysql.user WHERE user = '_1d2d4dbdf71edd7b' AND host = '10.0.27.8';
```

```sql
ALTER USER '_1d2d4dbdf71edd7b'@'10.0.27.8' IDENTIFIED BY 'qNWJ6lIaSiob19M59GhQ';
```

```sql
GRANT ALL PRIVILEGES ON _1d2d4dbdf71edd7b.* TO '_1d2d4dbdf71edd7b'@'10.0.27.8' IDENTIFIED BY 'qNWJ6lIaSiob19M59GhQ';
FLUSH PRIVILEGES;
```

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.0.17.4' IDENTIFIED BY 'LLiuG55P1VQ1ScS' WITH GRANT OPTION;
```
