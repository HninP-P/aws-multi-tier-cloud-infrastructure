# AWS Multi-Tier Cloud Infrastructure & Pilot Solution — Mercury Computing Solutions (MCS)

### Architecture diagram for MCS
![Architecture diagram for MCS](<images/Architecture diagram for MCS.png>)
<div align="center"><b>Figure: High-Availability Cloud Architecture Topology</b></div>

<br><br>

## 📌 Project Overview
This project delivers a secure, highly available, and scalable cloud infrastructure on **Amazon Web Services (AWS)** (Singapore Region — `ap-southeast-1`) designed for **Mercury Computing Solutions (MCS)**. 

The architecture transitions a traditional on-premise development network into an isolated, multi-tier cloud environment. It enables remote software engineers to securely develop, test, and deploy web applications while ensuring strict network isolation, automated load distribution, centralized monitoring, and automated fault remediation.

---

## 🏗️ Network Architecture & Subnet Planning

The core network foundation is built inside a custom **Amazon Virtual Private Cloud (VPC)** utilizing an IPv4 CIDR block of `192.168.2.0/24`. The network space is sub-netted across multiple Availability Zones to decouple public-facing traffic from sensitive database and backend compute workloads.

- **VPC:** `MCS-HPP-VPC` (`192.168.2.0/24`)
- **Public Subnet AZ-a:** `MCS-HPP-Public-AZ-a` (`192.168.2.0/26`) — Access Load Balancers & Primary Web Server
- **Public Subnet AZ-b:** `MCS-HPP-Public-AZ-b` (`192.168.2.64/26`) — Secondary Public Web Node & Management Mounts
- **Private Subnet AZ-c:** `MCS-HPP-Private-AZ-c` (`192.168.2.128/26`) — Isolated Storage Mounts & Database Secondary
- **Private Subnet AZ-b:** `MCS-HPP-Private-AZ-b` (`192.168.2.192/26`) — Managed Database Client, RDS Instance, & CloudWatch Test 

---

## ⚙️ Core AWS Infrastructure Components & Configuration

### 1️⃣ Virtual Private Cloud (VPC), Routing & NAT Access
- **Internet Gateway (`MCS-HPP-IGW`):** Handles inbound and outbound traffic for public-facing assets.
- **NAT Gateway (`MCS-HPP-NAT-GW`):** Deployed in the public subnet and bound to Elastic IP (`MCS-HPP-EIP` / `47.130.120.54`). Allows private instances to perform outbound software updates (`apt update` / `yum update`) while blocking unsolicited inbound connections.

<details>
<summary><b>📷 View Network Setup & Gateways</b></summary>

#### VPC & Subnet Segmentation
![MCS VPC](<images/MCS VPC.png>)
<div align="center"><b>Figure: Custom Amazon VPC Dashboard</b></div>

<br><br>

![Subnets division for MCS](<images/Subnets division for MCS.png>)
<div align="center"><b>Figure: Subnet Division Overview</b></div>

<br><br>

#### VPC Creation & Subnet Configurations
![Create VPC](<images/Create VPC.png>)
<div align="center"><b>Figure: Creating the VPC Resource</b></div>

<br><br>

![Create subnet](<images/Create subnet.png>)
<div align="center"><b>Figure: Creating Subnets within the VPC</b></div>

<br><br>

![Subnet 1](<images/Subnet 1 (MCS-HPP-Public-AZ-a).png>)
<div align="center"><b>Figure: Public Subnet AZ-a Configuration</b></div>

<br><br>

![Subnet 2](<images/Subnet 2 (MCS-HPP-Public-AZ-b).png>)
<div align="center"><b>Figure: Public Subnet AZ-b Configuration</b></div>

<br><br>

![Subnet 3](<images/Subnet 3 (MCS-HPP-Private-AZ-c).png>)
<div align="center"><b>Figure: Private Subnet AZ-c Configuration</b></div>

<br><br>

![Subnet 4](<images/Subnet 4 (MCS-HPP-Private-AZ-b).png>)
<div align="center"><b>Figure: Private Subnet AZ-b Configuration</b></div>

<br><br>

#### Internet Gateway Attachment
![Create internet gateway](<images/Create internet gateway.png>)
<div align="center"><b>Figure: Creating Internet Gateway</b></div>

<br><br>

![Attach IGW](<images/Attach IGW to MCS-HPP-VPC.png>)
<div align="center"><b>Figure: Attaching IGW to Custom VPC</b></div>

<br><br>

![IGW list](<images/IGW list.png>)
<div align="center"><b>Figure: Internet Gateways List</b></div>

<br><br>

#### Route Tables (Public & Private)
![Create public route table](<images/Create public rout table (MCS-HPP-Public-RT).png>)
<div align="center"><b>Figure: Creating Public Route Table</b></div>

<br><br>

![Subnet associations public](<images/Subnet associations for public rout table.png>)
<div align="center"><b>Figure: Public Subnet Route Associations</b></div>

<br><br>

![Edit routes public](<images/Edit routes for public rout table with MCS-HPP-IGW.png>)
<div align="center"><b>Figure: Editing Public Routes with IGW</b></div>

