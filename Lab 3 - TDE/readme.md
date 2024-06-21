# MySQL Transparent Data Encryption

## Setup

Create manifest file
```
$ nano /usr/sbin/mysqld.my

Add this 
{
  "components": "file://component_keyring_encrypted_file"
}
```

Cek plugin component keyring
```
ls /usr/lib64/mysql/plugin/component_keyring_encrypted_file*
```

Create config file
```
sudo nano /usr/lib64/mysql/plugin/component_keyring_encrypted_file.cnf

Add this , direct path pada datadir
{
  "path": "/var/lib/mysql/component_keyring_encrypted_file",
  "password": "password",
  "read_only": false
}
```

Restart & Cek Keyring component is loaded succesfully
```
mysql -uroot -h127.0.0.1 -proot -e "restart"
systemctl restart mysqld
mysql -uroot -h127.0.0.1 -proot -e "SELECT * FROM performance_schema.keyring_component_status;"
```

Install General-Purpose Keyring Functions
```
mysql -uroot -p

INSTALL PLUGIN keyring_udf SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_generate RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_length_fetch RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_type_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_store RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_remove RETURNS INTEGER
  SONAME 'keyring_udf.so';
```

Create sample key:
```
SELECT keyring_key_generate('muhba25', 'AES', 32);
+--------------------------------------------+
| keyring_key_generate('muhba25', 'AES', 32) |
+--------------------------------------------+
|                                          1 |
+--------------------------------------------+
1 row in set (0.00 sec)
```

Check keyring data file content
```
\! cat /var/lib/mysql/component_keyring_encrypted_file
exit;
```

##  Testing to DB

Create Database
```
mysql -u root -p
mysql> create database coba;
mysql> use coba
mysql> CREATE TABLE pegawai (
    ID int NOT NULL,
    Name varchar(255) NOT NULL,
    Email varchar(255),
    Age int,
    PRIMARY KEY (ID)
);

mysql> INSERT INTO pegawai (ID,Name,Email,Age) VALUES 
(0506070801,'Andy','basoadr1@gmail.com',24),
(0506070802,'Baso','basoadr2@gmail.com',25),
(0506070803,'Bas','basoadr6@gmail.com',26),
(0506070804,'Besa','basoadr7@gmail.com',27),
(0506070805,'Basok','basoadr8@gmail.com',28),
(0506070806,'Libas','basoadr9@gmail.com',29),
(0506070807,'Baskoro','basoadr10@gmail.com',30);

mysql> select * from pegawai;
+-----------+---------+---------------------+------+
| ID        | Name    | Email               | Age  |
+-----------+---------+---------------------+------+
| 506070801 | Andy    | basoadr1@gmail.com  |   24 |
| 506070802 | Baso    | basoadr2@gmail.com  |   25 |
| 506070803 | Bas     | basoadr6@gmail.com  |   26 |
| 506070804 | Besa    | basoadr7@gmail.com  |   27 |
| 506070805 | Basok   | basoadr8@gmail.com  |   28 |
| 506070806 | Libas   | basoadr9@gmail.com  |   29 |
| 506070807 | Baskoro | basoadr10@gmail.com |   30 |
+-----------+---------+---------------------+------+
7 rows in set (0.00 sec)
```

See content of table coba.pegawai without login to mysql (some values are in clear text):
```
$ strings -a /var/lib/mysql/coba/pegawai.ibd
```

Encrypt the TABLE
```
mysql -uroot -p
mysql> use coba

alter table coba.pegawai encryption='Y';
Query OK, 7 rows affected (0.03 sec)
Records: 7  Duplicates: 0  Warnings: 0

select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
mysql> select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
+--------------+------------+
| name         | encryption |
+--------------+------------+
| coba/pegawai | Y          |
+--------------+------------+
1 row in set (0.00 sec)

mysql> exit
```

Check content of table coba.pegawai without login to mysql (all values are encrypted):
```
$ strings -a /var/lib/mysql/coba/pegawai.ibd
```

Unencrypt table coba.pegawai
```
mysql> alter table coba.pegawai encryption='N';
Query OK, 7 rows affected (0.03 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql> select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
Empty set (0.00 sec)

mysql> exit;
```

Check content of table coba.pegawai without login to mysql (all values are unencrypted):
```
$ strings -a /var/lib/mysql/coba/pegawai.ibd
```

## Rotate master key

Encrypt table coba.pegawai and rotate master key (master key rotation is atomic, it does not need to decrypt and encrypt table data):
```
mysql -uroot -p

mysql> alter table coba.pegawai encryption='Y';
mysql> select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
mysql> alter instance rotate innodb master key;
Query OK, 0 rows affected (0.01 sec)
mysql> exit;

$ strings -a /var/lib/mysql/coba/pegawai.ibd
```