**We need to write code that will query the meta data of an instance within AWS
and provide a json formatted output.**

I have tried doing this in my console.  

To retrieve and formatting AWS instance metadata, I followed these steps: 


1. Ensure we have the AWS CLI and jq installed on our AWS EC2 instance.
   

jq is generally lightweight and powerful command-line tool used for processing JSON data.  

Commands to follow to complete this process effectively are provides below-  


sudo yum update  

sudo yum install awscli  

sudo yum update  

sudo yum install jq  


2. check the version of AWS CLI and jq using the below commands:
   

aws --version  

jq --version  


3. Before executing the meta data script configure the aws cli using the below command:
   

aws configure  

provide access key ID   

and secret key ID   

#Which we have to provide from our AWS account  

![image](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/4d0978f7-b2d0-4f86-89cd-e0390ba108ec)   


4. Create a new file, named get_instance_metadata.sh,
    

touch get_instance_metadata.sh  


5. Edit the file "get_instance_metadata.sh" :
   

vim get_instance_metadata.sh   


6. Script:
   

#!/bin/bash  



if ! command -v aws &> /dev/null; then  

    echo "AWS CLI is not installed. Please install it first."   
    
    exit 1  
    
fi   


# Retrieve instance metadata and format as JSON   

instance_metadata=$(aws ec2 describe-instances --instance-id $(curl -s http://18.233.152.140/latest/meta-data/instance-id) --query 'Reservations[0].Instances[0]' --output json)  


# Check if the AWS CLI command was successful   

if [ $? -eq 0 ]; then  

    # Print the JSON output  
    
    echo "$instance_metadata"   
    
else  

    echo "Failed to retrieve instance metadata."  
    
fi  

![image (2)](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/751d163e-3d92-48c5-82ee-529c7d57bfad)  



7. Make sure the script is executable:
   

 chmod +x get_instance_metadata.sh  
 

8. Run the script:
   
 
./get_instance_metadata.sh    



![image (1)](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/db1dcfe7-e41f-41eb-ab45-324e91d15a58)  



![image (3)](https://github.com/Ankitkumarjaiswal23/Tech_challenge/assets/112700507/80357435-175d-408f-8ecb-1c6a1372d39b)   



This is the json format output which I got after running the query meta data of an instance within AWS. 