<br><br>

![Create private route table](<images/Create private rout table (MCS-HPP-Private-RT).png>)
<div align="center"><b>Figure: Creating Private Route Table</b></div>

<br><br>

![Subnet associations private](<images/Subnet associations for private rout table.png>)
<div align="center"><b>Figure: Private Subnet Route Associations</b></div>

<br><br>

![Route tables list](<images/Rout tables list.png>)
<div align="center"><b>Figure: Route Tables Console List</b></div>

<br><br>

#### Elastic IP Allocation & NAT Gateway
![Allocate Elastic IP](<images/Allocate Elastic IP (MCS-HPP-EIP).png>)
<div align="center"><b>Figure: Allocating Elastic IP for NAT</b></div>

<br><br>

![Create NAT gateway](<images/Create NAT gateway (MCS-HPP-NAT-GW).png>)
<div align="center"><b>Figure: Creating NAT Gateway</b></div>

<br><br>

![Edit routes private NAT](<images/Edit routes for private rout table with MCS-HPP-NAT-GW.png>)
<div align="center"><b>Figure: Editing Private Route Table with NAT Gateway</b></div>

</details>

---

### 2️⃣ Bastion Jump Host, SSH Keys & NAT Reachability
- **Public Jump Server (`MCS-HPP-WebServer`):** Deployed on Ubuntu 24.04 LTS (`t3.micro`) in the public subnet with an auto-assigned public IP (`13.229.74.217`).
- **Private Compute Node (`MCS-HPP-NAT-Server`):** Situated in `MCS-HPP-Private-AZ-b` with public IP disabled (`192.168.2.216`).
- **Secure Access Workflow:** Managed access via key-pair (`db.pem`) with strict permissions (`chmod 400`). Outbound ICMP reachability (`ping 8.8.8.8`) verified active internet routing through the NAT gateway.

<details>
<summary><b>📷 View Instance Launch & SSH Access Logs</b></summary>

#### Public Web Server Launch Settings
![Launch web server](<images/Launch web server instance (MCS-HPP-WebServer).png>)
<div align="center"><b>Figure: Launching Web Server Instance</b></div>

<br><br>

![Instance type and key pair](<images/Instance type and key pair of web server.png>)
<div align="center"><b>Figure: Instance Type & Key Pair Configuration</b></div>

<br><br>

![Network settings](<images/Network settings of web server.png>)
<div align="center"><b>Figure: Network Settings & Security Group</b></div>

<br><br>

![Inbound rules 1](<images/Inbound security group rules of web server security group.png>)
<div align="center"><b>Figure: Inbound Security Group Rules (SSH & HTTP)</b></div>

<br><br>

![Inbound rules 2](<images/Inbound security group rules of web server security group(1).png>)
<div align="center"><b>Figure: Inbound Security Group Rules (ICMP Ping)</b></div>

<br><br>

![Configuration storage](<images/Configuration storage of web server.png>)
<div align="center"><b>Figure: Storage Configuration</b></div>

<br><br>

#### Private NAT Server Launch Settings
![Launch NAT server](<images/Launch NAT server instance (MCS-HPP-NAT-Server).png>)
<div align="center"><b>Figure: Launching NAT Server Instance</b></div>

<br><br>

![Instance type NAT](<images/Instance type and key pair of NAT server.png>)
<div align="center"><b>Figure: NAT Server Instance Type & Key Pair</b></div>

<br><br>

![Network settings NAT](<images/Network settings of NAT server.png>)
<div align="center"><b>Figure: NAT Server Network Settings</b></div>

<br><br>

![Configure storage NAT](<images/Configure storage of NAT server.png>)
<div align="center"><b>Figure: NAT Server Storage Configuration</b></div>

<br><br>

#### Jump Box SSH Terminal Sessions & Key Management
![Connect to WebServer](<images/Connect to MCS-HPP-WebServer.png>)
<div align="center"><b>Figure: Connecting to WebServer via EC2 Instance Connect</b></div>

<br><br>

![Private SSH attempt](<images/Private SSH attempt and key file creation.png>)
<div align="center"><b>Figure: Private Key File Creation</b></div>

<br><br>

![SSH key paste](<images/SSH private key paste (db.pem).png>)
<div align="center"><b>Figure: Pasting Private Key Contents (`db.pem`)</b></div>

<br><br>

![SSH Access private instance](<images/SSH Access to private instance.png>)
<div align="center"><b>Figure: Successful SSH Access to Private Instance</b></div>

<br><br>

#### Internet Connectivity Ping Test
![Ping Test Verification](<images/Internet reachability ping test from private instance.png>)
<div align="center"><b>Figure: Internet Reachability Ping Test from Private Instance</b></div>

<br><br>

#### Elastic IP Association to Private Node
![Elastic IP target selection](<images/Elastic IP target selection (MCS-HPP-NAT-server).png>)
<div align="center"><b>Figure: Selecting Target for Elastic IP Association</b></div>

<br><br>

![Private IP address mapping](<images/Private IP address mapping (192.168.2.216).png>)
<div align="center"><b>Figure: Private IP Address Mapping</b></div>

