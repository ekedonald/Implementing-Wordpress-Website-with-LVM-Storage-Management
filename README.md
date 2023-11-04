# Implementing a Wordpress Website with LVM Storage Management

## Understanding Three-Tier Architecture

A **Three-Tier Architecture** is a client-server architecture model that separates an application into three interconnected but distinct layers, each responsible for specific aspects of the application's functionality. 

Generally, web or mobile solutions are implemented based on a **Three-Tier Architecture** to improve scalability and flexibility. The three distinct layers are:

1. **Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.

2. **Business Layer (BL)**: This is the backend program that implements business logic (_i.e. Application or Web Server_).

3. **Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access (_i.e. Database Server or File System Server such as FTP Server or NFS Server_).

## What is Logical Volume Manager (LVM)?
LVM stands for Logical Volume Manager, a technology used in Linux and other Unix-like operating systems to manage storage devices and create flexible, resizable storage configurations. LVM provides a layer of abstraction between the physical storage devices (_such as hard drives, SSDs, or partitions_) and the file systems or logical volumes used by the operating system.

Key components of LVM include:

1. **Physical Volumes (PVs)**: These are the physical storage devices or partitions (_i.e. hard drives or SSDs_) that are added to the LVM system.

2. **Volume Groups (VGs)**: Volume Groups are created by combining one or more Physical Volumes. VGs serve as a pool of storage that can be allocated to various Logical Volumes.

3. **Logical Volumes (LVs)**: Logical Volumes are similar to traditional partitions and are created within a Volume Group. They are what you format with a file system and use to store data. Logical Volumes can be resized and moved dynamically, which is a significant advantage of LVM.

## How To Implement a WordPress Website with LVM Storage Management

The following steps are taken to implement a WordPress Website with LVM Storage Management:

### Step 1: Provision a Web Server EC2 Instance

Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Web Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair Name: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Security Group: WordPress
7. Inbound Rules: Allow Traffic From Anywhere On Port 80 and Port 22.

_Instance Summary for Web Server_

* On the Instances tab, you will see the Availability Zone (_i.e. us-east-1d_). This will be used when creating Elastic Block Volumes for the Web Server Instance.

### Step 2: Create and Attach 3 Elastic Block Store Volumes to the Web Server EC2 Instance

* On the EC2 dashboard, click on **Volumes** on the Elastic Block Store tab.

* Click on the **Create volume** button.

* Give the EBS Volume the following parameters and click on the **create volume** button:

1. Size (GiB): 10
2. Availability Zone: us-east-1d (_Note that the Availability Zone you select must match the Availability zone of the Web Server Instance_)

* Repeat the steps above to create two more EBS Volumes.

* You will see the 3 EBS Volumes you created have an **Available** Volume state.

* Click on one of the Volumes then click on the **Actions** button, you will see a drop-down and click on the **Attach volume** option.

* Select the Web Server Instance and click on the Attach volume button.

* Repeat these steps for the other 2 volumes and you will see that the volumes have been attached to the Web Server Instance as shown below:

### Step 3: Implement LVM Storage Management on the Web Server

* Open terminal on your computer.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

* Run the following command to give read permissions to the `.pem` key pair file.

```sh
chmod 400 <private-key-pair-name>.pem
```
* SSH into the Web Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

* Use the `lsblk` command to inspect what block devices are attached to the server.

_Notice the names of the newly created devices. All devices in Linux reside in the **/dev** directory._

* Use the `df -h` command to see all mounts and free space on your server.

* Use `gdisk` utility to create a single partition on **/dev/xvdf** disk.

```sh
sudo gdisk /dev/xvdf
```

* Type `n` to create a new partition and fill in the data shown below into the parameters:

 1. Partition number (1-128, default 1): 1
 2. First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 2048
 3. Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: 20971486
 4. Current type is 8300 (Linux filesystem)
 Hex code or GUID (l to show codes, Enter = 8300): 8300

* Type `p`to print the partition table of the /dev/xvdf device.

* Type `w` to write the table to disk and type `y` to exit.

* Repeat the `gdisk` utility partitioning steps for **/dev/xvdg** and **/dev/xvdh** disks.

* Use the `lsblk` command to view the newly configured partition on each of the 3 disks.

* Install `lvm2` package using the command shown below:

```sh
sudo yum install lvm2 -y
```

* Run the following command to check for available partitons:

```sh
sudo lvmdiskscan
```

* Use `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs) to be used by LVM.

```sh
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

* Verify that your physical volumes (PVs) have been created successfully by running `sudo pvs`

* Use `vgcreate` utility to add 3 physical volumes (PVs) to a volume group (VG). Name the volume group **webdata-vg**.

```sh
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```

* Verify that your volume group (VG) has been created successfully by running `sudo vgs`

* Use `lvcreate` utility to create 2 logical volumes: **apps-lv (use half of the PV size)** and **logs-lv (use the remaining space of the PV size)**. _Note that **apps-lv** will be used to store data for the website while **logs-lv** will be used to store data for logs._

