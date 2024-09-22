# Building a 3-Tier VPC in AWS

This guide will walk you through the process of creating a 3-tier Virtual Private Cloud (VPC) in Amazon Web Services (AWS). We'll cover both the AWS Management Console method and provide AWS CLI commands for those who prefer a command-line approach.

## Table of Contents
1. [Overview of a 3-Tier VPC](#overview)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Guide](#guide)
   - [Step 1: Create the VPC](#step1)
     - [Understanding CIDR Blocks](#cidr-blocks)
   - [Step 2: Enable DNS Hostnames](#step2)
   - [Step 3: Create Internet Gateway](#step3)
   - [Step 4: Create Subnets](#step4)
   - [Step 5: Enable Auto-assign Public IP for Public Subnets](#step5)
   - [Step 6: Create NAT Gateways](#step6)
   - [Step 7: Create Route Tables](#step7)
   - [Step 8: Create Security Groups](#step8)
4. [Conclusion](#conclusion)


## Overview of a 3-Tier VPC <a name="overview"></a>

A 3-tier VPC typically consists of:
- Public tier (Web layer)
- Private tier (Application layer)
- Data tier (Database layer)

This architecture provides enhanced security and isolation for your applications.

## Prerequisites <a name="prerequisites"></a>

Before you begin, ensure you have the following:

1. An AWS account
2. AWS Management Console access
3. (Optional) AWS CLI installed and configured
   - To install the AWS CLI, follow the [official AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   - After installation, configure the AWS CLI with your credentials:
     ```
     aws configure
     ```
   - You'll need to provide your AWS Access Key ID, Secret Access Key, default region name, and output format
4. Basic understanding of AWS networking concepts
5. Sufficient permissions to create and manage VPC resources

Note: If you're using the AWS CLI, make sure your configured credentials have the necessary permissions to create and manage VPC resources.

[Remainder of the content remains unchanged]


# Building a 3-Tier VPC in AWS

### Step 1: Create the VPC <a name="step1"></a>

#### Understanding CIDR Blocks

Before creating the VPC, it's important to understand CIDR (Classless Inter-Domain Routing) blocks:

- A CIDR block represents a range of IP addresses.
- The format is typically `x.x.x.x/y`, where `x.x.x.x` is the network address and `y` is the prefix length (also known as the subnet mask).
- The prefix length determines the number of available IP addresses.

For our VPC, we'll use `10.0.0.0/16`:
- This CIDR block provides 65,536 available IP addresses (2^(32-16) = 2^16 = 65,536).
- It includes all IP addresses from 10.0.0.0 to 10.0.255.255.

To calculate the number of available IP addresses for any CIDR block:
1. Subtract the prefix length from 32 (for IPv4).
2. Raise 2 to the power of this result.

For example:
- `/24` subnet: 2^(32-24) = 2^8 = 256 IP addresses
- `/28` subnet: 2^(32-28) = 2^4 = 16 IP addresses

#### Console Instructions:
1. Navigate to the VPC dashboard in the AWS Management Console
2. Click "Create VPC"
3. Choose "VPC and more"
4. Configure your VPC settings:
   - Name tag: My3TierVPC
   - IPv4 CIDR block: 10.0.0.0/16
5. Leave other settings as default and click "Create VPC"

#### CLI Command:
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=My3TierVPC}]'
```

This command creates a VPC with the CIDR block 10.0.0.0/16 and tags it with the name "My3TierVPC".

Note: When planning your VPC, ensure you choose a CIDR block that doesn't overlap with your other networks and provides enough IP addresses for your current and future needs.


### Step 2: Enable DNS Hostnames <a name="step2"></a>

Enabling DNS hostnames is an important step in setting up your VPC. Here's why:

1. **Automatic DNS naming**: When enabled, EC2 instances in your VPC automatically receive DNS hostnames that correspond to their public IP addresses.

2. **Internal DNS resolution**: It allows instances within your VPC to resolve the DNS hostnames of other instances to their private IP addresses.

3. **Integration with other AWS services**: Many AWS services rely on DNS resolution. Enabling this feature ensures smoother integration with services like Amazon RDS, ElastiCache, and ELB.

4. **Simplified network management**: It makes it easier to reference instances by name rather than IP address, which can simplify network management and troubleshooting.

5. **Support for custom domain names**: If you plan to use custom domain names within your VPC, this feature is essential.

#### Console Instructions:
1. Select your VPC in the VPC dashboard
2. Click "Actions" and choose "Edit VPC settings"
3. Check the box for "Enable DNS hostnames"
4. Click "Save changes"

#### CLI Command:
```bash
aws ec2 modify-vpc-attribute --vpc-id <vpc-id> --enable-dns-hostnames "{\"Value\":true}"
```

This command enables DNS hostnames for your VPC.


### Step 3: Create Internet Gateway <a name="step3"></a>

#### Console Instructions:
1. In the VPC dashboard, navigate to "Internet Gateways"
2. Click "Create internet gateway"
3. Name your internet gateway and create it
4. Select the newly created internet gateway
5. Click "Actions" and choose "Attach to VPC"
6. Select your VPC and click "Attach"

#### CLI Commands:
```bash
# Create Internet Gateway
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=My3TierVPC-IGW}]'

# Attach Internet Gateway to VPC
aws ec2 attach-internet-gateway --vpc-id <vpc-id> --internet-gateway-id <igw-id>
```
These commands create an Internet Gateway and attach it to your VPC.


### Step 4: Create Subnets <a name="step4"></a>

In this step, we'll create six subnets: two public subnets, two private (application) subnets, and two data subnets, spread across two Availability Zones for high availability.

#### Console Instructions:
1. In the VPC dashboard, navigate to "Subnets"
2. Click "Create subnet"
3. Select your VPC
4. Create the following subnets:
   - Public Subnet 1 (AZ1)
   - Public Subnet 2 (AZ2)
   - Private Subnet 1 (AZ1)
   - Private Subnet 2 (AZ2)
   - Data Subnet 1 (AZ1)
   - Data Subnet 2 (AZ2)
5. For each subnet, specify a unique CIDR block within your VPC CIDR range

#### CLI Commands:
```bash
# Create public subnets
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet1}]'
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.2.0/24 --availability-zone us-east-1b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet2}]'

# Create private (application) subnets
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.3.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet1}]'
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.4.0/24 --availability-zone us-east-1b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet2}]'