<br><br>

![Elastic IP association confirmation](<images/Elastic IP association confirmation.png>)
<div align="center"><b>Figure: Elastic IP Association Confirmation</b></div>

<br><br>

![Elastic IP Ping Success](<images/Internet reachability ping success from private instance.png>)
<div align="center"><b>Figure: Successful Ping Verification via Elastic IP</b></div>

</details>

---

### 3️⃣ High Availability Application Tier & Elastic Load Balancing (ELB)
- **Web Nodes (`MCS-HPP-Website1` & `MCS-HPP-Website2`):** Provisioned on Amazon Linux 2023 with custom bash bootstrap scripts (`user data`) to install Apache (`httpd`) and serve dynamic instance hostnames.
- **Target Group (`MCS-HPP-TG`):** Registered web instances on HTTP port 80 with active health checks on the root path (`/`).
- **Application Load Balancer (`MCS-HPP-ELB`):** Internet-facing ALB configured across multiple public AZs. Verified round-robin traffic balancing by loading and refreshing the ALB DNS endpoint.

<details>
<summary><b>📷 View Web Instance & Load Balancer Verification</b></summary>

#### Website Node Deployments & Bootstrap Scripts
![Website 1 name](<images/Website 1 name and AMI selection.png>)
<div align="center"><b>Figure: Website 1 Name & AMI Selection</b></div>

<br><br>

![Website 1 type](<images/Website 1 instance type and key pair.png>)
<div align="center"><b>Figure: Website 1 Instance Type & Key Pair</b></div>

<br><br>

![Website 1 network](<images/Website 1 network settings and security group name.png>)
<div align="center"><b>Figure: Website 1 Network Settings</b></div>

<br><br>

![Website 1 rules](<images/Website 1 inbound rules.png>)
<div align="center"><b>Figure: Website 1 Inbound Rules</b></div>

<br><br>

![Website 1 storage](<images/Website 1 storage configuration.png>)
<div align="center"><b>Figure: Website 1 Storage Configuration</b></div>

<br><br>

![Website 1 user data](<images/Website 1 user data Apache web server script.png>)
<div align="center"><b>Figure: Website 1 Bootstrap Script (`user data`)</b></div>

<br><br>

#### Multi-Node Launch & Direct IP Verification
![Launch more like this](<images/Launch more like this template selection from MCS-HPP-Website1.png>)
<div align="center"><b>Figure: Launch More Like This Template Selection</b></div>

<br><br>

![Website 2 name](<images/Website 2 instance name tag.png>)
<div align="center"><b>Figure: Website 2 Instance Name Tag</b></div>

<br><br>

![Website 2 script](<images/Custom bootstrap script modification for Website 2.png>)
<div align="center"><b>Figure: Website 2 Custom Bootstrap Script</b></div>

<br><br>

![Created Website instances list](<images/Created Website instances list.png>)
<div align="center"><b>Figure: Created Website Instances List</b></div>

<br><br>

![Copy Website 1 IP](<images/Copy Website 1 public IPv4 address (47.131.250.113).png>)
<div align="center"><b>Figure: Copying Website 1 Public IPv4 Address</b></div>

<br><br>

![Website 1 browser access](<images/Website 1 browser access and response validation.png>)
<div align="center"><b>Figure: Website 1 Browser Access & Validation</b></div>

<br><br>

![Copy Website 2 IP](<images/Copy Website 2 public IPv4 address (47.129.160.193).png>)
<div align="center"><b>Figure: Copying Website 2 Public IPv4 Address</b></div>

<br><br>

![Website 2 browser access](<images/Website 2 browser access and response validation.png>)
<div align="center"><b>Figure: Website 2 Browser Access & Validation</b></div>

<br><br>

#### Target Group Setup & Health Checks
![Target type selection](<images/Target type selection and Target group name.png>)
<div align="center"><b>Figure: Target Type & Target Group Name Selection</b></div>

<br><br>

![Selecting protocol](<images/Selecting protocol, port, IP address type, and VPC.png>)
<div align="center"><b>Figure: Protocol, Port & VPC Configuration</b></div>

<br><br>

![Health checks](<images/Health checks.png>)
<div align="center"><b>Figure: Target Group Health Check Settings</b></div>

<br><br>

![Target optimizer](<images/Target optimizer setting.png>)
<div align="center"><b>Figure: Target Group Advanced Settings</b></div>

<br><br>

![Registering instances](<images/Registering MCS-HPP-Website1 and MCS-HPP-Website2 instances on port 80.png>)
<div align="center"><b>Figure: Registering Web Instances to Target Group</b></div>

<br><br>

![Reviewing pending targets](<images/Reviewing pending targets and instance details before confirmation.png>)
<div align="center"><b>Figure: Reviewing Pending Targets</b></div>

<br><br>

![Final target list review](<images/Final target list review for target group creation.png>)
<div align="center"><b>Figure: Final Target List Review</b></div>

<br><br>

![Created target group list](<images/Created target group list.png>)
<div align="center"><b>Figure: Created Target Group List</b></div>

<br><br>

