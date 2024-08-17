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
2. An Auto Scaling Group (ASG) that will dynamically provision EC2 instances.
3. An Application Load Balancer (ALB) to help route incoming traffic to the proper targets.

Now I create a EC2 instance :
AMI: Amazon 2 Linux,
Instance type: t2.micro (1GB – Free Tier),
KEY: A new or existing key pair

![ec2-1](https://github.com/surbhisamota10/Projects-New/assets/95540023/ccbb05f6-6bea-41e7-9978-577a93f22040)
![ec2-2](https://github.com/surbhisamota10/Projects-New/assets/95540023/de818778-fea2-4ad4-abf5-5ed0dfd4dbc4)

Now your server1 is ready and you can hit the public IP on browser
![ec2-3](https://github.com/surbhisamota10/Projects-New/assets/95540023/ec583652-73af-40f5-b34b-884f29bc85e7)
![ec2-4](https://github.com/surbhisamota10/Projects-New/assets/95540023/ee11b06f-e035-41f3-a83f-c45073f337d6)


2. Create an Auto scaling group (ASG):
To ensure high availability for the web app and limit single points of failure, we will create an ASG that will dynamically provision EC2 instances, as needed, across multiple AZs in our public subnets.

Navigate to the ASG console from the sidebar menu and create a new group. you can create a server1  launch template 
and do it stepwise.

![autos-1](https://github.com/surbhisamota10/Projects-New/assets/95540023/b48310c6-232b-4ee0-9a49-3966f351244a)
![autos-4](https://github.com/surbhisamota10/Projects-New/assets/95540023/fac126c3-d473-46ce-b19f-ed5bb34d8047)

Select the MyVPC_web_app-vpc along with the two public subnets.
![autos-5](https://github.com/surbhisamota10/Projects-New/assets/95540023/4a475dd7-17f0-4c3d-ac4c-04904050750c)

Now see one more instance create with using asg
![serverready](https://github.com/surbhisamota10/Projects-New/assets/95540023/63a3b7fc-2229-4ac9-aa45-b4214388f148)

Both instance are running 
![Screenshot 2024-07-03 121435](https://github.com/surbhisamota10/Projects-New/assets/95540023/01f2eba9-482a-43d2-9bc1-0452664fb90c)

3. Application load balancer (ALB)
We’ll need an ALB to distribute incoming HTTP traffic to the proper targets (our EC2s). The ALB will be named, 
‘My-alb.’ We want this ALB to be ‘Internet-facing,’ so it can listen for HTTP/S requests.

*Create the target group*:

First, we’re going to create a target group for our ALB to direct traffic to.

To create a target group, go to the ‘Target groups’ dashboard under the ‘Load balancing section’ in the EC2 console.

1)Our target type will be our EC2 instances, so make sure ‘Instances’ is selected and give your group a name.
2)We’re going to direct traffic to HTTP | port 80, so make sure the ‘Protocol’ is HTTP.
3)Select the same VPC we used for our ASG and EC2 instances.
4)On the next page, we should see the 2 running instances available to select, but we’ll leave it be for now. Create the group and we’re ready for the next step.

*Create the ALB*:
To create an ALB, go to the ‘Load balancing’ dashboard in the EC2 console and select Application Load Balancer.

1)Give your ALB a name and make sure the ‘Scheme’ is ‘Internet-facing.’
2)Under ‘Network mapping,’ select the proper VPC and three subnets.
3)Select the proper security group (you can add up to 5, but make sure only the one we created is attached).
4)For our ‘Listener,’ we are going to listen for HTTP requests and forward them to our target group, so select the target group we created. Create the ALB!


![LB1](https://github.com/surbhisamota10/Projects-New/assets/95540023/51ae5246-6067-4548-98cd-0b4c475ad3c7)
![Screenshot 2024-07-03 123314](https://github.com/surbhisamota10/Projects-New/assets/95540023/2a48b02d-411d-4806-a161-303854e737f5)
![LB3](https://github.com/surbhisamota10/Projects-New/assets/95540023/3ab1fa93-516b-40ab-a6a6-99de55423159)

*Connect the ALB to the ASG*:
Last step! Now that we've created our load balancer, we need to connect it to our ASG. So let’s go back to our ASG and scroll down to ‘Load balancing.’ Select the Application load balancer and the target group we created.
![Screenshot 2024-07-03 124421](https://github.com/surbhisamota10/Projects-New/assets/95540023/81427a07-3845-434e-824b-ed6cc304e1c3)

![Screenshot 2024-07-03 124342](https://github.com/surbhisamota10/Projects-New/assets/95540023/073529f6-0fde-44f6-8b6d-5b783f52aec4)

*Success!*
We’ve successfully created an auto scaling web app environment that dynamically scales our EC2 instances and properly routes traffic to the right target groups!

*Note: Once you’re done using your ASG and ALB, be sure to delete them or you’ll get charged for the running EC2 instances. The instances will automatically terminate.*












