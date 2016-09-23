The AWS region you're using for my nodes is: US-east-1b
The AMI I'm using for my nodes is: Red Hat Enterprise Linux 7.2 (HVM), SSD Volume Type - ami-2051294a
The public IP of the node I will use to host my MySQL server is: 54.211.188.15

The command and output for ls /usr/java on this node:
[root@ip-172-31-23-163 ec2-user]# ls /usr/java
ls: cannot access /usr/java: No such file or directory

Add the following Linux accounts to all nodes
- User christie with a UID of 2500
- User weiner with a UID of 2501
- Create the group pictures and add weiner to it
- Create the group bridges and add christie to it

My commands to perform the Linux account setup on one node is below:

1. As root, create the /etc/pictures / and /etc/bridges directories by typing the following at 
a shell prompt:
[root@ip-172-31-23-162 ec2-user]# mkdir /etc/pictures
[root@ip-172-31-23-162 ec2-user]# mkdir /etc/bridges

2. Add the pictures and bridges groups to the system:
[root@ip-172-31-23-162 ec2-user]# groupadd pictures
[root@ip-172-31-23-162 ec2-user]# groupadd bridges

3. Associate the contents of the /etc/pictures/ directory with the pictures group:
[root@ip-172-31-23-162 ec2-user]# chown root:pictures /etc/pictures
Associate the contents of the /etc/bridges/ directory with the bridges group:
[root@ip-172-31-23-162 ec2-user]# chown root:bridges /etc/bridges

4. Allow users in the groups to create files within the directories and set the setgid bit:
[root@ip-172-31-23-162 ec2-user]# chmod 2775 /etc/pictures
[root@ip-172-31-23-162 ec2-user]# chmod 2775 /etc/bridges

At this point, all members of the pictures group can create and edit files in the 
/etc/pictures/ directory and all members of the bridges group can create and edit files in the 
/etc/bridges/ directory. To verify that the permissions have been set correctly, run the 
following commands:

[root@ip-172-31-23-162 ec2-user]# ls -ld /etc/pictures
drwxrwsr-x 2 root pictures 6 Sep 23 09:26 /etc/pictures

[root@ip-172-31-23-162 ec2-user]# ls -ld /etc/bridges
drwxrwsr-x 2 root bridges 6 Sep 23 09:26 /etc/bridges


5. Create users:
[root@ip-172-31-23-162 ec2-user]# useradd -u 2500 christie
[root@ip-172-31-23-162 ec2-user]# useradd -u 2501 weiner

6. Add users to the pictures group:
[root@ip-172-31-23-162 ec2-user]# usermod -aG pictures weiner
Add users to the bridges group:
[root@ip-172-31-23-162 ec2-user]# usermod -aG bridges christie

7. Set password for user weiner:
[root@ip-172-31-23-162 ec2-user]# passwd weiner
--password set to weiner--
Set password for user christie:
[root@ip-172-31-23-162 ec2-user]# passwd christie
--password set to christie--

List the /etc/passwd entries for christie and weiner in your setup file:

[root@ip-172-31-23-162 ec2-user]# more /etc/passwd | grep christie
christie:x:2500:2500::/home/christie:/bin/bash

[root@ip-172-31-23-162 ec2-user]# more /etc/passwd | grep weiner
weiner:x:2501:2501::/home/weiner:/bin/bash


List the /etc/group entries for pictures and bridges in your setup file

[root@ip-172-31-23-162 ec2-user]# more /etc/group | grep pictures
pictures:x:1001:weiner

[root@ip-172-31-23-162 ec2-user]# more /etc/group | grep bridges
bridges:x:1002:christie