#### Application Load Balancer Creation
![Load balancer type](<images/Load balancer type selection.png>)
<div align="center"><b>Figure: Load Balancer Type Selection</b></div>

<br><br>

![Load balancer config](<images/Load balancer name, internet-facing scheme, and IPv4 type configuration.png>)
<div align="center"><b>Figure: ALB Name & Scheme Configuration</b></div>

<br><br>

![Network mapping](<images/Network mapping selection.png>)
<div align="center"><b>Figure: ALB Network Mapping Selection</b></div>

<br><br>

![Availability zones](<images/Availability zones and subnets selection.png>)
<div align="center"><b>Figure: Availability Zones & Subnets Selection</b></div>

<br><br>

![Security group association](<images/Security group association.png>)
<div align="center"><b>Figure: ALB Security Group Association</b></div>

<br><br>

![Listeners and routing](<images/Listeners and routing settings.png>)
<div align="center"><b>Figure: ALB Listeners & Routing Settings</b></div>

<br><br>

![Created load balancer list](<images/Created load balancer list.png>)
<div align="center"><b>Figure: Created Load Balancer List</b></div>

<br><br>

![Copy DNS name](<images/Copy DNS name from created load balancer.png>)
<div align="center"><b>Figure: Copying ALB DNS Endpoint</b></div>

<br><br>

#### ALB Load Balancing Verification (Round-Robin Browser Refresh)
![Load balancer browser access 1](<images/Load balancer browser access showing MCS Website - 1.png>)
<div align="center"><b>Figure: ALB Response - Website 1</b></div>

<br><br>

![Load balancer browser access 2](<images/Browser Page refresh switching traffic to MCS Website - 2.png>)
<div align="center"><b>Figure: ALB Response - Website 2 (Round-Robin Traffic Switch)</b></div>

</details>

---

### 4️⃣ Amazon EFS Shared Storage & Cross-Server Synchronization
- **Shared Storage (`MCS-HPP-EFS`):** Provisioned Amazon EFS with mount targets established across all subnets (`ap-southeast-1a`, `1b`, `1c`) tied to dedicated security group `MCS-EFS-Secgp` (NFS port 2049).
- **Multi-Server Mount & Sync:** Attached file system to `/efs` on compute instances (`MCS-HPP-EFS-Server1` and `MCS-HPP-EFS-Server2`) using `amazon-efs-utils`. Verified real-time file creation (`hpp.txt`) and directory synchronization across instances.

<details>
<summary><b>📷 View EFS Configuration & Real-Time Sync</b></summary>

#### EFS Creation & Mount Targets
![EFS file system name](<images/EFS file system name and custom VPC selection.png>)
<div align="center"><b>Figure: EFS File System Name & VPC Selection</b></div>

<br><br>

![Created file system](<images/Created file system.png>)
<div align="center"><b>Figure: Successfully Created EFS File System</b></div>

<br><br>

![Security group creation NFS](<images/Security group creation with an inbound rule for NFS traffic.png>)
<div align="center"><b>Figure: EFS Security Group with NFS Inbound Rule</b></div>

<br><br>

![Outbound rules configuration](<images/Outbound rules configuration allowing all traffic.png>)
<div align="center"><b>Figure: EFS Security Group Outbound Rules</b></div>

<br><br>

![Security group creation success](<images/Security group creation success confirmation.png>)
<div align="center"><b>Figure: Security Group Creation Confirmation</b></div>

<br><br>

![EFS dashboard Network tab](<images/EFS dashboard highlighting the Network tab and Manage.png>)
<div align="center"><b>Figure: EFS Dashboard Network & Mount Target Management</b></div>

<br><br>

![Configuring EFS mount targets](<images/Configuring EFS mount targets across availability zones.png>)
<div align="center"><b>Figure: Configuring EFS Mount Targets Across AZs</b></div>

<br><br>

#### EFS Test Server Launches
![EFS server 1 selection](<images/EFS server 1 instance name and AMI selection.png>)
<div align="center"><b>Figure: EFS Server 1 Instance & AMI Selection</b></div>

<br><br>

![Instance type EFS](<images/Instance type and Key pair selection.png>)
<div align="center"><b>Figure: EFS Server 1 Instance Type & Key Pair</b></div>

<br><br>

![Network settings EFS](<images/Network Settings.png>)
<div align="center"><b>Figure: EFS Server 1 Network Settings</b></div>

<br><br>

![Create security group EFS](<images/Create security group name.png>)
<div align="center"><b>Figure: EFS Server Security Group Setup</b></div>

<br><br>

![Inbound rules SSH](<images/Inbound security rules for SSH and ICMP ping.png>)
<div align="center"><b>Figure: EFS Server Inbound Security Rules</b></div>

<br><br>

![Storage config EFS](<images/Storage configuration and Instance launch summary.png>)
<div align="center"><b>Figure: EFS Server Storage Configuration & Launch Summary</b></div>

<br><br>

![EFS server 2 selection](<images/EFS server 2 instance name and AMI selection.png>)
<div align="center"><b>Figure: EFS Server 2 Instance & AMI Selection</b></div>

<br><br>

