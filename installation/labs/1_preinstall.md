<<<<<<< HEAD
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

Install mysql client:
yum install mysql

Install mysql server:
yum install mysql-community-server

Download and copy the JDBC connector to all nodes:
Download JDBC driver from: https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Web_Platform/5/html/Administration_And_Configuration_Guide/ch11s02.html
File to copy is to all nodes: mysql-connector-java-5.1.39-bin.jar
Location on nodes: /usr/share/java

Create link for mysql connector actual-filename.jar to what Cloudera Manager generically looks for (mysql-connector-java.jar)

ln -s mysql-connector-java-5.1.39-bin.jar mysql-connector-java.jar


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

Install the Oracle Java Development Kit (JDK) on the Cloudera Manager Server host. Download from: 
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/RPMS/x86_64/
sudo yum install oracle-j2sdk1.7

Install the Cloudera Manager Server packages either on the host where the database is installed, or on a host that has access to the database. Download from: 
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/RPMS/x86_64/

sudo yum install cloudera-manager-daemons-5.8.1-1.cm581.p0.7.el7.x86_64.rpm cloudera-manager-server-5.8.1-1.cm581.p0.7.el7.x86_64.rpm

Create databases and user accounts for components that require databases:
• If you are not using the Cloudera Manager installer, the Cloudera Manager Server.
• Cloudera Management Service roles:
– Activity Monitor (if using the MapReduce service)
– Reports Manager
• Each Hive metastore
• Sentry Server
• Cloudera Navigator Audit Server
• Cloudera Navigator Metadata Server
You can create these databases on the host where the Cloudera Manager Server will run, or on any other hosts in the
cluster. For performance reasons, you should install each database on the host on which the service runs, as determined by the roles you assign during installation or upgrade. In larger deployments or in cases where database administrators are managing the databases the services use, you can separate databases from services, but use caution.
The database must be configured to support UTF-8 character set encoding.
Record the values you enter for database names, usernames, and passwords. The Cloudera Manager installation wizard requires this information to correctly connect to these databases.
1. Log into MySQL as the root user:
$ mysql -u root -p
Enter password:
2. Create databases for the Activity Monitor, Reports Manager, Hive Metastore Server, Sentry Server, Cloudera Navigator Audit Server, and Cloudera Navigator Metadata Server:

mysql> create database database DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)
mysql> grant all on database.* TO 'user'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.00 sec)

database, user, and password can be any value. The examples match the default names provided in the Cloudera Manager configuration settings:

Role 			Database	User 	Password
Activity 	Monitor 	amon 		amon 	amon_password
Reports Manager 	rman 		rman 	rman_password
Hive Metastore Server 	metastore 	hive	 hive_password
Sentry Server 		sentry 		sentry 	sentry_password
Cloudera Navigator Audit nav 		nav 	nav_password
Server
Cloudera Navigator	navms 		navms	 navms_password
Oozie			oozie		oozie	
Hue			hue		hue
SCM Server		scm		scm



Preparing a Cloudera Manager Server External Database
Run the scm_prepare_database.sh script on the host where the Cloudera Manager Server package is installed:
• Installer or package install
/usr/share/cmf/schema/scm_prepare_database.sh

scm_prepare_database.sh Syntax
scm_prepare_database.sh database-type [options] database-name username password

Table 6: Required Parameters
Parameter Description
database-type One of the supported database types:
		• MariaDB - mysql
		• MySQL - mysql
		• Oracle - oracle
		• PostgreSQL - postgresql
database-name 	The name of the Cloudera Manager Server database to create or use.
username	The username for the Cloudera Manager Server database to create or use.
password	The password for the Cloudera Manager Server database to create or use. If you do 			not specify the password on the command line, the script prompts you to enter it.

Table 7: Options (if not already performed)
Option 		Description
-h or --host	The IP address or hostname of the host where the database is installed. The default is
		to use the local host.
-P or --port	The port number to use to connect to the database. The default port is 3306 for 			MariaDB, 3306 for MySQL, 5432 for PostgreSQL, and 1521 for Oracle. This option is 			used for a remote connection only.
-u or --user	The admin username for the database application. For -u, no space occurs between 			the option and the provided value. If this option is supplied, the script creates a user 			and database for the Cloudera Manager Server; otherwise, it uses the user and 			database you created previously.
-p or --password The admin password for the database application. The default is no password. For -			p, no space occurs between the option and the provided value.
--scm-host	The hostname where the Cloudera Manager Server is installed. Omit if the Cloudera
		Manager Server and the database are installed on the same host.
