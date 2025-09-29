Mariadb Connect Issue When Create Site

```sql
SELECT user, host FROM mysql.user WHERE user='root';

GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.0.46.5' IDENTIFIED BY 'qNWJ6lIaSiob19M59' WITH GRANT OPTION;
FLUSH PRIVILEGES;

UPDATE mysql.user SET password=PASSWORD('qNWJ6lIaSiob19M59') WHERE user='root';
FLUSH PRIVILEGES;

SET PASSWORD FOR 'root'@'localhost' = PASSWORD('qNWJ6lIaSiob19M59');