![Network settings server 2](<images/Network settings of EFS server 2.jpg>)
<div align="center"><b>Figure: EFS Server 2 Network Settings</b></div>

<br><br>

![Created EFS servers list](<images/Created EFS servers list.png>)
<div align="center"><b>Figure: Created EFS Test Servers List</b></div>

<br><br>

![Connect server 1](<images/Connecting to MCS-HPP-EFS-Server1 via EC2 instance connect.png>)
<div align="center"><b>Figure: Connecting to EFS Server 1 via Instance Connect</b></div>

<br><br>

![Connect server 2](<images/Connecting to MCS_HPP-EFS-Server2 via EC2 instance connect.png>)
<div align="center"><b>Figure: Connecting to EFS Server 2 via Instance Connect</b></div>

<br><br>

#### Terminal Mount & Real-Time File Sync Test
![Terminal session 1](<images/Terminal session on MCS-HPP-EFS-Server1 installing EFS.png>)
<div align="center"><b>Figure: Installing EFS Utils on Server 1 Terminal</b></div>

<br><br>

![Terminal session 2](<images/Terminal session on MCS-HPP-EFS-Server2 installing EFS.png>)
<div align="center"><b>Figure: Installing EFS Utils on Server 2 Terminal</b></div>

<br><br>

![Copy EFS mount helper](<images/Copy EFS mount helper.png>)
<div align="center"><b>Figure: Copying EFS Mount Command Helper</b></div>

<br><br>

![Terminal mount 1](<images/Terminal session on MCS-HPP-EFS-Server1 mounting the EFS file system, creating test file hpp.txt, and verifying storage.jpg>)
<div align="center"><b>Figure: Mounting EFS & Creating Test File (`hpp.txt`) on Server 1</b></div>

<br><br>

![Terminal mount 2](<images/Terminal session on MCS-HPP-EFS-Server2 mounting the EFS file system and listing directory contents to confirm shared file synchronization (hpp.txt).jpg>)
<div align="center"><b>Figure: Mounting EFS & Confirming Real-Time File Sync on Server 2</b></div>

</details>

---

### 5️⃣ Amazon S3 Asset Storage
- **Object Bucket (`hpp-s3-bucket-gallery`):** Deployed in `ap-southeast-1` with bucket versioning enabled.
- **Public Asset Policy:** Applied a custom JSON bucket policy permitting `s3:GetObject` read permissions to securely serve static media assets (`YOUTH-International-College.png`) over public web URLs.

<details>
<summary><b>📷 View Storage Service (S3) Configuration</b></summary>

#### S3 Bucket Creation
![Creating S3 Bucket](<images/Creating S3 Bucket (hpp-s3-bucket-gallery).png>)
<div align="center"><b>Figure: Creating S3 Object Bucket</b></div>

<br><br>

![Configuring block public access](<images/Configuring block public access settings for the bucket.png>)
<div align="center"><b>Figure: Configuring Block Public Access Settings</b></div>

<br><br>

![Bucket Versioning](<images/Bucket Versioning for object recovery.png>)
<div align="center"><b>Figure: Enabling S3 Bucket Versioning</b></div>

<br><br>

![Uploading asset file](<images/Uploading asset file (YOUTH-International-College.png) to the S3 bucket.png>)
<div align="center"><b>Figure: Uploading Asset File (`YOUTH-International-College.png`)</b></div>

<br><br>

![Upload status summary](<images/Upload status summary confirming successful file transmission.png>)
<div align="center"><b>Figure: Upload Status Summary Confirmation</b></div>

<br><br>

![Edit Bucket Policy](<images/Edit Bucket Policy with a JSON statement allowing public access.png>)
<div align="center"><b>Figure: Editing Bucket Policy with JSON Statement</b></div>

<br><br>

![Object Overview dashboard](<images/Object Overview dashboard showing details and copying the Object URL.png>)
<div align="center"><b>Figure: S3 Object Overview & Public URL Copy</b></div>

<br><br>

![Browser validation S3](<images/Browser validation displaying the publicly hosted image via the S3 URL.png>)
<div align="center"><b>Figure: Browser Validation of Publicly Hosted S3 Asset</b></div>

</details>

---

### 6️⃣ Relational Database Service (RDS MySQL) & Zero-Open-Port SSM Access
- **Private RDS MySQL Instance (`mcs-hpp-aws-rds`):** MySQL 8.4 engine deployed in private subnet group `mcs-db-subnet-group`. Restricted inbound traffic on MySQL port 3306 strictly to application instances in `MCS-EC2-Secgp`.
- **Zero-Open-Port Management (AWS SSM):** Attached IAM role `MCS-SSM-EC2-Role` (`AmazonSSMManagedInstanceCore` policy) to private instance `MCS-HPP-SQLdatabase-client`. Used **AWS Systems Manager Session Manager** to access terminal sessions and manage MySQL databases (`HPPdb`) without opening inbound SSH ports or assigning public IP addresses.

<details>
<summary><b>📷 View RDS Deployment, SSM Role & MySQL Query Logs</b></summary>