# Create data subnets
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.5.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=DataSubnet1}]'
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.6.0/24 --availability-zone us-east-1b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=DataSubnet2}]'
```

These commands create six subnets: two for each tier (public, private, and data) across two different Availability Zones. Adjust the CIDR blocks and Availability Zones as needed for your specific requirements.

Note: Ensure that your chosen CIDR blocks fit within your VPC's CIDR range and do not overlap.


### Step 5: Enable Auto-assign Public IP for Public Subnets <a name="step5"></a>

Enabling auto-assign public IP for your public subnets is crucial for instances in these subnets to be accessible from the internet. Here's why this step is important:

1. **Internet Accessibility**: Instances launched in these subnets will automatically receive a public IP address, allowing them to be reached from the internet.
2. **Outbound Internet Access**: It enables instances to initiate outbound connections to the internet without the need for a NAT gateway.
3. **Simplified Configuration**: You don't need to manually assign Elastic IP addresses to instances that need to be publicly accessible.

#### Console Instructions:
1. In the VPC dashboard, navigate to "Subnets"
2. Select one of your public subnets
3. Click "Actions" and choose "Edit subnet settings"
4. Check the box for "Enable auto-assign public IPv4 address"
5. Click "Save"
6. Repeat for the other public subnet

#### CLI Commands:
```bash
# Enable auto-assign public IP for PublicSubnet1
aws ec2 modify-subnet-attribute --subnet-id <public-subnet-1-id> --map-public-ip-on-launch

# Enable auto-assign public IP for PublicSubnet2
aws ec2 modify-subnet-attribute --subnet-id <public-subnet-2-id> --map-public-ip-on-launch
```

These commands enable the auto-assign public IP feature for both of your public subnets.


### Step 6: Create and Configure Public Route Table <a name="step6"></a>

Creating and configuring a route table for your public subnets is a crucial step in setting up your VPC. This route table will enable internet access for resources in your public subnets.

#### Why this step is important:
1. **Internet Access**: It allows resources in public subnets to access the internet.
2. **Inbound Traffic**: It enables incoming traffic from the internet to reach resources in public subnets.
3. **Subnet Association**: It defines which subnets are public by associating them with this route table.

#### Console Instructions:
1. In the VPC dashboard, navigate to "Route Tables"
2. Click "Create route table"
3. Name it (e.g., "Public Route Table") and select your VPC
4. Click "Create"
5. Select the newly created route table
6. In the "Routes" tab, click "Edit routes"
7. Add a new route:
   - Destination: 0.0.0.0/0
   - Target: Select your Internet Gateway
8. Click "Save routes"
9. In the "Subnet associations" tab, click "Edit subnet associations"
10. Select your public subnets and click "Save associations"

#### CLI Commands:
```bash
# Create the route table
aws ec2 create-route-table --vpc-id <vpc-id> --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public Route Table}]'

# Add route to Internet Gateway
aws ec2 create-route --route-table-id <route-table-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <internet-gateway-id>

# Associate public subnets with the route table
aws ec2 associate-route-table --route-table-id <route-table-id> --subnet-id <public-subnet-1-id>
aws ec2 associate-route-table --route-table-id <route-table-id> --subnet-id <public-subnet-2-id>
```

These commands create a new route table, add a route to the Internet Gateway, and associate it with your public subnets.
