# MySQL Enterprise Audit
## 1. Install Audit Plugin and shutdown database
```
mysql -uroot -h127.0.0.1 -proot < /usr/share/mysql-8.0/audit_log_filter_linux_install.sql
```
Add variables into option file
```
echo "audit_log_format=json" >> /etc/my.cnf
echo "audit_log=FORCE_PLUS_PERMANENT" >> /etc/my.cnf
```
Restart database instance
```
mysql -uroot -h127.0.0.1 -proot 

restart;
```
## 2. Check audit log plugin 
```
show plugins;
show variables like '%audit_log%';
select * from mysql.audit_log_user;
select * from mysql.audit_log_filter;
```
## 3. Install audit log filter to audit all activities and apply to all users
```
SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('%', 'log_all');
select * from mysql.audit_log_user;
select * from mysql.audit_log_filter;
exit;
```
Check content of audit log file
```
cat /var/lib/mysql/audit.log
```
## 6. Login to database and create test database
Login to database:
```
mysql -uroot -h127.0.0.1 -proot
```
Run the following SQL statements:
```
create database test;
```
Check audit log from mysql session (without logout from mysql):
```
select audit_log_read_bookmark();
select audit_log_read(audit_log_read_bookmark());
exit;
```
Check audit log:
```
cat /var/lib/mysql/audit.log
```
## 7. Rotate the audit log
Automatic audit log rotation
```
mysql -uroot -proot -h127.0.0.1 -e "set persist audit_log_rotate_on_size=1048576;"
```
Manual audit log rotation
```
mv /var/lib/mysql/audit.log /var/lib/mysql/audit.log.bak
mysql -uroot -h127.0.0.1 -proot -e "set global audit_log_rotate_on_size=0"
mysql -uroot -h127.0.0.1 -proot -e "set global audit_log_flush=ON;"
ls /var/lib/mysql/audit.*
```