#### Security Groups & DB Client Launch
![Creating security group MCS-EC2-Secgp](<images/Creating security group (MCS-EC2-Secgp).png>)
<div align="center"><b>Figure: Creating `MCS-EC2-Secgp` Security Group</b></div>

<br><br>

![Security Group Details 1](<images/Security Group Details console confirming the creation of MCS-EC2-Secgp.png>)
<div align="center"><b>Figure: Security Group Details Confirmation (`MCS-EC2-Secgp`)</b></div>

<br><br>

![Creating security group MCS-RDS-Secgp](<images/Creating security group (MCS-RDS-Secgp).png>)
<div align="center"><b>Figure: Creating `MCS-RDS-Secgp` Security Group</b></div>

<br><br>

![Security Group Details 2](<images/Security Group Details console confirming the successful creation of MCS-RDS-Secgp.png>)
<div align="center"><b>Figure: Security Group Details Confirmation (`MCS-RDS-Secgp`)</b></div>

<br><br>

![Launch database client](<images/Launch database client instance (MCS-HPP-SQLdatabase-client).png>)
<div align="center"><b>Figure: Launching Database Client Instance</b></div>

<br><br>

![Instance type DB client](<images/Instance type and key pair selection for database client.png>)
<div align="center"><b>Figure: Database Client Instance Type & Key Pair</b></div>

<br><br>

![Network settings DB client](<images/Network settings of DB client.png>)
<div align="center"><b>Figure: Database Client Network Settings</b></div>

<br><br>

![Storage config DB client](<images/Storage configuration of DB client.png>)
<div align="center"><b>Figure: Database Client Storage Configuration</b></div>

<br><br>

![Created DB client list](<images/Created DB client list.png>)
<div align="center"><b>Figure: Created Database Client Instance List</b></div>

<br><br>

#### RDS Creation & Subnet Group Isolation
![Create database full config](<images/Create database with full configuration.png>)
<div align="center"><b>Figure: Creating Database with Full Configuration</b></div>

<br><br>

![Selecting MySQL engine](<images/Selecting MySQL as the database engine type.png>)
<div align="center"><b>Figure: Selecting MySQL Engine Type</b></div>

<br><br>

![Choosing Full configuration](<images/Choosing Full configuration mode and selecting the Free tier template.png>)
<div align="center"><b>Figure: Selecting Free Tier Template</b></div>

<br><br>

![Single-AZ deployment](<images/Selecting Single-AZ DB instance deployment for availability and durability.png>)
<div align="center"><b>Figure: Single-AZ DB Instance Deployment</b></div>

<br><br>

![MySQL version](<images/MySQL Community edition and selecting engine version.png>)
<div align="center"><b>Figure: Selecting MySQL Community Edition Version</b></div>

<br><br>

![DB instance identifier](<images/DB instance identifier and credentials settings.png>)
<div align="center"><b>Figure: DB Instance Identifier & Credentials</b></div>

<br><br>

![Password authentication](<images/Password authentication and selecting Burstable classes under instance configuration.png>)
<div align="center"><b>Figure: Instance Class & Authentication Settings</b></div>

<br><br>

![Storage Settings RDS](<images/Storage Settings.png>)
<div align="center"><b>Figure: RDS Storage Settings</b></div>

<br><br>

![Setting up connectivity](<images/Setting up connectivity.png>)
<div align="center"><b>Figure: Setting Up RDS Connectivity</b></div>

<br><br>

![Create DB subnet group](<images/Create DB subnet group.png>)
<div align="center"><b>Figure: Creating Custom DB Subnet Group</b></div>

<br><br>

![Selecting AZ subnets](<images/Selecting availability zones with associate subnets.png>)
<div align="center"><b>Figure: Selecting AZ Subnets for DB Group</b></div>

<br><br>

![RDS subnet groups list](<images/RDS subnet groups list.png>)
<div align="center"><b>Figure: RDS Subnet Groups List</b></div>

<br><br>

![Configuring DB subnet group public access](<images/Configuring DB subnet group with public access to No and associate with security group.png>)
<div align="center"><b>Figure: Configuring Private DB Subnet Group Access</b></div>

<br><br>

![Setting AZ](<images/Setting the availability zone.png>)
<div align="center"><b>Figure: Setting Availability Zone Configuration</b></div>

<br><br>

![Tags](<images/Tags.png>)
<div align="center"><b>Figure: Resource Tagging Configuration</b></div>

<br><br>

![Additional monitoring](<images/Additional monitoring settings.png>)
<div align="center"><b>Figure: Additional Monitoring Settings</b></div>

<br><br>

![Database Options](<images/Database Options.png>)
<div align="center"><b>Figure: Database Options Configuration</b></div>

<br><br>

![Maintenance settings](<images/Maintenance settings.png>)
<div align="center"><b>Figure: Maintenance Window Settings</b></div>

<br><br>

![RDS Databases console](<images/RDS Databases console confirming the successful creation.png>)
<div align="center"><b>Figure: RDS Databases Console Confirmation</b></div>

<br><br>

![Connection details wizard](<images/Connection details wizard.png>)
<div align="center"><b>Figure: RDS Connection Details Wizard</b></div>

