Security Lab
SEBC/security/kerberos.md

Integrating Kerberos with Cloudera Manager

Plan one: follow the documentation here
Plan two: Launch the Kerberos wizard and complete the checklist.

Set up an MIT KDC (put on Cloudera Manager server usually)
=== Instruction directly below ====

Create a Linux account (all nodes) with your GitHub name
# useradd -m bmumford

Once integration is sucessful, add these files to security/labs:
/etc/krb5.conf
/var/kerberos/krb5kdc/kdc.conf
/var/kerberos/krb5kdc/kadm5.acl
Create a file kinit.md that includes:
The kinit command you use to authenticate your user

[root@ip-172-31-25-164 etc]# kinit cloudera-scm
Password for cloudera-scm@BMUMFORD.COM:

The output from klist showing your credentials

[root@ip-172-31-25-164 etc]# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: cloudera-scm@BMUMFORD.COM

Valid starting       Expires              Service principal
09/22/2016 12:12:04  09/23/2016 12:12:04  krbtgt/BMUMFORD.COM@BMUMFORD.COM

Create a file cm_creds.png that shows the service principals listed in CM


===========Set up an MIT KDC==========
Cloudera Manager version 5 or later

Pre-requisite:
You have a Hadoop Cluster managed by Cloudera Manager. 


Note: Stop All services
Step 1:
To install packages for a Kerberos server first:

# yum -y install krb5-server krb5-libs krb5-auth-dialog krb5-workstation
(The above installs the client as well)

To install packages for a Kerberos client on all other nodes as well:

# yum -y install krb5-workstation krb5-libs krb5-auth-dialog

Step 2:
Server:
–> Change Realm Name > BMUMFORD.COM
–> Add parameters > max_life = 1d and max_renewable_life = 7d

# vim /var/kerberos/krb5kdc/kdc.conf
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
  BMUMFORD.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
  max_life = 1d
  max_renewable_life = 7d
 }
Step 3:
Add below properties in All Clients:
> udp_preference_limit = 1
> default_tgs_enctypes = arcfour-hmac
> default_tkt_enctypes = arcfour-hmac

# vim /etc/krb5.conf 
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 default_realm = BMUMFORD.COM
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 udp_preference_limit = 1
 default_tgs_enctypes = arcfour-hmac
 default_tkt_enctypes = arcfour-hmac

[realms]
 BMUMFORD.COM = {
 kdc = ip-172-31-25-164.ec2.internal
 admin_server = ip-172-31-25-164.ec2.internal
}

[domain_realm]
 .ec2.internal = BMUMFORD.COM
 ec2.internal = BMUMFORD.COM

Step 4:
Create the database using the kdb5_util utility. (Server)

# /usr/sbin/kdb5_util create -s
Loading random data
Initializing database '/var/kerberos/krb5kdc/principal' for realm 'BMUMFORD.COM',
master key name 'K/M@BMUMFORD.COM'
You will be prompted for the database Master Password.
It is important that you NOT FORGET this password.
Enter KDC database master key:
Re-enter KDC database master key to verify:

Step 5:
In Server, add cloudera-scm principal, it will be used by Cloudera Manager later to manage Hadoop principals.

# kadmin.local
Authenticating as principal root/admin@BMUMFORD.COM with password.
kadmin.local:  addprinc cloudera-scm@BMUMFORD.COM
WARNING: no policy specified for cloudera-scm@BMUMFORD.COM; defaulting to no policy
Enter password for principal "cloudera-scm@BMUMFORD.COM":
Re-enter password for principal "cloudera-scm@BMUMFORD.COM":
Principal "cloudera-scm@BMUMFORD.COM" created.
kadmin.local:  exit

Step 6:
Add */admin and cloudera-scm to ACL(Access Control List), which gives privilege to add principals for admin and cloudera-scm principal

# vim /var/kerberos/krb5kdc/kadm5.acl 
*/admin@BMUMFORD.COM    *
cloudera-scm@BMUMFORD.COM admilc

Step 7:
Adds the password policy to the database.

# kadmin.local
kadmin.local:  addpol admin
kadmin.local:  addpol users
kadmin.local:  addpol hosts
kadmin.local:  exit

Step 8:
Start Kerberos using the following commands:

#service krb5kdc start
#service kadmin start

Before completing Step 9 below test kdc installation: see top section


Step 9:
Now go to Cloudera Manager UI:
Administration > Kerberos > Enable Kerberos
Check all the boxes:

 
