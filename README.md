**Building a 3-tier web application architecture with AWS**

![3-tier web application architecture](https://github.com/surbhisamota10/Projects-New/blob/main/3-tier%20web%20application%20architecture.jfif

Here’s a simplified description:

Pre-requisites:
-> AWS account
-> Basic knowledge of AWS services like EC2, RDS, and VPC, Auto scaling groups,Load balancer, and security groups.
-> Familiarity with Linux commands, scripting, and SSH.
-> Familiarity with Linux commands, basic scripting, and SSH.
-> Access to a command line tool.

*Login in your aws account*

*Set Up Your VPC (Virtual Private Cloud)*
This base network consists of:

A VPC.
Two (2) public subnets spread across two availability zones (Web Tier).
Two (2) private subnets spread across two availability zones (Application Tier).
Two (2) private subnets spread across two availability zones (Database Tier).
One (1) public route table that connects the public subnets to an internet gateway.
One (1) private route table that will connect the Application Tier private subnets and a NAT gateway.

In the VPC console, let’s create a new VPC. We’ll select the 'VPC and more' option and name our project 'MyVPC_web_app' with a CIDR block of 10.0.0.0/16

![VPCcreation-image1](https://github.com/surbhisamota10/Projects-New/assets/95540023/604a4eac-eec6-4274-b264-8d774300e56a)


![VPCcreation-image2](https://github.com/surbhisamota10/Projects-New/assets/95540023/1f47a951-677a-453d-b552-0c161fb13db2)

*Enable auto-assign*:

Now you need to go inside subnet section ,
we need to make sure we 'Enable auto-assign public IPv4 address' for BOTH public subnets so we can access its resources via the Internet.

![subnetsetting1](https://github.com/surbhisamota10/Projects-New/assets/95540023/15f56251-cd44-4fae-8f02-395150dca710)

*Change route table*:

Now change the default route table to newly created route table with VPC.
![routetablechange](https://github.com/surbhisamota10/Projects-New/assets/95540023/e9421921-4d5e-403a-89b8-4197e47bc5f2)

*Create a NAT Gateway*:

A NAT gateway allows instances from the private subnets to connect to resources outside of the VPC and the Internet (for necessary services such as patches or package updates).

Navigate to ‘NAT Gateways’ and create a new gateway called public-NAT-1. Select one of the public subnets, allocate an elastic IP, and create the gateway.
![NATgateway](https://github.com/surbhisamota10/Projects-New/assets/95540023/701d4953-8cf7-4aaf-ba8e-8d7862d061eb)

*Configure private route tables*:

As we can see, a route table has been created for each private subnet (4) by default. However, we only need one private route table (for the Application Tier subnets). This is where clear naming conventions can immensely help guard against confusion, so let’s navigate to our route tables and clean things up a bit.

![editroutetable](https://github.com/surbhisamota10/Projects-New/assets/95540023/58ba7e33-50e1-40fe-9544-195abfc8d30e)
![editroutetable1](https://github.com/surbhisamota10/Projects-New/assets/95540023/3b3dd53c-987e-404a-ac49-afce1e2f8343)



**Tier 1: Web tier (Frontend)**
What we’ll build:

-> A web server launch template to define what kind of EC2 instances will be provisioned for the application.
-> An Auto Scaling Group (ASG) that will dynamically provision EC2 instances.
-> An Application Load Balancer (ALB) to help route incoming traffic to the proper targets.

1. Create a web server launch template








