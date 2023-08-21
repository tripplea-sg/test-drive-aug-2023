# MySQL Enterprise Security
## Transparent Data Encryption (TDE)
![Image of picture1](https://github.com/tripplea-sg/test-drive-aug-2023/blob/main/Images/Screenshot%202023-08-22%20at%206.24.23%20AM.png)
Install plugin 
```
vi /home/opc/mysql-sandboxes/3307/my.cnf

## content
early-plugin-load=keyring_encrypted_file.so
keyring_encrypted_file_data=/home/opc/mysql-sandboxes/3307/sandboxdata/keyring-encrypted
keyring_encrypted_file_password=password
```
Restart 
```
mysql -uroot -h::1 -P3307 -e "restart"
```
Check current keyring installation status
```
mysql -uroot -h::1 -P3307 -e "show plugins" | grep keyring_encrypted_file
```
Gather table data from OS
```
strings -a /home/opc/mysql-sandboxes/3307/sandboxdata/world_x/countryinfo.ibd
```
Encrypt table using TDE
```
mysql -uroot -h::1 -P3307 -e "alter table world_x.countryinfo encryption='Y'"
```
Gather table data from OS
```
strings -a /home/opc/mysql-sandboxes/3307/sandboxdata/world_x/countryinfo.ibd
```
Select data from MySQL
```
mysql -uroot -h::1 -P3307 -e "select * from world_x.countryinfo limit 1"
```
## MySQL Enterprise Audit
Install plugin on 3307, 3308, 5506, 2206
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster setPrimaryInstance localhost:3306

mysql -uroot -h::1 -P3307 -e "set global super_read_only=off; set sql_log_bin=0; INSTALL PLUGIN audit_log SONAME 'audit_log.so'; set sql_log_bin=1; set global super_read_only=on;"
mysql -uroot -h::1 -P3308 -e "set global super_read_only=off; set sql_log_bin=0; INSTALL PLUGIN audit_log SONAME 'audit_log.so'; set sql_log_bin=1; set global super_read_only=on;"
mysql -uroot -h::1 -P2206 -e "set global super_read_only=off; set sql_log_bin=0; INSTALL PLUGIN audit_log SONAME 'audit_log.so'; set sql_log_bin=1; set global super_read_only=on;"
mysql -uroot -h::1 -P5506 -e "set global super_read_only=off; set sql_log_bin=0; INSTALL PLUGIN audit_log SONAME 'audit_log.so'; set sql_log_bin=1; set global super_read_only=on;"
```
On 3306, run script to install Audit
```
mysql -uroot -h::1 -e "create database mysql_audit"

mysql -uroot -D mysql_audit -h::1 < /usr/share/mysql-8.1/audit_log_filter_linux_install.sql

mysql -uroot -h::1 -e "show plugins"
```
Create audit log filter and assign to all
```
mysql -uroot -h::1 -e "SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');"
mysql -uroot -h::1 -e "SELECT audit_log_filter_set_user('%', 'log_all');"
```
Flush
```
mysql -uroot -h::1 -P3307 -e "SELECT audit_log_filter_flush() AS 'Result';"
mysql -uroot -h::1 -P3308 -e "SELECT audit_log_filter_flush() AS 'Result';"
mysql -uroot -h::1 -P2206 -e "SELECT audit_log_filter_flush() AS 'Result';"
mysql -uroot -h::1 -P5506 -e "SELECT audit_log_filter_flush() AS 'Result';"
```



