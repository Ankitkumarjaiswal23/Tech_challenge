**3-tier Environment Set-Up in AWS Cloud Using IaC tool (Terraform).**

-> **Deploying a 3-Tier WordPress Architecture on AWS using Terraform.**                 

 **There is basic Pre-requisites required before going to perform this project.** 
 
These are like which I have understand and gone through are given below-  

**-AWS Knowledge**  

    Virtual Private Cloud (VPC) 
    
    Subnets, NACLs, Security groups, NAT gateways, Internet gateways 
    
    Elastic compute cloud EC2 
    
    Auto scaling 
    
    Application Load balancer 
    
    Relational database service 
    
    Install and configure AWS CLI locally. 
    
**-Terraform knowledge** 

    Infrastructure as Code (IaC) Hashicorp Configuration Language (HCL) 
    
    Understanding of Terraform moduless 
    
    Install and configure Terraform locally. 
    

    
**First of all we should know about the 3 tier architecture- ** 

The 3-tier architecture is a common software architectural design that comprises three layers, each with its obligations:

**Presentation layer:** This layer is responsible for presenting information to the end user, receiving data from the user and dealing with client interactions.

**Application layer:** This layer processes information/data and performs any business rationale/logic.

**Data layer:** This layer stores and manages data

If done properly, this architecture provides an adaptable, versatile and manageable mode of deploying applications, taking factors such as high availability, scalability, security and cost-effectiveness into consideration.

**Terraform**                  

We’ll be utilizing Infrastructure as Code (IaC) to build this project. IaC enables us to create a blueprint for our environment; allowing the provision and configuration of environments in a fast, dependable and repeatable way.

This project provides you with a complete setup for a scalable three-tier WordPress application on AWS, using Terraform configuration files to create a custom virtual private cloud (VPC), the necessary compute resources and a database backend. All these will be separated into 3 tiers:

**Tier 1:** Public layer with public access — Application load balancer and bastion/jump host

**Tier 2:** Private layer with restricted access – Multiple webservers hosting the WordPress application

**Tier 3:** Private layer with restricted access – Databases (primary & backup)

**Project Overview: Understanding Terraform Configuration files**

I have separated the Terraform code into a series of configuration files to provide better organization, modularity, and reusability. By doing so, it becomes easier to manage each component independently, making it simpler to understand & maintain. Additionally, splitting files allows you to reuse certain components in other projects, leading to more efficient and consistent infrastructure management. Let’s take a closer look at each of these files and their respective functions.

**s3-bucket-state:** This is a sub-directory in our repository. It contains provider.tf and bucket.tf files that create an S3 bucket that will remotely and securely store our terraform state files. Run this first before creating the actual infrastructure with the files below.

# Create private S3 bucket that stores terraform state   

resource "aws_s3_bucket" "tfstate" {    

  bucket        = "wordpress-3tier-state-files" 
  
  force_destroy = true 
  
} 


resource "aws_s3_bucket_ownership_controls" "s3_ownership" {  

  bucket = aws_s3_bucket.tfstate.id  
  
  rule { 
  
    object_ownership = "BucketOwnerPreferred"  
    
  } 
  
}  


resource "aws_s3_bucket_acl" "tfstate_acl" {  

  depends_on = [aws_s3_bucket_ownership_controls.s3_ownership]  
  
  bucket     = aws_s3_bucket.tfstate.id  
  
  acl        = "private"   
  
}  


# Add bucket versioning for state rollback  

