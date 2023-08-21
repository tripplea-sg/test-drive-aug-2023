# Installation, Setup and Configuration
## MySQL Server and MySQL Shell Installation
Extract MySQL Server binary
```
unzip MySQL-server-8.1-01.zip
```
Install MySQL Server
```
sudo yum localinstall -y mysql-commercial-*
```
Extract MySQL Shell
```
unzip MySQL-Shell-8.1.zip
```
Install MySQL Shell
```
sudo yum localinstall -y mysql-shell-commercial-8.1.0-1.1.el8.x86_64.rpm
```
Clean up
```
rm $HOME/*.rpm
```
## Know your server
Find OS version
```
cat /etc/os-release
```
CPU size
```
lscpu
```
RAM size
```
cat /proc/meminfo | grep MemTotal
```
## OS Parameter Tuning
Edit /etc/fstab using "sudo vi /etc/fstab" and change 
```
/dev/mapper/ocivolume-root /                       xfs     defaults        0 0
```
to
```
/dev/mapper/ocivolume-root /                       xfs     defaults,nodiratime,noatime      0 0
```
Reboot the VM.
```
sudo reboot
```
Check vm.swappiness
```
cat /proc/sys/vm/swappiness
```
Change vm.swappiness to "1" by open sysctl.conf
```
sudo vi /etc/sysctl.conf
```
Add the following line, save and exit
```
vm.swappiness=1
```
Reboot the VM.
```
sudo reboot
```
Check IO Scheduler
```
cat /sys/block/sda/queue/scheduler
```
Change IO Scheduler to none
```
sudo su
echo none > /sys/block/sda/queue/scheduler
exit
cat /sys/block/sda/queue/scheduler
```
## Setup and Configure Instance 3306
Create Sandbox Instance 3306
```
sudo systemctl stop mysqld
sudo systemctl disable mysqld
mysqlsh -e "dba.deploySandboxInstance(3306)"

## press enter for empty root password

```
Configure MySQL Instance 3306
```
mysql -uroot -h::1

set persist_only innodb_redo_log_capacity=2147483648;
set persist_only innodb_flush_neighbors=2;
set persist_only innodb_io_capacity=3000;
set persist_only innodb_io_capacity_max=3000;
set persist_only innodb_buffer_pool_size=25769803776;
set persist_only innodb_buffer_pool_instances=12;
set persist_only innodb_lru_scan_depth=250;
set persist_only innodb_page_cleaners=12;
set persist_only innodb_checksum_algorithm=strict_crc32;
set persist_only binlog_row_image=MINIMAL;

set persist_only innodb_thread_sleep_delay=500; 
set persist_only innodb_spin_wait_delay=2;
set persist_only innodb_spin_wait_pause_multiplier=10;

set persist_only sql_generate_invisible_primary_key=on;

restart;

exit;
```
Download and install world_x schema/data
```
wget http://downloads.mysql.com/docs/world_x-db.zip

unzip world_x-db.zip

mysql -uroot -h::1 < world_x-db/world_x.sql

mysql -uroot -h::1 -e "show databases"
```
