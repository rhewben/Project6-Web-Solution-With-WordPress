### WEB SOLUTION WITH WORDPRESS

## Three-tier Architecture

![three-tier](./images/three-tier.png)

# Your 3-Tier Setup

*An EC2 Linux Server as a web server (This is where you will install WordPress),An EC2 Linux server as a database (DB) server*

*Use RedHat OS for this project*

## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.

![EC2](./images/EC2%20instances.png)

# Step 1 — Prepare a Web Server

*Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB*

![create](./images/creating%20volumes.png)

![newvolumes](./images/newvolumes.png)

# Attach all three volumes one by one to your Web Server EC2 instance

![attach vol](./images/attache%20volume.png)

# Open up the Linux terminal to begin configuration

![webserver](./images/webserver1.png)

*to see availabe mount points*

`df -h`
![mntpoints](./images/mntpoints.png)

*Use gdisk utility to create a single partition on each of the 3 disks*

`sudo gdisk /dev/....`

*where the ...indicates the names of your volumes*

![partition](./images/partition.png)

*same is done to the othe two volumes*

![partition2](./images/partition2.png)

*Use lsblk utility to view the newly configured partition on each of the 3 disks.*

`lsblk`

![lsblk](./images/lsblk.png)

*Install lvm2 package using*. 

`sudo yum install lvm2`

*Run the command below to check for available partitions*.

`sudo lvmdiskscan`

![diskscan](./images/diskscan.png)

*Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM*

`sudo pvcreate /dev/nvme1n1p1`
`sudo pvcreate /dev/nvme2n1p1`
`sudo pvcreate /dev/nvme3n1p1`

![physicalvol](./images/physical%20volumes.png)

*Verify that your Physical volume has been created successfully by running the below*

`sudo pvs`

![physicalvol](./images/volconfirmation.png)

*Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG (webdata-vg)*

`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

*Verify that your VG has been created successfully by running*

`sudo vgs`

![vgs](./images/vgs.png)

*Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs*

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

![lvs](./images/lvs.png)

*Verify the entire setup*

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

*Use mkfs.ext4 to format the logical volumes with ext4 filesystem*

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![format](./images/formatting.png)

*Create /var/www/html directory to store website files*

`sudo mkdir -p /var/www/html`

*Create /home/recovery/logs to store backup of log data*

`sudo mkdir -p /home/recovery/logs`

*Mount /var/www/html on apps-lv logical volume*

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

*Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)*

`sudo rsync -av /var/log/. /home/recovery/logs/`

*Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important*

`sudo mount /dev/webdata-vg/logs-lv /var/log`

*Restore log files back into /var/log directory*

`sudo rsync -av /home/recovery/logs/log/. /var/log`

*Update /etc/fstab file so that the mount configuration will persist after restart of the server.The UUID of the device will be used to update the /etc/fstab file;*

`sudo blkid`

![blkid](./images/blkid.png)

*Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.*

`sudo vi  /etc/fstab`

![uuid](./images/uuid.png)

*Test the configuration and reload the daemon*

`sudo mount -a`

`sudo systemctl daemon-reload`

![check](./images/check.png)

*Verify your setup by running, output must look like this:*

`df -h`

![verify](./images/verify.png)



## PREPARE THE DATABASE SERVER

*Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/*

![creatingdb vol](./images/creating%20db%20vol.png)

![creatingdb vol](./images/creating%20db%20vol%202.png)

*attach the new volumes to the database instance*

![vol attaching](./images/attachdbvol.png)

*check the drives status*

`lsblk`

![lsblk](./images/lsblk.png)

*creating partitions*

![db partition](./images/db%20partition.png)

*check the new partitions*

`lsblk`

![newpartitions](./images/newpartitions.png)


*install LVM*

`sudo yum install lvm2 -y`

*Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM for the DB*


`sudo pvcreate /dev/nvme1n1p1`
`sudo pvcreate /dev/nvme2n1p1`
`sudo pvcreate /dev/nvme3n1p1`

![pvcreate](./images/pvcrveate.png)

*Verify that your Physical volume for the db has been created successfully by running the below*

`sudo pvs`

*Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG (db-vg)*

`sudo vgcreate database-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

`sudo vgs`

![vgs](./images/vgs2.png)

*Use lvcreate utility to create 2 logical volumes. db-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size*

`sudo lvcreate -n db-lv -L 20G database-vg`

*create a directory*

`sudo mkdir /db`

![lvcreate](./images/lvcreate.png)

*Use mkfs.ext4 to format the logical volumes with ext4 filesystem*

`sudo mkfs.ext4 /dev/database-vg/db-lv`

*mount to the logical volume*

![mount](./images/mnt2.png)

*Update /etc/fstab file so that the mount configuration will persist after restart of the server.The UUID of the device will be used to update the /etc/fstab file;*

`sudo blkid`

![blkid](./images/blkid2.png)

*Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.*

`sudo vi  /etc/fstab`

![uuid](./images/uuid2.png)

*Test the configuration and reload the daemon*

`sudo mount -a`

`sudo systemctl daemon-reload`


## Step 3 — Install WordPress on your Web Server EC2

`sudo yum -y update`

*Install wget, Apache and it’s dependencies*

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

*Start Apache*

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

![apache](./images/apache.png)

*Installing php and its dependencies*

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo yum module list php`

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

*Restart Apache*

`sudo systemctl restart httpd`

*Downloading wordpress and moving it into the web content directory*

`mkdir wordpress`

`cd   wordpress`

`sudo wget http://wordpress.org/latest.tar.gz`

`sudo tar xzvf latest.tar.gz`

`sudo rm -rf latest.tar.gz`

`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`

`sudo cp -R wordpress /var/www/html/`

*Configure SELinux Policies*

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chcon -t httpd_sys_rw_content_t /var/www/html wordpress -R`

`sudo setsebool -P httpd_can_network_connect=1`


# Step 4 — Install MySQL on your DB Server EC2

`sudo yum update`

`sudo yum install mysql-server`

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

![mysqld](./images/mqlsd%20status.png)


# Step 5 — Configure DB to work with WordPress

`sudo mysql`

`CREATE DATABASE wordpress;`

`CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';`

`GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';`

`FLUSH PRIVILEGES`

`SHOW DATABASES`


![show databases](./images/sh%20db.png)

# Step 6 — Configure WordPress to connect to remote database.

![inbound](./images/inbound%20rules.png)


*Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client*

`sudo yum install mysql`

`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

*Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases from the webserver*

![show databases](./images/sh%20db2.png)

*Change permissions and configuration so Apache could use WordPress*

`# mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`


*in the database*

`sudo vi /etc/my.cnf/`

*add bind to it*

![bind](./images/bind.png)

`systemctl restart mysqld`

*go to the webserver*
`sudo vi wp-config.php`

`sudo systemctl restart httpd`


![bind](./images/bind%202.png)

*go to the webpage*

*http://<Web-Server-Public-IP-Address>*

![connection](./images/connection%20.png)

![connection](./images/connection%202.png)




