--config-path	The path to the Cloudera Manager Server configuration files. The default is
		/etc/cloudera-scm-server.
--schema-path	The path to the Cloudera Manager schema files. The default is
		/usr/share/cmf/schema (the location of the script).
-f 		The script does not stop if an error occurs.
-? or --help 	Display help.

Run prepare scm schema script:
/usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm ''

Save the appropriate Cloudera Manager repo file (cloudera-manager.repo) for your system:
RHEL/CentOS 7: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo
Copy the repo file to the /etc/yum.repos.d/ directory.

Start the Cloudera Manager Server. Run this command on the Cloudera Manager Server host:
sudo service cloudera-scm-server start

Start and Log into the Cloudera Manager Admin Console
The Cloudera Manager Server URL takes the following form http://Server host:port, where Server host is the
fully qualified domain name or IP address of the host where the Cloudera Manager Server is installed, and port is the
port configured for the Cloudera Manager Server. The default port is 7180.
1. Wait several minutes for the Cloudera Manager Server to start. To observe the startup process, run tail -f
/var/log/cloudera-scm-server/cloudera-scm-server.log on the Cloudera Manager Server host. If
the Cloudera Manager Server does not start, see Troubleshooting Installation and Upgrade Problems on page 513.
2. In a web browser, enter http://Server host:7180, where Server host is the fully qualified domain name or
IP address of the host where the Cloudera Manager Server is running.
The login screen for Cloudera Manager Admin Console displays.
3. Log into Cloudera Manager Admin Console. The default credentials are: Username: admin Password: admin.
Cloudera Manager does not support changing the admin username for the installed account. You can change the
password using Cloudera Manager after you run the installation wizard. Although you cannot change the admin
username, you can add a new user, assign administrative privileges to the new user, and then delete the default
admin account.
4. After logging in, the Cloudera Manager End User License Terms and Conditions page displays. Read the terms
and conditions and then select Yes to accept them.
5. Click Continue.
The Welcome to Cloudera Manager page displays.
After logging in, the Cloudera Manager End User License Terms and Conditions page displays. Read the terms
and conditions and then select Yes to accept them.
5. Click Continue.
The Welcome to Cloudera Manager page displays.
Choose Cloudera Manager Edition
From the Welcome to Cloudera Manager page, you can select the edition of Cloudera Manager to install and, optionally,
install a license:
1. Choose which edition to install:
• Choose Cloudera Enterprise Enterprise Data Hub Edition Trial, which does not require a license, but expires after 60 days and cannot be renewed.
Information is displayed indicating what the CDH installation includes. At this point, you can click the Support
drop-down menu to access online Help or the Support Portal.
4. Click Continue to proceed with the installation.

Choose which hosts will run CDH and managed services
If you are using Cloudera Manager to install software, search for and choose hosts:
1. To enable Cloudera Manager to automatically discover hosts on which to install CDH and managed
services, enter the cluster hostnames or IP addresses. You can also specify hostname and IP address
ranges. 
For example:

Range Definition 		Matching Hosts
10.1.1.[1-4] 			10.1.1.1, 10.1.1.2, 10.1.1.3, 10.1.1.4
host[1-3].company.com 		host1.company.com, host2.company.com, host3.company.com
host[07-10].company.com	host07.company.com, host08.company.com, host09.company.com,
				host10.company.com

Enter the EC2 "private" hostnames or IP's.
You can specify multiple addresses and address ranges by separating them with commas, semicolons,
tabs, or blank spaces, or by placing them on separate lines. Use this technique to make more specific
searches instead of searching overly wide ranges. The scan results will include all addresses scanned,
but only scans that reach hosts running SSH will be selected for inclusion in your cluster by default. If
you do not know the IP addresses of all of the hosts, you can enter an address range that spans over
unused addresses and then clear the hosts that do not exist (and are not discovered) later in this
procedure. However, keep in mind that wider ranges will require more time to scan.
2. Click Search. Cloudera Manager identifies the hosts on your cluster to allow you to configure them for services. If there are a large number of hosts on your cluster, wait a few moments to allow them to be discovered and shown in the wizard. If the search is taking too long, you can stop the scan by clicking Abort Scan. To find additional hosts, click New Search, add the host names or IP addresses and click Search again. Cloudera Manager scans hosts by checking for network connectivity. If there are some hosts where you want to install services that are not shown in the list, make sure you have network connectivity between the Cloudera Manager Server host and those hosts. Common causes of loss of connectivity are firewalls and interference from SELinux.
3. Verify that the number of hosts shown matches the number of hosts where you want to install services. Clear host entries that do not exist and clear the hosts where you do not want to install services.

