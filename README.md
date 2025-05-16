# Creating Our Own Data Center on AWS

This project demonstrates how to build a secure, scalable, and isolated cloud-based data center using core AWS services such as VPC, EC2, Subnets, Route Tables, Internet Gateway, and NAT Gateway, Application Load Balancer.

-------------------------------------------------------------------------------

# VPC Setup

- Created a **Virtual Private Cloud (VPC)** to isolate our network from others.
- Assigned a CIDR block to define the IP range.
- This serves as the foundation for our AWS infrastructure.

![image](https://github.com/user-attachments/assets/d1726fa3-d961-4711-a069-d67629363f61)


-------------------------------------------------------------------------------

## Subnet Configuration

- Created two subnets within the VPC:
  - **publicSubnet** (`10.10.2.0/24`)
  - **privateSubnet** (`10.10.1.0/24`)
- **Public Subnet**: Allows direct internet access, ideal for web servers.
- **Private Subnet**: No direct internet access, suited for internal services like databases.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/12d4790c-6257-4af7-8bd3-58aeaad21c0e" />

-------------------------------------------------------------------------------

## Internet Gateway (IGW)

- Set up an **Internet Gateway** to enable internet communication.
- Attached the IGW to the VPC.
- Added a route in the **main route table** for internet access.
- Associated this routing table with the `publicSubnet`.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/b6c66bc6-d965-4dd5-a7a2-95e9360acffd" />

-------------------------------------------------------------------------------

## Route Tables

- **Main Route Table**:
  - Includes a route to the Internet Gateway.
  - Linked to the `publicSubnet`.
- **Custom Route Table**:
  - No internet gateway route.
  - Associated with the `privateSubnet` for internal traffic only.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/9e540190-6b4f-43d3-86f0-dcb9228aa354" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/9f5f3164-08f0-43c7-96e1-e4a5f82c0e31" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/781fa0d7-3cf9-4aa0-aa4f-1f1c912fa6a1" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/182629e0-2ab3-454f-a50f-3d6a1f642f42" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/26d2e37d-9e6d-4c93-94c6-c441f8fbddd7" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/3e420cf4-bda5-47c4-b096-f9b49292927b" />

------------------------------------------------------------------------------

## NAT Gateway Setup

- Created a **NAT Gateway** in the `publicSubnet` with an Elastic IP.
- Updated the **private route table** to route outgoing traffic through the NAT Gateway.
- This allows private instances to access the internet for updates, without accepting incoming connections.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/7b58f780-f63b-4790-bd06-f4f1611e781c" />

<img width="468" alt="image" src="https://github.com/user-attachments/assets/d7dfae84-8210-4657-aeb7-e00005f8e595" />


------------------------------------------------------------------------------

# Amazon EBS Volume Setup on EC2 (Persistent Storage Guide)

This guide explains the step-by-step process of attaching, mounting, and making an Amazon EBS volume persistent on an EC2 instance. This setup is ideal for database or persistent storage needs.

-------------------------------------------------------------------------------

## Steps:

STEP 1: Attached a 10 GB EBS volume to EC2 instance, and identify attached disks using fdisk -l
The 'fdisk -l' command lists all block storage devices. Here, it confirms that /dev/xvdb is present and ready for partitioning.

<img width="326" alt="image" src="https://github.com/user-attachments/assets/ddca7583-06a5-4fe8-9387-1e6e3bfdc165" />
ds -h : to check what disk is already mounted and in use.

Step 2: Partition the Disk!

We used 'fdisk /dev/xvdb' to create a new partition. This prepares the volume to hold a file system.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/638768a8-a12e-4a89-8f5c-89fe688b57b5" />

Step 3: format the partition
Using 'mkfs.xfs /dev/xvdb1', we formatted the partition with the XFS file system, which is efficient and widely supported.

Step 4: Create a mounted Directory 
The '/data' directory was created to serve as the mount point for the new volume.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/0cb6d45c-d2de-4eb3-bfe4-c74bf527b87c" />

Step 5: Mount the Volume
The partition was mounted to /data using 'mount /dev/xvdb1 /data'. The 'df -h' command confirmed that the mount was successful.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/bd3d23b6-b111-4f6c-b945-a6f3cc14c741" />

Step 6: Make the Mount Persistent
To automatically mount the volume after a reboot, we edited the '/etc/fstab' file and added the entry:
/dev/xvdb1   /data   xfs   defaults,noatime   1 1

<img width="468" alt="image" src="https://github.com/user-attachments/assets/be2f4080-4a62-4ad7-bf6b-d32612daed8d" />

Step 8:  Verify mount Persistence
After rebooting the instance, 'df -h' again showed /data mounted correctly, confirming that the configuration was successful.

<img width="468" alt="image" src="https://github.com/user-attachments/assets/106efede-3dce-4cd5-b0cf-ceba02e814e6" />

------------------------------------------------------------------------------

## EC2 Instances

- Launched two EC2 instances:
  - **publicEC2**: Placed in `publicSubnet`, accessible via SSH.
  - **privateEC2**: Placed in `privateSubnet`, not directly reachable from the internet.

------------------------------------------------------------------------------

# Load Balancer Configuration for High Availability

### Create a Load balancer
1.	Select all the availability zones in which our instances are present.
2.	Select same security group.  	

### Step 1: Create the Load Balancer
1.	Navigate to the EC2 dashboard and select Load Balancers.
2.	Click “Create Load Balancer” and choose Application Load Balancer.
3.	Assign a name and select the VPC where your instances reside.
4.	Choose all Availability Zones where your EC2 frontend instances are running to ensure redundancy.
5.	Configure a listener (typically HTTP or HTTPS).
6.	Attach a Target Group (either create new or use an existing one).

<img width="373" alt="image" src="https://github.com/user-attachments/assets/646bf085-7861-4187-8a51-81b91b93da4d" />

### Step 2: Configure the Target Group
1.	Create a Target Group and select Instances as the target type.
2.	Choose the protocol (e.g., HTTP) and port used by your application (e.g., 80).
3.	Select the EC2 instances that serve the frontend of your application.
4.	Register them as targets.

<img width="361" alt="image" src="https://github.com/user-attachments/assets/1fbdaa43-44ef-4ab5-ae65-c38550d52fe5" />


------------------------------------------------------------------------------

# Final Outcome

Successfully built a mini data center on AWS with:
- Isolated networking using public and private subnets.
- Secure access patterns via NAT Gateway and bastion host.
- Realistic cloud architecture mimicking production-grade setups.

------------------------------------------------------------------------------

# Contributors

- **Ritikesh Nikam**
- **Ashutosh Dikshit**
- **Vedika Chavan**
