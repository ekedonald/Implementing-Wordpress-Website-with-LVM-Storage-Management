# Implementing a Wordpress Website with LVM Storage Management
## Understanding Three-Tier Architecture
A **Three-Tier Architecture** is a client-server architecture model that separates an application into three interconnected but distinct layers, each responsible for specific aspects of the application's functionality. 

Generally, web or mobile solutions are implemented based on **Three-Tier Architecture** to improve scalability and flexibility. The three distinct layers are:

**1. Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.

**2. Business Layer (BL)**: This is the backend program that implements business logic (_i.e. Application or Web Server_).

**3. Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access (_i.e. Database Server or File System Server such as FTP Server or NFS Server_).

## How To Implement a Wordpress Website with LVM Storage Management
The following steps are taken to implement a Wordpress Website with LVM Storage Management:

### Step 1: Provision a Web Server EC2 Instance
Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Web Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair Name: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Security Group: WordPress
7. Inbound Rules: Allow Traffic From Anywhere On Port 80 and Port 22.

* On the Instances tab, you will see the Availabilty Zone (_i.e. us-east-1d_). This will be used when creating Elastic Block Volumes for the Web Server Instance.

### Step 2: Create and Attach 3 Elastic Block Store Volumes to the Web Server EC2 Instance

* On the EC2 dashboard, click on **Volumes** on the Elastic Block Store tab.

* Click on the **Create volume** button.

* Give the EBS Volume the following parameters:

1. Size (GiB): 10
2. Availability Zone: us-east-1d (_Note that the Availabiltiy Zone you select must match the Availability zone of the Web Server Instance_)

* Click on the create volume button.

* Repeat the steps above to create two more EBS Volumes.

* You will see the 3 EBS Volumes you created have an **Available** Volume state.

* Click on one of the Volumes then click on the Actions button, you will see a drop down and click on the Attach volume option.

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

_Notice the names of the new created devices. All devices in Linux reside in the **/dev** directory._

* Use the `df -h` command to see all mounts and free space on your server.

* Use `gdisk` utility to create a single partiton on **/dev/xvdf** disk.

```sh
sudo gdisk /dev/xvdf
```

* Type `n` to create a new partition and fill in the data shown below into the parameters:

 1. Partiton number (1-128, default 1): 1
 2. First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 2048
 3. Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: 20971486
 4. Current type is 8300 (Linux filesystem)
 Hex code or GUID (l to show codes, Enter = 8300): 8300

* Type `p`to print the partition table of the /dev/xvdf device.

* Type `w` to write the table to disk and type `y` to exit.

* Repeat the `gdisk` utility partitioning steps for **/dev/xvdg** and **/dev/xvdh** disks.

* Use the `lsblk` command to view the newly configured partiton on each of the 3 disks.

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


### Step 4: Install Wordpress on the Web Server

### Step 5: Provision a Database Server EC2 Instance
Use the following parameters when configuring the EC2 Instance:

1. Name of Instance: Database Server
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. Key Pair Name: web11
4. New Security Group: WordPress
5. Inbound Rules: Allow Traffic From Anywhere On Port 22 and Traffic from the Private IPv4 address of the Web Server.

* On the Instances tab, you will see the Availabilty Zone (_i.e us-east-1c_). This will be used when creating Elastic Block Volumes for the Web Server Instance.

### Step 6: Create and Attach 3 Elastic Block Store Volumes to the Database Server EC2 Instance

* Repeat Step 3 but attach the Volumes to the Database Server and ensure the volumes are attached to the Availability Zone (_i.e. us-east-1c_) of the Database Server.

### Step 7: Install MySQL on the Database Server

### Step 8: Configure the Database Server to work with WordPress

### Step 9: Configure Wordpress to connect to the remote Database Server