Click Continue.
The Cluster Installation Select Repository screen displays.

Choose the software installation type and CDH and managed service version:
• Use Parcels (for lab none needed)
1. Choose the parcels to install. The choices depend on the repositories you have chosen; a repository can contain multiple parcels. Only the parcels for the latest supported service versions are configured by default.
You can add additional parcels for previous versions by specifying custom repositories. For example, you can find the locations of the previous CDH 4 parcels at
https://archive.cloudera.com/cdh4/parcels/. Or, if you are installing CDH 4.3 and want to
use policy-file authorization, you can add the Sentry parcel using this mechanism.
1. To specify the parcel directory, specify the local parcel repository, add a parcel repository, or specify
the properties of a proxy server through which parcels are downloaded, click the More Options
button and do one or more of the following:
• Parcel Directory and Local Parcel Repository Path - Specify the location of parcels on cluster
hosts and the Cloudera Manager Server host. If you change the default value for Parcel Directory
and have already installed and started Cloudera Manager Agents, restart the Agents:
sudo service cloudera-scm-agent restart
• Parcel Repository - In the Remote Parcel Repository URLs field, click the button and enter
the URL of the repository. The URL you specify is added to the list of repositories listed in the
Configuring Cloudera Manager Server Parcel Settings on page 55 page and a parcel is added
to the list of parcels on the Select Repository page. If you have multiple repositories configured,
you see all the unique parcels contained in all your repositories.
• Proxy Server - Specify the properties of a proxy server.
2. Click OK.
2. If you are using Cloudera Manager to install software, select the release of Cloudera Manager Agent.
You can choose either the version that matches the Cloudera Manager Server you are currently using
or specify a version in a custom repository. If you opted to use custom repositories for installation files,
you can provide a GPG key URL that applies for all repositories.

Select Install Oracle Java SE Development Kit (JDK) to allow Cloudera Manager to install the JDK on each cluster host. If you have already installed the JDK, do not select this option. If your local laws permit you to deploy unlimited strength encryption, and you are running a secure cluster, select the Install Java Unlimited Strength Encryption
Policy Files checkbox.
Note: If you already manually installed the JDK on each cluster host, this option to install the
JDK does not display.

DO NOT SELECT SINGLE USER MODE check box.

If you chose to have Cloudera Manager install software, specify host installation properties:
• Select root or enter the username for an account that has password-less sudo permission. (for EC2 environment use ec2-user.
• Select an authentication method: (use blank)
– If you choose password authentication, enter and confirm the password.
– If you choose public-key authentication, provide a passphrase and path to the required key files.
• You can specify an alternate SSH port. The default value is 22.
• You can specify the maximum number of host installations to run at once. The default value is 10.
6. Click Continue. If you chose to have Cloudera Manager install software, Cloudera Manager installs the Oracle JDK, Cloudera Manager Agent, packages and CDH and managed service parcels or packages. During parcel installation, progress is indicated for the phases of the parcel installation process in separate progress bars. If you are installing multiple parcels, you see progress bars for each parcel. When the Continue button at the bottom of the screen turns blue, the installation process is completed.
Click Continue.
The Host Inspector runs to validate the installation and provides a summary of what it finds, including all the versions of the installed components. If the validation is successful, click Finish.

Fix Hugepage issue with provided instructions.

On the Database Setup page, configure settings for required databases:
1. Enter the database host, database type, database name, username, and password for the database that you created when you set up the database.
2. Click Test Connection to confirm that Cloudera Manager can communicate with the database using the information you have supplied. If the test succeeds in all cases, click Continue




```
=======
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
sudo tune2fs -l /dev/sdb1 | grep â€˜Reserved block countâ€™

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
>>>>>>> 364cbcb04c41b3c000de109a8906a99abf817c41
