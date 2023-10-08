**3-tier Environment Set-Up in AWS Cloud Using Terraform.**

  ![jlug5u00pgpyq649o36r](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/09f31bc9-8379-4d9f-b3b5-acbafcb33d00)


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
Eg- The webpage, the item we see or end user see in their cart, said to be the presentation layer.

Generally the presentation layer has a AWS service called CloudFront. It delivers all the web pages all the content images to the end user.

**Application layer:** This layer processes information/data and performs any business rationale/logic.
Eg- Suppose you buy something from amazon, the computation of cost, happens in logic layer, for that we need EC2, ECS which is compute services and it is also called business layer.

**Data layer:** This layer stores and manages data.
We store in RDS or in database layer.

If done properly, this architecture provides an adaptable, versatile and manageable mode of deploying applications, taking factors such as high availability, scalability, security and cost-effectiveness into consideration.

**Terraform**                  

We’ll be utilizing Infrastructure as Code (IaC) to build this project. IaC enables us to create a blueprint for our environment; allowing the provision and configuration of environments in a fast, dependable and repeatable way.

Practical-
I have performed 3-tier architecture using Terraform from Ububtu machine.
To install the AWS CLI in Ubuntu, you can use the following steps:

Update your package manager.

sudo apt update

Install the Python 3 package manager.

sudo apt install python3-pip

Install the AWS CLI.

pip3 install awscli
or
sudo apt install awscli

Once you have installed the AWS CLI, you can verify that it is installed correctly by running the following command:

aws --version

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/33213937-69ef-4c86-bea5-78bccc83eb65)

Then Configure AWS with AWS Access Key ID and AWS Secret Access Key.

aws configure

**Step 2: Install Terraform**

If you haven't already installed Terraform, you can do so by following these commands:

sudo apt-get update 

sudo apt-get install unzip -y

wget https://releases.hashicorp.com/terraform/1.0.8/terraform_1.0.8_linux_amd64.zip

unzip terraform_1.0.8_linux_amd64.zip

sudo mv terraform /usr/local/bin/

terraform --version

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/4869eb1b-ff52-4837-bd37-d00604755ad1)

Step 3: Create a Terraform configuration

Create a directory for your Terraform configuration and create a main.tf file within it. This file will define your infrastructure.

mkdir terraform-3-tier

cd terraform-3-tier

touch main.tf

vim main.tf

Edit the main.tf file using your preferred text editor and add the following configuration:

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/0857b8f0-a212-41db-8357-5e70b7e6eb2d)


# Define AWS provider

provider "aws" {

  region = "us-east-1" # Change this to your desired region
  
}


# Create a VPC

resource "aws_vpc" "my_vpc" {

  cidr_block = "10.0.0.0/16"
  
}


# Create subnets (for each tier)

resource "aws_subnet" "web_subnet" { 

  vpc_id                  = aws_vpc.my_vpc.id 
  
  cidr_block              = "10.0.1.0/24" 
  
  availability_zone       = "us-east-1a" 
  
  map_public_ip_on_launch = true
  
} 


resource "aws_subnet" "app_subnet" {

  vpc_id                  = aws_vpc.my_vpc.id
  
  cidr_block              = "10.0.2.0/24"
  
  availability_zone       = "us-east-1b"
  
}


resource "aws_subnet" "db_subnet" {

  vpc_id                  = aws_vpc.my_vpc.id
  
  cidr_block              = "10.0.3.0/24"
  
  availability_zone       = "us-east-1c"
  
}


# Create security groups (for each tier)

resource "aws_security_group" "web_sg" {

  name        = "web_sg"
  
  description = "Web Security Group"
  
  vpc_id      = aws_vpc.my_vpc.id
  

  # Define inbound and outbound rules as needed
  
}


resource "aws_security_group" "app_sg" {

  name        = "app_sg"
  
  description = "Application Security Group"
  
  vpc_id      = aws_vpc.my_vpc.id
  

  # Define inbound and outbound rules as needed
  
}


resource "aws_security_group" "db_sg" {

  name        = "db_sg"
  
  description = "Database Security Group"
  
  vpc_id      = aws_vpc.my_vpc.id
  

  # Define inbound and outbound rules as needed
  
}


# Create EC2 instances (for each tier)

resource "aws_instance" "web_instance" {

  ami           = "ami-0c55b159cbfafe1f0" # Replace with your desired AMI
  
  instance_type = "t2.micro"
  
  subnet_id     = aws_subnet.web_subnet.id
  
  security_groups = [aws_security_group.web_sg.id]
  

  # Configure other instance settings as needed
  
}


resource "aws_instance" "app_instance" {

  ami           = "ami-0c55b159cbfafe1f0" # Replace with your desired AMI
  
  instance_type = "t2.micro"
  
  subnet_id     = aws_subnet.app_subnet.id
  
  security_groups = [aws_security_group.app_sg.id]
  

  # Configure other instance settings as needed
  
}


resource "aws_instance" "db_instance" { 

  ami           = "ami-0c55b159cbfafe1f0" # Replace with your desired AMI 
  
  instance_type = "t2.micro" 
  
  subnet_id     = aws_subnet.db_subnet.id 
  
  security_groups = [aws_security_group.db_sg.id]
  

  # Configure other instance settings as needed
  
}


![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/2efd0415-8b46-40d6-ac0d-82bc23488a71)

terraform init 

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/6cdebdac-53c7-49f0-8b42-e7835c4fd1d9)

terraform plan

terraform apply 

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/00f661f8-b7a0-476c-989c-adbd35933088)

Now, by the help of terrform we have created 3-tier architecture in AWS cloud
Presentation layer
Application layer
Data layer

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/940bd062-465c-4641-aae2-7b86b9e49f09)


This is how basic 3-tier architecutre looks like. We can do configuration on the basis of our requirement.





















