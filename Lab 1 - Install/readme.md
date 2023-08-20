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
example
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
binlog-format=ROW
log-bin=/log/sql-innodbcluster1.log
enforce_gtid_consistency=ON
gtid_mode=ON
server-id=251 // server-id berbeda dengan server lain dan jg server_uuid pada datadir/auto.cnf
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