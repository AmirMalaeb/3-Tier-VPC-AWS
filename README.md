# Building a 3-Tier VPC in AWS

This guide will walk you through the process of creating a 3-tier Virtual Private Cloud (VPC) in Amazon Web Services (AWS). We'll cover both the AWS Management Console method and provide AWS CLI commands for those who prefer a command-line approach.

## Table of Contents
1. [Overview of a 3-Tier VPC](#overview)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Guide](#guide)
   - [Step 1: Create the VPC](#step1)
   - [Step 2: Create Subnets](#step2)
   - [Step 3: Create Internet Gateway](#step3)
   - [Step 4: Create NAT Gateways](#step4)
   - [Step 5: Create Route Tables](#step5)
   - [Step 6: Create Security Groups](#step6)
4. [Conclusion](#conclusion)

## Overview of a 3-Tier VPC <a name="overview"></a>

A 3-tier VPC typically consists of:
- Public tier (Web layer)
- Private tier (Application layer)
- Data tier (Database layer)

This architecture provides enhanced security and isolation for your applications.

## Prerequisites <a name="prerequisites"></a>

- An AWS account
- AWS Management Console access or AWS CLI installed and configured
- Basic understanding of AWS networking concepts

## Step-by-Step Guide <a name="guide"></a>

### Step 1: Create the VPC <a name="step1"></a>

#### Console Instructions:
1. Navigate to the VPC dashboard in the AWS Management Console
2. Click "Create VPC"
3. Choose "VPC and more"
4. Configure your VPC settings (e.g., name, IPv4 CIDR block)

#### CLI Command:
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=My3TierVPC}]'
```

This command creates a VPC with the CIDR block 10.0.0.0/16 and tags it with the name "My3TierVPC".

### Step 2: Create Subnets <a name="step2"></a>

#### Console Instructions:
1. In the VPC dashboard, navigate to "Subnets"
2. Click "Create subnet"
3. Select your VPC and create subnets for each tier in different Availability Zones

#### CLI Commands:
```bash
# Create public subnet in AZ1
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet1}]'

# Create private subnet in AZ1
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.2.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet1}]'

# Create data subnet in AZ1
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.3.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=DataSubnet1}]'

# Repeat for AZ2 with different CIDR blocks
```

These commands create subnets for each tier in two different Availability Zones.