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

Name of Instance: Web Server
AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
New Key Pair Name: web11
Key Pair Type: RSA
Private Key File Format: .pem
New Security Group: WordPress
Inbound Rules: Allow Traffic From Anywhere On Port 80 and Port 22.

* On the Instances tab, you will see the Availabilty Zone. This will be used when creating Elastic Block Volumes for the Web Server Instance.

### Step 2: Create and Attach 3 Elastic Block Store Volumes to the Web Server EC2 Instance

* On the EC2 dashboard, click on **Volumes** on the Elastic Block Store tab.

* Click on the **Create volume** button.

* Give the EBS Volume the following parameters:

Size (GiB): 10
Availability Zone: us-east-1d (_Note that the Availabiltiy Zone you select must match the Availability zone of the Web Server Instance_)

* Click on the create volume button.

* Repeat the steps above to create two more EBS Volumes.

* You will see the 3 EBS Volumes you created have an **Available** Volume state.

* Click on one of the Volumes then click on the Actions button, you will see a drop down and click on the Attach volume option.

* Select the Web Server Instance and click on the Attach volume button.

* Repeat these steps for the other 2 volumes and you will see that the volumes have been attached to the Web Server Instance as shown below:

### Step 3: Implement LVM Storage Management on the Web Server

### Step 4: Install Wordpress on the Web Server

### Step 5: Provision a Database Server EC2 Instance
Use the following parameters when configuring the EC2 Instance:

Name of Instance: Database Server
AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
Key Pair Name: web11
New Security Group: WordPress
Inbound Rules: Allow Traffic From Anywhere On Port 22 and Traffic from the Private IPv4 address of the Web Server.

* On the Instances tab, you will see the Availabilty Zone. This will be used when creating Elastic Block Volumes for the Web Server Instance.

### Step 6: Create and Attach 3 Elastic Block Store Volumes to the Database Server EC2 Instance

* Repeat Step 3 but attach the Volumes to the Database Server and ensure the volumes are attached to the Availability Zone (_i.e. us-east-1c_) of the Database Server.

### Step 7: Install MySQL on the Database Server

### Step 8: Configure the Database Server to work with WordPress

### Step 9: Configure Wordpress to connect to the remote Database Server