<br><br>

#### IAM Role Setup & SSM Session Manager Access
![IAM Roles console wizard](<images/IAM Roles console wizard.png>)
<div align="center"><b>Figure: IAM Roles Console Wizard</b></div>

<br><br>

![Selecting EC2 use case](<images/Selecting EC2 as the service use case for the IAM role.png>)
<div align="center"><b>Figure: Selecting EC2 Service Use Case</b></div>

<br><br>

![Add permissions policies](<images/Add permissions policies.png>)
<div align="center"><b>Figure: Adding IAM Permission Policies</b></div>

<br><br>

![Entering role name](<images/Entering the role name (MCS-SSM-EC2-Role) and review details.png>)
<div align="center"><b>Figure: Naming IAM Role (`MCS-SSM-EC2-Role`)</b></div>

<br><br>

![IAM Roles dashboard](<images/IAM Roles dashboard confirming the successful creation.png>)
<div align="center"><b>Figure: IAM Roles Dashboard Confirmation</b></div>

<br><br>

![Modify IAM role](<images/Modify IAM role.png>)
<div align="center"><b>Figure: Modifying Instance IAM Role Assignment</b></div>

<br><br>

![Connect to Linux instance console](<images/Connect to Linux instance console with SSM Session Manager.png>)
<div align="center"><b>Figure: Connecting via AWS SSM Session Manager</b></div>

<br><br>

#### MySQL Terminal Execution & Database Creation
![Updating package list](<images/Updating the package list and installing mysql-client-core-8.0 in terminal.png>)
<div align="center"><b>Figure: Installing MySQL Client in Terminal</b></div>

<br><br>

![Executing MySQL connection command](<images/Executing the MySQL client connection command with RDS Endpoint information.png>)
<div align="center"><b>Figure: Executing MySQL Connection with RDS Endpoint</b></div>

<br><br>

![SQL queries view and create](<images/SQL queries to view and create new database.png>)
<div align="center"><b>Figure: Executing SQL Queries to Create Database (`HPPdb`)</b></div>

<br><br>

![Verification created database](<images/Verification of the created new database.png>)
<div align="center"><b>Figure: Verification of Newly Created Database</b></div>

</details>

---

### 7️⃣ CloudWatch Operational Dashboards, Alarms & Automated Remediation
- **Custom Dashboard (`For-MCS`):** Configured real-time CPU utilization telemetry widgets for private test node `MCS-HPP-CloudWatch-Test`.
- **High-Stress Alarm (`MCS-alarm`):** Set a static threshold rule triggering when average CPU utilization exceeds 40% over 1 minute.
- **Automated Remediation & SNS Alerting:** Integrated AWS Simple Notification Service (`MCS_CloudWatch_Alarms_Topic`) for automated email alerts. Attached an automated EC2 action to force server termination whenever CPU load spikes under stress testing (`stress --cpu 30 --timeout 600`).

<details>
<summary><b>📷 View Telemetry, Stress Test & Auto-Termination Flow</b></summary>

#### CloudWatch Node & Metrics Widget Setup
![Launch CloudWatch instance](<images/Launch CloudWatch instance.png>)
<div align="center"><b>Figure: Launching CloudWatch Test Instance</b></div>

<br><br>

![Instance type key pair CW](<images/Instance type and key pair of CloudWatch.png>)
<div align="center"><b>Figure: CloudWatch Instance Type & Key Pair</b></div>

<br><br>

![Network settings CW](<images/Network settings of CloudWatch.png>)
<div align="center"><b>Figure: CloudWatch Network Settings</b></div>

<br><br>

![Configure storage CW](<images/Configure storage for CloudWatch.png>)
<div align="center"><b>Figure: CloudWatch Storage Configuration</b></div>

<br><br>

![Created CloudWatch Instance](<images/Created CloudWatch Instance.png>)
<div align="center"><b>Figure: Created CloudWatch Test Instance</b></div>

<br><br>

![Adding CloudWatchAgentServerPolicy](<images/Adding (CloudWatchAgentServerPolicy) to the IAM permission policies of MCS-SSM-EC2-Role.png>)
<div align="center"><b>Figure: Adding CloudWatch Agent Policy to IAM Role</b></div>

<br><br>

![CloudWatch instance connect SSM](<images/MCS-HPP-CloudWatch-Test instance connect with SSM Session Manager.png>)
<div align="center"><b>Figure: Connecting to CloudWatch Test Node via SSM</b></div>

<br><br>

![Installing htop](<images/Installing htop in terminal.png>)
<div align="center"><b>Figure: Installing `htop` Utility in Terminal</b></div>

<br><br>

![Successful htop installation](<images/Successful installation of htop-3.2.1.png>)
<div align="center"><b>Figure: Successful `htop` Installation Confirmation</b></div>

<br><br>

![htop process viewer](<images/htop process viewer displaying real-time CPU, memory, task usage, and running system processes.png>)
<div align="center"><b>Figure: `htop` Process Viewer Displaying System Metrics</b></div>

<br><br>

