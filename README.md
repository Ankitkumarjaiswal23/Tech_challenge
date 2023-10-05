**Deploying a 3-Tier WordPress Architecture on AWS using Terraform.**                 

**First of all we should know about the 3 tier architecture- ** 

The 3-tier architecture is a common software architectural design that comprises three layers, each with its obligations:

Presentation layer: This layer is responsible for presenting information to the end user, receiving data from the user and dealing with client interactions.

Application layer: This layer processes information/data and performs any business rationale/logic.

Data layer: This layer stores and manages data

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





