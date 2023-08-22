# MySQL Enterprise Server Installation
## 1. Install MySQL Enterprise Server, MySQL Enterprise Shell and MySQL Enterprise Router
Download file Mysql Enterprise Server, Router and MySQL Enterprise Shell and Install

```
cd /home/mysql/
mkdir -p software/mysqlserverouter
mkdir -p software/mysqlshell

unzip MySQL-Server-8.0.30.zip -d mysqlserverouter
unzip MySQL-Shell-8.0.30.zip -d mysqlshell

cd software/mysqlserverouter
sudo yum -y localinstall mysql-commercial-*
cd software/mysqlshell
sudo yum -y localinstall mysql-shell-commercial-*
```

## Stop and disable service firewalld and selinux

```
sestatus
vi /etc/selinux/config
SELINUX=disabled

systemctl stop firewalld
systemctl disable firewalld
```

## set hostname on /etc/hosts
```
//example
192.168.170.130 innodbcluster1
192.168.170.131 innodbcluster2
192.168.170.132 innodbcluster3
```

## create user
```
create user 'cluster'@'%' identified with mysql_native_password by 'tes250299';
GRANT ALL PRIVILEGES ON *.* TO 'cluster'@'%' WITH GRANT OPTION;
```
## setup cluster
```
mysqlsh cluster@localhost:3306 
OR
mysqlsh
\c cluster@localhost:3306

dba.createCluster("DC_Cluster");
```

## Set config Innodbcluster
```
nano /etc/my.cnf
log-error=/log/sql-innodbcluster1.log

server-id=251 // server-id berbeda dengan server lain dan jg server_uuid pada datadir/auto.cnf
binlog-format=ROW
log-bin=/log/binlog-innodbcluster1.log
binlog_transaction_dependency_tracking=WRITESET
enforce_gtid_consistency=ON
gtid_mode=ON


//Additional config for tuning (optional)
innodb_buffer_pool_size //default 128 MB for tuning umumnya diset 75% dari total memory
innodb_buffer_pool_instances //default 8
sql_mode='' //set empty, default value ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
binlog_expire_logs_seconds=864000 (10 hari) // default 30 hari
table_open_cache=20000  //default 4000
max_connections=10000 //default 151
wait_timeout=120 (2 menit) //default 28800 (8 jam)
sort_buffer_size=2M //default 0
read_buffer_size=1M //default 262144 (256 KB)
read_rnd_buffer_size=8M // default 262144 (256 KB)
join_buffer_size=1M // default 262144 (256 KB)
bulk_insert_buffer_size=16M //default 8388608 (8 MB)
log_timestamps=SYSTEM //default UTC
max_allowed_packet=1G //default 67108864 (64 MB)
slow_query_log=ON //default OFF
long_query_time=5 // default 10
thread_cache_size=512 //default 0
plugin_load_add='thread_pool.so'
plugin_load_add='group_replication.so'

https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html
https://dev.mysql.com/doc/refman/8.0/en/common-errors.html

```
## Add Instance
```
mysqlsh cluster@localhost:3306 
dba.createCluster("DC_Cluster");
var cluster = dba.getCluster()
cluster.status()
cluster.addInstance("innodbcluster2:3306")
cluster.addInstance("innodbcluster3:3306")

mysql -uroot -p -e "SELECT * FROM performance_schema.replication_group_members";
```

## Additional Setup router
```
cd software/mysqlserverouter
sudo yum -y localinstall mysql-router-commercial-*

mkdir -p /router/mysqlrouter
mysqlrouter --bootstrap cluster@innodbcluster1:3306 --directory /router/mysqlrouter --account=mysqlrouter --user=mysqlrouter --conf-base-port=3306 --name=MysqlRouterAplikasi1

vi /router/mysqlrouter/mysqlrouter.conf 
Add to mysqlrouter.conf on bootstrap-ro and bootstrap-rw
max_connections=64000
max_connect_errors=4294967295

./start.sh 
tail -100f /router/mysqlrouter/log/mysqlrouter.log
```