![Create dashboard CloudWatch](<images/Create dashboard in CloudWatch.png>)
<div align="center"><b>Figure: Creating CloudWatch Custom Dashboard</b></div>

<br><br>

![Add widget configuration](<images/Add widget configuration.png>)
<div align="center"><b>Figure: Adding Dashboard Widget Configuration</b></div>

<br><br>

![Add metric graph EC2](<images/Add metric graph with EC2.png>)
<div align="center"><b>Figure: Adding EC2 Metric Graph</b></div>

<br><br>

![Selecting Per-Instance Metrics](<images/Selecting Per-Instance Metrics from EC2.png>)
<div align="center"><b>Figure: Selecting EC2 Per-Instance Metrics</b></div>

<br><br>

![Selecting CPU Utilization](<images/Selecting CPU Utilization.png>)
<div align="center"><b>Figure: Selecting CPU Utilization Metric</b></div>

<br><br>

![Specify metric conditions](<images/Specify metric and conditions.png>)
<div align="center"><b>Figure: Specifying Metric Conditions & Thresholds</b></div>

<br><br>

![Picking EC2 metric](<images/Picking EC2 metric.png>)
<div align="center"><b>Figure: Picking EC2 Target Metric</b></div>

<br><br>

![Per-Instance Metrics](<images/Per-Instance Metrics from EC2.png>)
<div align="center"><b>Figure: Per-Instance Metrics Panel Overview</b></div>

<br><br>

![CPUUtilization metric](<images/CPUUtilization metric for instance (MCS-HPP-CloudWatch-Test).png>)
<div align="center"><b>Figure: CPU Utilization Metric for Test Instance</b></div>

<br><br>

#### Alarm Threshold & SNS Topic Configuration
![Setting metric conditions static threshold](<images/Setting metric conditions with a static threshold.png>)
<div align="center"><b>Figure: Setting Static Threshold Conditions for Alarm</b></div>

<br><br>

![Configuring SNS notification action](<images/Configuring SNS notification action by creating alarm topic.png>)
<div align="center"><b>Figure: Configuring SNS Notification Action Topic</b></div>

<br><br>

![Defining EC2 automated action](<images/Defining an EC2 automated action to terminate the instance when the alarm state is triggered.png>)
<div align="center"><b>Figure: Defining Automated EC2 Termination Action</b></div>

<br><br>

![Add alarm details](<images/Add alarm details.png>)
<div align="center"><b>Figure: Adding Alarm Name & Details</b></div>

<br><br>

![Created alarm OK state](<images/Created alarm showing OK state alongside a pending SNS subscription warning.png>)
<div align="center"><b>Figure: Created Alarm in OK State with Pending Subscription</b></div>

<br><br>

![Confirming SNS email subscription](<images/Confirming the AWS SNS email subscription.png>)
<div align="center"><b>Figure: Confirming AWS SNS Email Subscription</b></div>

<br><br>

![Verification of alarm Actions enabled](<images/Verification of alarm displays Actions enabled.png>)
<div align="center"><b>Figure: Alarm Verification Displaying Actions Enabled</b></div>

<br><br>

#### Terminal Stress Testing, CPU Spike & Automated EC2 Termination
![Installing stress](<images/Installing stress in terminal.png>)
<div align="center"><b>Figure: Installing Stress Test Utility in Terminal</b></div>

<br><br>

![Dual terminal stress hop](<images/Dual terminal stress hop.png>)
<div align="center"><b>Figure: Running CPU Stress Test & Observing htop Spike</b></div>

<br><br>

![AWS SNS email alert inbox](<images/AWS SNS email alert inbox message confirming state transition from OK to ALARM.png>)
<div align="center"><b>Figure: AWS SNS Email Alert Inbox Notification</b></div>

<br><br>

![CloudWatch Alarms dashboard](<images/CloudWatch Alarms dashboard detailed view displaying MCS-alarm in In-alarm state.png>)
<div align="center"><b>Figure: CloudWatch Alarm Dashboard in In-Alarm State</b></div>

<br><br>

![Broadcast messages stating](<images/Broadcast messages stating in terminal.png>)
<div align="center"><b>Figure: Broadcast Messages Stating Termination in Terminal</b></div>

<br><br>

![Verification of instance Terminated](<images/Verification of instance MCS-HPP-CloudWatch-Test transitioned into the Terminated state.png>)
<div align="center"><b>Figure: Verification of Instance Transitioned to Terminated State</b></div>

</details>

---

## 🛠️ Key Technical Skills Demonstrated
- **Cloud Infrastructure:** AWS VPC, Internet Gateway, NAT Gateway, Elastic IP, Route Tables
- **Compute & Load Balancing:** EC2 Instance Provisioning, Security Groups, Application Load Balancers, Target Groups
- **Storage Systems:** Amazon EFS (NFS multi-mount), Amazon S3 (Bucket Policies & Public Access Controls)
- **Database & Security Management:** Amazon RDS MySQL, Private Subnet Group Isolation, AWS SSM Session Manager, IAM Roles & Policies
- **Monitoring & Operations:** CloudWatch Dashboards, Custom Metric Alarms, AWS SNS, Automated EC2 Actions