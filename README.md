# AutoScalingMultiAZWithNotifications CloudFormation Template :
 Create a multi-az, load balanced and Auto Scaled sample web site running on an Apache Web Serever. The application is configured to span all Availability Zones in the region and is Auto-Scaled based on the CPU utilization of the web servers. Notifications will be sent to the operator email address on scaling events.it also has NAT Instance with RDS instance to enable internet connection for RDS

# The template creates the below components
1 Public Subnet and route table respectively
2 Private Subnets and route tables respectively
2 Private subnet Web Servers in auto scaling group with Load Balancer in Public Subnet
1 bastion Host with EIP in Public Subnet
1 NAT Instance  to provide internet route for private subnet instances
1 RDS Postgres Instance in Private Subnet
1 IAM role with policy to put and get objects from S3Bucket through webserver
1 SNS topic to send notifications
1 Internet Gateway
1 VPC 

# Web Servers :
1.Webservers are install with apache during boot up process with user data option and also updates system libraries with yum update
2.The static website which is deployed on webservers will be accessed through public load balancer on port 80 only
3. Webserver Auto scaling Policies:
        Scale up the 1 server when CPU is more than 80%
        Scale down 1 server when CPU is less than 20%
        
# S3 Bucket:
Creating S3 bucket and Allowing EC2 instance to access objects

# Process to Access S3 bucket from Application on Ec2 Instance:
Create a role to have access to read and write access to S3 bucket and assign role to Ec2 instance
using the following IAM role policy to be able to list and write objects to S3 Bucket from Ec2 Isntance :

 "Type" : "AWS::IAM::Role",
			  "Properties" : {
				"AssumeRolePolicyDocument": {
				  "Version" : "2012-10-17",
				  "Statement" : [{ "Effect" : "Allow",
   				  "Principal" : { "Service" : ["ec2.amazonaws.com"] },
					  "Action" : ["sts:AssumeRole" ]}]
				},
				"Path" : "/",
				"Policies": [ {
				   "PolicyName": "s3accesspolicy",
				   "PolicyDocument": {
					  "Version" : "2012-10-17",
					  "Statement": [ {
						 "Effect": "Allow",
						 "Action": ["s3:GetObject", "s3:PutObject"],
						 "Resource": { "Fn::Join" : ["", ["arn:aws:s3:::", { "Ref" : "S3Bucket" } , "/*" ]]}} ]
				   }
				   } ]
			  }

# RDS Instance :
Creating one RDS Postgres instance to maintain Database and it will have access to internet through NAT instance 

# SNS Notification:
SNS topic notifies whenever a webserver launched, terminated and also allows operator input email id for subscription

# Bastion Host:
Bastion host with EIP,private subnet instances will be ssh accessed through this host