```sh
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

* Verify that your logical volume (LV) has been created successfully by running `sudo lvs`

* Verify the entire setup

```sh
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```

* Use `mkfs.ext4` to format the logical volumes (LV) with **ext4** file system.

```sh
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

* Create **/var/www/html** directory to store website files.

```sh
sudo mkdir -p /var/www/html
```

* Create **/home/recovery/logs** to store backup of log data.

```sh
sudo mkdir -p /home/recovery/logs
```

* Mount **/var/www/html** on **apps-lv** logical volume.

```sh
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

* Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (_This is required before mounting the file system._)

```sh
sudo rsync -av /var/log/. /home/recovery/logs/
```

* Mount **/var/log** on **logs-lv** logical volume. (_Note that all the existing data on **/var/log** will be deleted_).

```sh
sudo mount /dev/webdata-vg/logs-lv /var/log
```

* Restore log files back into **/var/log** directory.

```sh
sudo rsync -av /home/recovery/logs/ /var/log
```

* Update `/etc/fstab` file so that the mount configuration will persist after restarting the server. The UUID of the device will be used to update the `/etc/fstab` file. Run the command shown below to get the UUID of the **apps-lv** and **logs-lv** logical volumes:

```sh
sudo blkid
```

* Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending quotes.

* Test the configuration using the command shown below:

```sh
sudo mount -a
```

* Reload the daemon using the command shown below:

```sh
sudo systemctl daemon-reload
```

* Verify your setup by running `df -h`.

### Step 4: Install WordPress on the Web Server

* Update the list of packages in the package manager.

```sh
sudo yum -y update
```

* Install wget, apache and its dependencies.

```sh
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

* Start apache

```sh
sudo systemctl enable httpd
```

```sh
sudo systemctl start httpd
```

* Install PHP and its dependencies.

```sh
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```

* Restart apache.

```sh
sudo systemctl restart httpd
```

* Download WordPress and copy WordPress to `/var/www/html`

```sh
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```

* Configure SELinux policies.

```sh
 sudo chown -R apache:apache /var/www/html/wordpress
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 sudo setsebool -P httpd_can_network_connect=1
```

### Step 5: Provision a Database Server EC2 Instance
Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Database Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. Key Pair Name: web11
4. New Security Group: WordPress
5. Inbound Rules: Allow Traffic From Anywhere On Port 22 and Traffic from the Private IPv4 address of the Web Server on Port 3306 (_i.e. MySQL_).

_Instance Summary for Database Server_

### Step 6: Create and Attach 3 Elastic Block Store Volumes to the Database Server EC2 Instance

* Repeat Step 3 but attach the Volumes to the Database Server and ensure the volumes are attached to the Availability Zone (_i.e. us-east-1c_) of the Database Server.

_The EBS Volumes have been attached to the Database Server_

### Step 7: Install MySQL on the Database Server

* Open another terminal on your computer.

* Go to the Downloads directory (i.e `.pem` key pair is stored here) using the command shown below:

```sh
cd Downloads
```

* SSH into the Database Server Instance using the command shown below:

```sh
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

* Update the list of packages in the package manager.

```sh
sudo yum update -y
```

* Install MySQL server.

```sh
sudo yum install mysql-server -y
```

* Verify that the service is up and running.

```sh
sudo systemctl status mysqld
```

* Enable the MySQL service.

```sh
sudo systemctl enable mysqld
```

* Restart the MySQL service.

```sh
sudo systemctl restart mysqld
```

### Step 8: Configure the Database Server to work with WordPress

* Log into the MySQL console application.

```sh
sudo mysql
```

* Create a database called **wordpress**.

```sh
CREATE DATABASE wordpress;
```

* Create a new user.

```sh
CREATE USER 'myuser'@'<web_server_private_ip_address>' IDENTIFIED BY 'mypass';
```

* Grant all privileges on the wordpress database to the user you created.

```sh
GRANT ALL ON wordpress.* TO 'myuser'@'<web_server_private_ip_address>';
```

* Run the following command to apply and make changes effective.

```sh
FLUSH PRIVILEGES;
```

* Display all the databases.

```sh
SHOW DATABASES;
```

* Exit the MySQL console.

### Step 9: Configure WordPress to connect to the remote Database Server

* Connect to the Web Server Instance.

* Install MySQL client.

```sh
sudo yum install mysql -y
```

* Test that you can connect from your Web Server to your Database Server by using `mysql-client`

```sh
sudo mysql -u admin -p -h <Database_Server_Private_IP_address>
```

* Verify if you can successfully execute `SHOW DATABAES;` command to see a list of existing databases.

* Run the following command to configure WordPress to establish connection with the Database Server.

```sh
sudo vi /var/www/html/wordpress/wp-config.php
```

_The highlighted parameters are the ones that need to be comfigured_

* Input the credentials of the user you created when configuring the Database Server then save and exit the file.

* Try to access the URL shown below from your browser:

```sh
http://<Web_Server_Public_IP_Address>/wordpress/
```
