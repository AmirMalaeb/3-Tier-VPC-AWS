# 3-Tier-VPC-AWS


Introduction

This README file will guide you through building a 3-tier VPC architecture on AWS. This tutorial is designed for cloud enthusiasts and DevOps practitioners who want to understand the basics of networking on AWS. We’ll be using the AWS Command Line Interface (CLI) to create resources and automate the process of setting up a scalable VPC.

By the end of this tutorial, you will have:

	•	A Virtual Private Cloud (VPC)
	•	Three subnets (public, private, and database)
	•	Proper routing between your subnets
	•	Internet access for public resources
	•	Secure access for private and database resources

Prerequisites

Before you begin, ensure that you have the following installed and configured:

	1.	Install the AWS CLI:
The AWS CLI is a unified tool to manage your AWS services. You can install it by following this guide.
	2.	Configure the AWS CLI:
After installing, you’ll need to configure your CLI with your credentials and default region:
```bash
aws configure
```