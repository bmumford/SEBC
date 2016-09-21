```

===Check for swappiness value on all nodes====
Get all the available kernel runtime parameters with their current value:
$ sudo sysctl -a | grep vm.swappiness

Set an instantaneous kernel runtime parameter (vm.swappiness) with the -w option:
$ sysctl -w vm.swappiness=1

You have to write it into the /etc/sysctl.conf file to get it re-applied at each boot (add vm.swappiness = 1 to a separate line):
$ sudo vi /etc/sysctl.conf

Then, you need to apply the change:
$ sudo sysctl -p

====Check that noatime is set on any non-root volumes you have===

$ mount command reports:
/dev/xvda2 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
so noatime is not set.
I am using xfs.
lsblk
lsblk -f
blkid

====Check that the reserve space of any non-root volumes to 0====

sudo tune2fs -m 0 /dev/sdb1 (or whatever the block device is) if it existed which id does not.

verify it worked (if it was there):
sudo tune2fs -l /dev/sdb1 | grep ‘Reserved block count’

====Check the user limits for maximum file descriptors and processes====

Yyou can limit httpd (or any other users) user to specific limits by editing /etc/security/limits.conf file, enter:
# vi /etc/security/limits.conf

Set httpd user soft and hard limits as follows:
httpd soft nofile 4096
httpd hard nofile 10240


====Verify/enable the nscd/ntpd service====
yum install nscd
yum install ntp

To tell if nscd/ntpd services are runing:
$ sudo systemctl status nscd
$ sudo systemctl status ntpd

Enabling NSCD:
$ sudo systemctl enable nscd
$ sudo systemctl enable ntpd

Enabling NTPD:
$ sudo systemctl enable ntpd

In addition to having nscd/ntpd started it is mandatory to be sure this service will be started after a reboot. Run:

$ sudo chkconfig nscd  on
$ sudo chkconfig ntpd  on

====Get MySQL 5.5====

Download mysql57-community-release-el7-9.noarch.rpm from http://dev.mysql.com/downloads/repo/yum/ to laptop.
FTP file to all nodes /etc/yum.repos.d/ using winSCP in my case.

Install MySQL
vi mysql-community.repo
Change MySQL 5.7 Community Server
enabled=0

Change MySQL 5.5 Community Server
enabled=1

yum install mysql
yum install mysql-community-server

Download and copy the JDBC connector to all nodes:
Download JDBC driver from: https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Web_Platform/5/html/Administration_And_Configuration_Guide/ch11s02.html
File to copy is to all nodes: mysql-connector-java-5.1.39-bin.jar
Location on nodes: /usr/share/java



====Start the mysqld service.=====
systemctl start mysqld


=====Disable SELinux
cd /etc/selinux
vi config
(change enforcing to disabled)
setenforce 0  <--- to make perminent


==== Use /usr/bin/mysql_secure_installation to:====

a. Set password protection for the server
b. Revoke permissions for anonymous users
c. Permit remote privileged login
d. Remove test databases
e. Refresh privileges in memory
f. Refreshes the mysqld service

run /usr/bin/mysql_secure_installation script and answer questions.




```
