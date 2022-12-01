# Documentation for Project 7: DEVOPS TOOLING WEBSITE SOLUTION

- As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

## Preparing prerequisites

- For this project, the following components will be implemented:
- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: GitHub
- Launch 4 Red Hat Servers (1 for NFS and 3 for web server)
- Launch an Ubuntu 20.04 server for Database purposes

1. PREPARE NFS SERVER

- Create 3 LV and attached them to the NFS server.

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg:

`lsblk`

![lsblk Status](./images/lsblk-status.PNG)

- Use gdisk utility to create a single partition on each of the 3 disks:

`gdisk`

- In the command line:

`sudo gdisk /dev/xvdf` repeat for xvdg and xvdh

- Command (? for help): type n after the colon to define as new

- Press enter until this displays "Hex code or GUID (L to show codes, Enter = 8300):"

- To change the default Linux file system to LVM, type 8e00 after the colon in line 42

The output should display as :

- Changed type of partition to 'Linux LVM'

- Command (? for help): p to display the action was done correctly.

- Command (? for help): w to write the action

- Do you want to proceed? (Y/N): Y to confirm

![Xvdf Partition Output](./images/xvdf-part1-output.PNG)

![Xvdg Partition Output](./images/xvdg-part1-outpu.PNG)

![Xvdh Partition Output](./images/xvdh-part1-outpu.PNG)

- Use lsblk utility to view the newly configured partition on each of the 3 disks.

`lsblk`

![lsblk New Status](./images/lsblk-new-output.PNG)

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions:

`sudo yum install lvm2 -y`

![Lvm2 Status](./images/lvm2-install-status.PNG)

- To confirm lvm is install:

`which lvm`

![Lvm Status](./images/lvm-success.PNG)

- To check for available partition:

`sudo lvmdiskscan`

![Lvmdiskscan Status](./images/lvmdiskscan-status.PNG)

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM:

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Pvcreate Physical Volumes](./images/pvcreate-volume-output.PNG)

- Check the pvcreate status:

`sudo pvs`

![Sudo Pvs Check](./images/sudo-pvs-output.PNG)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Vgcreate Output](./images/vgcreate-output.PNG)

- Verify that your VG has been created successfully by running:

`sudo vgs`

![Vgs Check](./images/vgs-output.PNG)

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs:

`sudo lvcreate -n lv-apps -L 9G webdata-vg`

![Vgs lv-apps](./images/lv-apps-output.PNG)

`sudo lvcreate -n lv-logs -L 9G webdata-vg`

![Vgs lv-los](./images/lv-logs-output.PNG)

`sudo lvcreate -n lv-opt -L 9G webdata-vg`

![Vgs lv-opt](./images/lv-opt-output.PNG)

- Verify that your Logical Volume has been created successfully by running:

`sudo lvs`

![Sudo Lvs Output](./images/sudo-lvs-status.PNG)

`lsblk`

![lsblk New Status](./images/check-attach-status.PNG)

- Run the command below to display the packages created:

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![View Complete Output](./images/entire-output.PNG)

- Use mkfs.xfs to format the logical volumes with ext4 filesystem:

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

![Mkfs Lvapps Output](./images/mkfs-lv-apps-output.PNG)

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

![Mkfs Lvlogs Output](./images/mkfs-lv-logs-output.PNG)

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![Mkfs Lvopt Output](./images/mkfs-lv-opt-output.PNG)

- Create mount points on /mnt directory for the logical volumes as follow:

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

![Create Mount Output](./images/create-mount-outputs.PNG)

- To mount the 3 volumes:

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`

![Lv-apps Mount Output](./images/lvapps-success.PNG)

`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`

![Lv-logs Mount Output](./images/lvlogs-success.PNG)

`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![Lv-opt Mount Output](./images/lvopt-success.PNG)

- Install NFS server, configure it to start on reboot and make sure it is u and running:

`sudo yum -y update`

![Yum Update](./images/yum-success.PNG)

`sudo yum install nfs-utils -y`

![Yum Update](./images/nfs-utils-success.PNG)

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

![Nfs Server Enable](./images/nfs-enable-success.PNG)

`sudo systemctl status nfs-server.service`

![Nfs Server Active](./images/nfs-active-success.PNG)

- Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

![Chown Commands](./images/chown-mnt-output.PNG)

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

![Chmod Commands](./images/chmod-mnt-output.PNG)

- Restart the Nfs server:

`sudo systemctl restart nfs-server.service`

- Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

`sudo vi /etc/exports`

![Vi Etc Export](./images/export-output.PNG)

`sudo exportfs -arv`

![Export Success](./images/export-success.PNG)

- Check which port is used by NFS and open it using Security Groups (add new Inbound Rule):

`rpcinfo -p | grep nfs`

![Port Output](./images/port-output.PNG)

- Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049:

![Security Group Output](./images/security-group-output.PNG)

STEP 2 — CONFIGURE THE DATABASE SERVER

- Run the update:

`sudo apt update`

- Install MySQL server:

`sudo apt install mysql-server`

![Lv-opt Mount Output](./images/install-mysqlserver-success.PNG)

- Create a database and name it tooling:

`sudo mysql`

![Mysql launch](./images/mysql-output.PNG)

- Create a database named tooling:

![Create Tooling Database](./images/tooling-db-created.PNG)

`create database tooling;`

![Create Tooling Database](./images/tooling-db-created.PNG)

- Create a user in the database:

`create user 'webaccess'@'172.31.32.0/20' identified by 'password';`

![User Created](./images/user-create-status.PNG)

- Grant permission to webaccess user on tooling database:

`grant all privileges on tooling.* to 'webaccess'@'172.31.32.0/20';`

![Grant Privileges](./images/grant-privilege-output.PNG)

- Flush privileges:

`flush privileges;`

![Flush Privileges](./images/flush-output.PNG)

- Show databases:

`show databases;`

![Show Databases](./images/show-db-output.PNG)

Step 3 — Prepare the Web Servers

- We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.
You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

- Launch a new EC2 instance with RHEL 8 Operating System.

- Install NFS client:

`sudo yum install nfs-utils nfs4-acl-tools -y`

![Nfs Install](./images/nfs-install-success.PNG)

- Mount /var/www/ and target the NFS server’s export for apps:

`sudo mkdir /var/www`

![Var WWW](./images/mkdir-www.PNG)

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

![Mount](./images/success.PNG)

- Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:

`df -h`

![Mount](./images/df-h-success.PNG)

- To show connect between the webserver1 and nfs server:

- Create a file in the webserver using this command:

`sudo touch /var/www/test.md`

- Go the nfs server command line and run:

`ls /mnt/apps`

- You should get this output:

![Webserver Nfs Connection](./images/webserver-nfs-connect-success.PNG)

- Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`

- and add the following

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

![Webserver Nfs Connection](./images/fstab-output.PNG)

- Install Remi’s repository, Apache and PHP:

`sudo yum install httpd -y`

![Apache Installation](./images/apache-success.PNG)

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![Epel Installation](./images/epel-success.PNG)

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![Remirepo Installation](./images/remirepo-success.PNG)

`sudo dnf module reset php`

![Php Module Reset](./images/php-reset-success.PNG)

`sudo dnf module enable php:remi-7.4`

![Php Enable](./images/php-enable-success.PNG)

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![Php Package](./image/php-package-success.PNG)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

![Php Start Enable](./images/php-start-enable-success.PNG)

`sudo systemctl status php-fpm`

![Php Status](./images/php-status-success.PNG)

`sudo setsebool -P httpd_execmem 1`

![Setse Bool](./images/setsebool-output.PNG)

- Repeat steps 290 - 376 for Webservers 2 & 3.

- Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers:

`ls /var/www` in the webserver

![ls Var Www](./images/ls-var-www-output.PNG)

`ls /mnt/apps` in the nfs server

![ls Mnt Apps](./images/ls-mnt-apps-output.PNG)

- Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs

`ls /var/log` 

![Apache Folder on Webserver](./images/apache-folder-webserver.PNG)

- Repeat steps №310, 330 & 334 to make sure the mount point will persist after reboot:

`sudo mount -t nfs -o rw,nosuid 172.31.44.17:/mnt/logs /var/log/httpd` 

![Mount Success](./images/mount-output.PNG)

- Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`

![Mount Success](./images/etc-fstab-update.PNG)

- Fork the tooling source code from Darey.io Github Account to your Github account. (Learn how to fork a repo here)

- Install git on the webserver:

`sudo yum install git`

![Git Install](./images/git-install-success.PNG)

- Run the command:

`git init`

![Git Init Install](./images/git-init-success.PNG)

- To fork the repo:

`git clone https://github.com/darey-io/tooling.git`

![Git Clone ](./images/git-clone-success.PNG)

- Run the ls command to confirm the clone:

`ls`

![Tooling Clone ](./images/tooling-success.PNG)

- Change the directory to tooling and run ls:

`cd tooling/`

`ls`

![Tooling Clone ](./images/cd-tooling-ls-output.PNG)

- Ensure that the html folder from the repository is deployed to /var/www/html:

Copy the html folder inside the tooling folder to the /var/www:

`sudo cp -R html/. /var/www/html`

![Copy Success](./images/copy-output.PNG)

- To confirm the copy was successful:

`ls /var/www/html`

![Copy Success](./images/www-html-success.PNG)

`ls html`

![Copy Success](./images/www-html-success.PNG)

- If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0

- Change directory back to home:

`cd ..`

- Runt this command:

`sudo setenforce 0`

![Set Enforce](./images/set-enforce.PNG)

- To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabledthen restrt httpd:

`sudo vi /etc/sysconfig/selinux`

![Selinux Set](./images/selinux-disable-output.PNG)

- Start the httpd:

`sudo systemctl start httpd`

`sudo systemctl status httpd`

![Selinux Set](./images/start-status-httpd.PNG)

- Update the website’s configuration to connect to the database (in /var/www/html/functions.php file):

`sudo vi /var/www/html/functions.php`

![Selinux Set](./images/php-update-with-user-cred.PNG)

- Change bindaddress in the database server:

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![Bind Address Change](./images/bind-address-change.PNG)

- Restart and check status My sql:

`sudo systemctl restart mysql`

`sudo systemctl status mysql`

![Restart and Status ](./images/restart-status-output.PNG)

- Apply tooling-db.sql script to your database using this command mysql -h <database-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql:

`mysql -h 172.31.41.255 -u webaccess -p tooling < tooling-db.sql`

![Logging Output](./images/logging-output.PNG)

- Go to the Database server and show databases in mysql

`sudo mysql;`

`show databases;`

![Tooling](./images/show-db.PNG)

`use tooling;`

![Database Change](./images/DB-change.PNG)

`show tables;`

![Show Tables](./images/show-tables.PNG)

`select * from users;`

![Query Tables](./images/query-table.PNG)

- Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the websute with myuser user:

- Launch the Webserver 1 Public ip any browser:

[Webserver Url](http://3.145.23.181/admin_tooling.php)

![Webserver launch Success](./images/admin-login-1.PNG)

![Webserver launch Success](./images/admin-login-2.PNG)