
# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

Terraform is an open-source, infrastructure as code, software tool created by HashiCorp. Users define and provide data center infrastructure using a declarative configuration language known as HashiCorp Configuration Language, or optionally JSON.

![Project Architecture Diagram] (https://darey.io/wp-content/uploads/2021/07/tooling_project_16.png)

#### Project Requirements
Install Terraform on PC 
Create an IAM profile on AWS with programmatic access
Install AWS CLI tool and configure the IAM profile with the access provided above

Install Terraform and AWS CLI

```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | \
    gpg --dearmor | \
    sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt-get install terraform

#AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

which aws
```

### Terraform 

Create VPC, Subnets and security groups

```
provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```

Executing Terraform code
```
terraform init #Initialize terraform project
terraform plan #Check the outputs of the instructions
terraform apply #Create the resources specified in the configuration file
```