resource "aws_s3_bucket_versioning" "state_versioning" {  

...  

...  

**provider. tf:** This file contains the required providers and AWS profile configuration. It sets up the foundation for our Terraform project, ensuring that the correct version of the AWS provider is used and that your AWS profile is properly configured.

terraform {  

  required_providers {  
  
    aws = {   
    
      source  = "hashicorp/aws"  
      
      version = "4.66.1"   
      
    }   
    
  } 
  
}   


provider "aws" { 

  region                   = var.aws_region  
  
  shared_credentials_files = ["~/.aws/credentials2"]   
  
}  

**backend. tf:** Specifies the location of our S3 bucket and configures the storage of our terraform state files in the same bucket.

# Store state file in S3 bucket
terraform { 

  backend "s3" {  
  
    bucket                  = "wordpress-3tier-state-files"  
    
    region                  = "us-east-1"  
    
    key                     = "wordpress-3tier/terraform.tfstate"  
    
    shared_credentials_file = "~/.aws/credentials2"   
    
  }  
  
}  


**security_groups. tf:** This contains the configurations of all the security groups & rules used by various components in the architecture.

# Security group for bastion/jump host  

resource "aws_security_group" "jumphost_sg" {  

  name        = "jumphost-SG"  
  
  description = "Allow inbound SSH traffic for jumphost"  
  
  vpc_id      = aws_vpc.project_vpc.id  
  
}  


# Allow SSH to bastion/jump host   

resource "aws_security_group_rule" "jumphost_ssh_rule" {  

  security_group_id = aws_security_group.jumphost_sg.id  
  
  type              = "ingress"  
  
  from_port         = 22  
  
  to_port           = 22  
  
  protocol          = "tcp"  
  
  cidr_blocks       = ["0.0.0.0/0"]   
  
}  


# Allow outbound traffic  

resource "aws_security_group_rule" "jumphost_outbound_rule" {  

  security_group_id = aws_security_group.jumphost_sg.id   
  
  type              = "egress"  
  
  from_port         = 0 
  
  to_port           = 0  
  
  protocol          = "-1"  
  
  cidr_blocks       = ["0.0.0.0/0"]  
  
}  

 
# Security group for elastic file system  

resource "aws_security_group" "efs_sg" {  

  name        = "efs-access"   
  
  description = "Allow NFS traffic"  
  
  vpc_id      = aws_vpc.project_vpc.id  
  
}  


resource "aws_security_group_rule" "nfs_rule" {  

  security_group_id        = aws_security_group.efs_sg.id  
  
  type                     = "ingress"  
  
  from_port                = 2049  
  
  to_port                  = 2049  
  
  protocol                 = "tcp"  
  
  source_security_group_id = aws_security_group.efs_sg.id  
  
}  

**network. tf:** Sets up a custom VPC for the project. It defines the network infrastructure, including public & private subnets, which enables better security and isolation for each tier. Other components are internet gateway, NAT gateway, route tables & associations, etc. 

# Create VPC 

resource "aws_vpc" "project_vpc" { 

  cidr_block           = var.vpc_cidr 
  
  enable_dns_hostnames = true 
  
  enable_dns_support   = true 
  

  tags = {  
  
    Name = var.vpc_name  
    
  }  
  
}  


# Create internet gateway  

resource "aws_internet_gateway" "vpc_igw" {  

  vpc_id = aws_vpc.project_vpc.id  
  

  tags = {  
  
    Name = "vpc-igw"  
    
  }  
  
}  


# Create public subnets for web tier  

resource "aws_subnet" "public_subnets" {  

...  

...  

**database. tf:** Creates a primary MySQL RDS instance and a replica (stand-by) instance in the database layer.

# Create RDS instance 

resource "aws_db_instance" "wordpress_db" { 

  identifier              = var.primary_rds_identifier 
  
  availability_zone       = var.az[0] 
  
  allocated_storage       = 10 
  
  engine                  = "mysql"  
  
  engine_version          = "8.0.32" 
  
  instance_class          = var.db_instance_type  
  
  storage_type            = "gp2"  
  
  db_subnet_group_name    = aws_db_subnet_group.RDS_subnet_grp.name  
  
  vpc_security_group_ids  = [aws_security_group.db_server_sg.id]  
  
  db_name                 = var.database_name  
  
  username                = var.database_user  
  
  password                = var.database_password  
  
  skip_final_snapshot     = true  
  
  backup_retention_period = 7  
  

  # Make sure RDS ignores any manual password change  
  
  lifecycle {  
  
    ignore_changes = [password]  
    
  }  
  
}  


# Create RDS instance replica  

resource "aws_db_instance" "wordpress_db_replica" {  

...  

... 

**compute. tf:** This file sets up the compute resources for the web and app tiers, including an elastic file system, launch templates, auto-scaling groups, an application load balancer & its target groups.


...
...
# Autoscaling group for application servers 

resource "aws_autoscaling_group" "app_server_asg" { 

  name_prefix         = "app-server-ASG" 
  
  min_size            = 2 
  
  max_size            = 4  
  
  desired_capacity    = 2  
  
  vpc_zone_identifier = aws_subnet.private_app_subnets.*.id  
  
  target_group_arns   = [aws_lb_target_group.alb_target_grp.arn]  
  

  launch_template {  
  
    id      = aws_launch_template.app_server_lt.id  
    
    version = "$Default"  
    
  }  
  

  tag {  
  
    key                 = "Name"  
    
    value               = "app-server"  
    
    propagate_at_launch = true  
    
  }  
  

  lifecycle {  
  
    create_before_destroy = true  
     
  }  
  

  depends_on = [aws_db_instance.wordpress_db, aws_efs_file_system.wordpress_EFS, aws_efs_mount_target.efs_mount]  
  
} 

**route53. tf:** This lets us route end-users to our application using a custom internet domain name. It creates a hosted zone and name servers that can be used to propagate a domain we own.

# Create hosted zone and subdomain for load balancer DNS name  

resource "aws_route53_zone" "wp_zone" {  

  name = var.domain 
  
}  


resource "aws_route53_record" "a_record" {  

  zone_id = aws_route53_zone.wp_zone.zone_id  
  
  name    = var.subdomain  
  
  type    = "A"  
  

  alias {  
  
    name                   = aws_lb.wordpress_alb.dns_name  
    
    zone_id                = aws_lb.wordpress_alb.zone_id  
    
    evaluate_target_health = true  
    
  }  
  
}  

**output. tf**: Requests for the exposure of data about specified resources within our configuration. Here, we are exposing Route53 name servers, the load balancer's URL, and the endpoints for the database and elastic file system.


... 

...  

# Loadbalancer DNS name  

output "ALB_DNS" {  

  value = aws_lb.wordpress_alb.dns_name  
  
}  


# NS records  

output "name_servers" {  

  value       = aws_route53_zone.wp_zone.name_servers  
  
  description = "records of domain name servers"  
  
}  

**variables. tf:** This contains input variables that are used to pass values from outside of the configuration. They are used to assign dynamic values to terraform's resource attributes.

# AWS region  

variable "aws_region" {  

  type        = string  
  
  default     = "us-east-1"  
  
  description = "aws region"  
  
}  


# VPC name  

variable "vpc_name" {  

  type        = string  
  
  default     = "project-vpc"  
  
  description = "name of VPC"  
  
}  


# VPC CIDR   

variable "vpc_cidr" {   

  type        = string   
  
  default     = "172.20.0.0/20"  
  
  description = "VPC CIDR block"   
  
}

# Public subnets CIDR list  

variable "public_subnets_cidr" {  

  type        = list(string)  
  
  default     = ["172.20.1.0/24", "172.20.2.0/24"]  
  
  description = "public subnets CIDR"   
  
}   

...   

...   


**userdata. tpl:** A bash script containing a collection of commands being passed to our web servers at launch time. This script creates LAMP servers, installs WordPress, automatically retrieves details of our database and populates them in the WordPress configuration file. It also mounts the elastic file system on the WordPress directory, allowing each web server to share the same WordPress files and any changes made to these files.


#!/bin/bash  

# WORDPRESS INSTALLER  


# Variables will be populated by terraform template  

db_username=${db_username}  

db_user_password=${db_user_password}  

db_name=${db_name}  

db_endpoint=${db_endpoint}  

efs_DNS=${efs_DNS}  


# Install LAMP   

apt update  

apt install -y apache2 php libapache2-mod-php php-mysql mysql-server  


# Change owner & permission of /var/www directory   

usermod -a -G apache ubuntu  

chown -R ubuntu:apache /var/www   

chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;  

find /var/www -type f -exec sudo chmod 0664 {} \;  

systemctl restart apache2  


# Install git and binutils (binutils is required for building DEB packages)  

apt install -y git binutils   

sleep 40   


# Build and install the amazon-efs-utils DEB package  

cd efs-utils/  

./build-deb.sh  

apt-get -y install ./build/amazon-efs-utils*deb  

sleep 40  


# Mount EFS in wordpress directory  

cd /var/www/html  

mkdir wordpress/  

mount -t efs -o tls ${efs_DNS}:/ /var/www/html/wordpress/  


# Download & extract wordpress zip file  

tar -xzvf latest.tar.gz  

rm latest.tar.gz  


# Create wordpress configuration file and update database values  

cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php 

sed -i "s/database_name_here/${db_name}/g" /var/www/html/wordpress/wp-config.php 

sed -i "s/username_here/${db_username}/g" /var/www/html/wordpress/wp-config.php 

sed -i "s/password_here/${db_user_password}/g" /var/www/html/wordpress/wp-config.php  

sed -i "s/localhost/${db_endpoint}/g" /var/www/html/wordpress/wp-config.php 


**To run:**
To create a replica of this project, simply follow the instructions provided in this section of the repository. Remember to change the values of the resources' attributes by tweaking the variables.tf file. If successful, the load balancer's DNS URL should redirect you to the Apache home page. To access the WordPress administrative configuration site, simply add '/wordpress' to this URL.

**For more detail, How I performed this project you can see the below points for better understanding.**

**AWS-3Tier-Wordpress-Architecture-with-Terraform**
This terraform project automates the deployment of a 3-tier architecture in AWS with WordPress installed and configured on webservers. It creates the following resources:
A custom VPC with 6 subnets: 2 public and 4 private, internet & NAT gateways, RDS MySQL databases & webservers in private subnets, security groups, a bastion host & an application load balancer in public subnets, an elastic file system, a Route53 hosted zone and autoscaling groups that create & scale bastion & webserver instances. It also creates an S3 bucket that securely stores terraform's state files.

The bastion/jump host & application load balancer reside in the public presentation layer
Wordpress application web servers are located in the private application layer
MySQL database & its standby replica are in the private database layer

**Prerequisites:**
AWS CLI configured with your access and secret keys.
Terraform installed on your local machine.
You must specify your keypair in the variables.tf file
You may also change name of VPC, CIDR, subnet IP addresses, database name/type, etc in the variables.tf file
Initialize terraform, view project's creation plan & apply:
terraform init
terraform plan
terraform apply
At this point, you will be prompted for your sensitive values i.e database password & username
Creation of resources will take a few minutes. After a successful run, the load balancer's DNS endpoint will be exposed as defined in the output.tf file. Copy this URL & paste in your browser with "/wordpress" and you will be redirected to the official Wordpress admin/registration page.

And after the completion of the 3-tier wordpress Architecture we will see this below screen-

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/12e0f419-d4f6-442a-8d03-918ab4c8cf0e)

Actually, this is how I performed 3-tier wordpress configuration. For any queries please reachout to directly over email or linkedin.








