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
Install plugin 
```
mysql -uroot -D mysql -h::1 -P3307 < /usr/share/mysql-8.1/audit_log_filter_linux_install.sql

mysql -uroot -h::1 -P3307 -e "show plugins"
```
Create audit log filter and assign to all
```
mysql -uroot -h::1 -P3307

SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('%', 'log_all');
SELECT audit_log_filter_flush() AS 'Result';

exit;
```
## MySQL Enterprise Firewall
### 1. Install Firewall
```
mysql -uroot -h127.0.0.1 -P3307 < /usr/share/mysql-8.1/linux_install_firewall.sql 

mysql -uroot -h127.0.0.1 -P3307

show variables like '%firewall%';

show plugins;

select * from mysql.firewall_users;

		lists names and operational modes of registered firewall account profiles

select * from mysql.firewall_whitelist;

		lists allowlist rules of registered firewall account profiles.

select * from information_schema.mysql_firewall_whitelist;

		a view into the in-memory data cache for MySQL Enterprise Firewall

select * from mysql.firewall_groups;

		lists names and operational modes of registered firewall group profiles

select * from mysql.firewall_group_allowlist;

		lists allowlist rules of registered firewall group profiles

select * from mysql.firewall_membership;

    lists the members (accounts) of registered firewall group profile

show global status like '%firewall%';

		Firewall_access_denied: The number of statements rejected 
		Firewall_access_granted: The number of statements accepted
		Firewall_access_suspicious: The number of statements logged
		Firewall_cached_entries: The number of statements recorded, including duplicates. 
```
## 2. Set important privileges
```
grant FIREWALL_EXEMPT on *.* to root@'localhost';
grant FIREWALL_ADMIN on *.* to root@'localhost';
```
## 3. Train Firewall
As root:
```
create user demo@'%' identified by 'demo';
grant all privileges on world_x.* to demo@'%';
call mysql.sp_set_firewall_group_mode('group1','RECORDING');
select * from mysql.firewall_groups;
call mysql.sp_firewall_group_enlist('group1','demo@%');
select * from mysql.firewall_membership;
exit;
```
As demo:
```
mysql -udemo -pdemo -h127.0.0.1 -P3307

show databases;
use world_x;
show tables;
select * from city;
exit;
```
As root:
```
mysql -uroot -h127.0.0.1 -P3307

SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'group1';
SELECT * FROM performance_schema.firewall_membership WHERE GROUP_ID = 'group1' ORDER BY MEMBER_ID;
SELECT RULE FROM performance_schema.firewall_group_allowlist WHERE NAME = 'group1';
```
## 4. Testing the Firewall
AS ROOT, turn on protecting mode
```
call mysql.sp_set_firewall_group_mode('group1','PROTECTING');
grant firewall_user on *.* to demo@'%';
exit
```
As demo:
```
mysql -udemo -pdemo -h127.0.0.1 -P3307

show databases;
use world_x;
show tables;
select * from city;
select * from city where id=1;
```



