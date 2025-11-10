## **Configuring a VPC**

## **Overview**

In this lab, I created a Virtual Private Cloud (VPC) from scratch with both public and private subnets, an Internet Gateway, a NAT Gateway, and route tables to manage traffic flow. I then launched a Bastion Server in the public subnet to securely access resources inside the private subnet.
This architecture allows secure, scalable communication between resources inside the private network and the internet through a NAT gateway, following AWS best practices.

## **Objectives & Learning Outcomes**

After completing this lab, I was able to:

Create a custom VPC with defined CIDR block and subnets

Configure Internet Gateway and NAT Gateway for external communication

Build and associate Route Tables for local and internet-bound routing

Launch a Bastion Server in the public subnet

Enable secure private instance access through the Bastion host

## **Architecture Diagram**
<img width="1536" height="600" alt="ff0c152d-7a81-4a77-8c65-4a8e88c8fe9c" src="https://github.com/user-attachments/assets/65ebfce9-04e9-4d23-9301-ada47440e69c" />

A VPC with a Public Subnet (hosting the Bastion Server and NAT Gateway) and a Private Subnet (for internal EC2 instances), both within the same Availability Zone. Route tables are configured for internal and internet traffic flow.

## **Commands & Steps**
```bash

# Step 1: Create the VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications \
'ResourceType=vpc,Tags=[{Key=Name,Value=Lab-VPC}]'

# Enable DNS hostnames
aws ec2 modify-vpc-attribute --vpc-id <VPC_ID> --enable-dns-hostnames "{\"Value\":true}"

# Step 2: Create Public Subnet
aws ec2 create-subnet \
--vpc-id <VPC_ID> \
--cidr-block 10.0.0.0/24 \
--availability-zone us-west-2a \
--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-Subnet}]'

# Step 3: Create Private Subnet
aws ec2 create-subnet \
--vpc-id <VPC_ID> \
--cidr-block 10.0.2.0/23 \
--availability-zone us-west-2a \
--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-Subnet}]'

# Step 4: Create Internet Gateway and Attach to VPC
aws ec2 create-internet-gateway --tag-specifications \
'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Lab-IGW}]'

aws ec2 attach-internet-gateway \
--internet-gateway-id <IGW_ID> \
--vpc-id <VPC_ID>

# Step 5: Create Public Route Table and Add Route to IGW
aws ec2 create-route-table \
--vpc-id <VPC_ID> \
--tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-Route-Table}]'

aws ec2 create-route \
--route-table-id <Public_RouteTable_ID> \
--destination-cidr-block 0.0.0.0/0 \
--gateway-id <IGW_ID>

# Step 6: Associate Public Subnet with Public Route Table
aws ec2 associate-route-table \
--route-table-id <Public_RouteTable_ID> \
--subnet-id <Public_Subnet_ID>

# Step 7: Create Private Route Table
aws ec2 create-route-table \
--vpc-id <VPC_ID> \
--tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Private-Route-Table}]'

# Step 8: Create and Configure NAT Gateway
aws ec2 allocate-address --domain vpc
aws ec2 create-nat-gateway \
--subnet-id <Public_Subnet_ID> \
--allocation-id <Elastic_IP_Alloc_ID> \
--tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=Lab-NAT-Gateway}]'

# Add route in Private Route Table to NAT Gateway
aws ec2 create-route \
--route-table-id <Private_RouteTable_ID> \
--destination-cidr-block 0.0.0.0/0 \
--nat-gateway-id <NAT_ID>

# Step 9: Launch Bastion Server in Public Subnet
aws ec2 run-instances \
--image-id ami-0abcdef1234567890 \
--count 1 \
--instance-type t3.micro \
--subnet-id <Public_Subnet_ID> \
--associate-public-ip-address \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Bastion-Server}]' \
--security-group-ids <SG_ID>

```

## **Screenshots**

vpc-created.png → Successfully created Lab VPC (10.0.0.0/16)
<img width="1631" height="400" alt="VPC resouce Map" src="https://github.com/user-attachments/assets/4f51971c-6ef2-4998-bbf3-242a9e97d6bc" />

public-private-subnets.png → Public and private subnets with CIDR and AZ
<img width="1551" height="300" alt="public-private subnet" src="https://github.com/user-attachments/assets/e2c092df-5651-48ba-b2e8-7a9a02c88a8f" />

route-table-association.png → Public route table showing IGW route and association
<img width="1679" height="500" alt="route table" src="https://github.com/user-attachments/assets/3fba42a7-43bf-4dee-a1c6-1970d773c8c4" />

nat-gateway-and-bastion.png → NAT gateway and Bastion instance running in public subnet
<img width="1641" height="500" alt="NAT gateway" src="https://github.com/user-attachments/assets/e4644ea2-2450-48a9-bb3e-cdecb0aace12" />

## **Tools Used**

Amazon VPC

Amazon EC2

Internet Gateway (IGW)

NAT Gateway

AWS CLI

Route Tables & Subnet Associations

## **Key Takeaways**

A VPC provides full control over networking and security boundaries

Public subnet uses an Internet Gateway; Private subnet uses a NAT Gateway for outgoing access

Route Tables determine how traffic flows between networks

A Bastion Host acts as a secure jump point into private instances

## **What Actually Happened** 

Created a new VPC (10.0.0.0/16) with both public and private subnets

Configured an Internet Gateway and attached it to the VPC

Created Route Tables — public routes internet-bound via IGW, private routes via NAT

Launched a Bastion Server in the public subnet for SSH access

Deployed and tested secure access from Bastion to private subnet EC2 instance

## **Author**
Amarachi Emeziem
Cloud Engineer/Security, AWS Certified Cloud Proactitioner

LinkedIn profile: https://www.linkedin.com/in/amarachilemeziem/
