# BACKUP-RESTORE

verify selinux disabled
```
$ sestatus
$ vi /etc/selinux/config
SELINUX=disabled
```

##  Backup Database
```
mysqlbackup --version
mysqlbackup -u cluster -p --socket=/var/lib/mysql/mysql.sock --datadir=/var/lib/mysql --backup-dir=/BACKUP/20230806/  --with-timestamp --compress --backup-image=fullbackup.img --skip-relaylog backup-to-image >> /BACKUP/20230806/backup.log

// Jika ada TDE nya perlu ditambahkan
mysqlbackup -u cluster -p --socket=/var/lib/mysql/mysql.sock --datadir=/var/lib/mysql --backup-dir=/BACKUP/20230814/  --with-timestamp --compress --backup-image=fullbackup.img --encrypt-password="password" --skip-relaylog backup-to-image >> /BACKUP/20230814/backup.log
```

## Restore Database
```
mysqlbackup -u cluster -p --socket=/var/lib/mysql/mysql.sock --datadir=/var/lib/mysql/ --backup-dir=/BACKUP/20230814/ --with-timestamp --uncompress --encrypt-password="password" --backup-image=/BACKUP/20230814/2023-08-14_01-05-08/fullbackup.img copy-back-and-apply-log >> /BACKUP/20230814/restore.log 2>&1 &

// Jika ada TDE nya perlu ditambahkan
mysqlbackup -u cluster -p --socket=/var/lib/mysql/mysql.sock --datadir=/var/lib/mysql/ --backup-dir=/BACKUP/20230814/ --with-timestamp --uncompress --backup-image=/BACKUP/20230814/2023-08-14_01-05-08/fullbackup.img copy-back-and-apply-log >> /BACKUP/20230814/restore.log 2>&1 &

// change owner data directory mysql pada /var/lib/mysql/ 
chown -R mysql:mysql /var/lib/mysql
remove auto.cnf di directory /var/lib/mysql/ (jika ada)
start service mysqld (maka akan generate file auto.cnf baru dengan server_uuid baru)

show master status\G
reset master;
reset slave;

cat cat /BACKUP/20230806/2023-08-07_06-34-55/meta/backup_gtid_executed.sql >>>> untuk melihat GTID terakhir pada saat backup
SET @@GLOBAL.GTID_PURGED=<'diambil dari gtid_executed_backup.sql'>;
SET @@GLOBAL.GTID_PURGED='808bb068-33a5-11ee-8c4b-000c293cc1a2:1-117,808bb800-33a5-11ee-8c4b-000c293cc1a2:1-11,c56e8f33-2cd2-11ee-95c8-000c293cc1a2:1-5';

mysqlsh cluster@localhost:3306
var cluster = dba.getCluster()
cluster.status()
ERROR "WARNING: server_uuid for instance has changed from its last known value. Use cluster.rescan() ... "

cluster.rescan()
cluster.addInstance("innodbcluster3:3306")
```