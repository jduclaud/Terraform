# Terraform

## 01_hello_terraform

Installation de Terraform sur la CloudShell AWS :
- git clone https://github.com/tfutils/tfenv.git ~/.tfenv
- echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
- exec $SHELL
- tfenv install latest
- tfenv use 1.6.4

- https://github.com/DocteurSEO/terraform.git
- cd terraform/01_hello_terraform/
- nano main.tf
 ``` 
provider "aws" {
  region = "eu-west-3"
}

resource "aws_instance" "debian12" {
 ami           = "ami-087da76081e7685da"
 instance_type = "t2.micro"
}
 ``` 
- terraform init
- terraform plan
- terraform apply

## 02_hello_vpc

- cd ../02_hello_vpc
- nano main.tf
 ``` 
provider "aws" {
  region = "eu-west-3"
}

resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true

  tags = {
    Name = "MonVPC"
  }
}

resource "aws_subnet" "my_subnet" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "MonSubnet"
  }
}
 ``` 
- terraform init
- terraform plan
- terraform apply

## 03_hello_apache

- ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
- cd 03-hello_apache
- nano main.tf
 ``` 
provider "aws" {
  region = "us-west-1"
}

resource "aws_key_pair" "my_key_pair" {
  key_name   = "my-key-pair"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_security_group" "my_security_group" {
  name        = "my-security-group"
  description = "Allow incoming HTTP traffic"

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "my_instance" {
  ami           = "ami-0cbd40f694b804622"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.my_key_pair.key_name
  vpc_security_group_ids = [aws_security_group.my_security_group.id]

  user_data = <<-EOF
              #!/bin/bash
              apt update
              apt install -y apache2
              echo "Hello Terraform" > /var/www/html/index.html
              systemctl start apache2
              systemctl enable apache2
              EOF

  tags = {
    Name = "MyEC2Instance"
  }
}
 ``` 
- terraform init
- terraform plan
- terraform apply

## 04_hello_debug

- cd 04_hello_debug
- nano main.tf
 ```
variable "region" {}
variable "vpc_cidr" {}
variable "subnet_cidr" {}
variable "availability_zone" {}

provider "aws" {
  region = "${var.region}"
}

resource "aws_vpc" "default" {
  cidr_block = "${var.vpc_cidr}"
}

resource "aws_subnet" "subnet-1" {
  vpc_id            = aws_vpc.default.id
  cidr_block        = "${var.subnet_cidr}"
  availability_zone = "${var.availability_zone}"
}

resource "aws_security_group" "default" {
  name        = "test-security-group"
  description = "Allow TCP/ICMP"
  vpc_id      = aws_vpc.default.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 1000
    to_port     = 2000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "nginx_instance" {
  ami           = "ami-0cbd40f694b804622" # Remplacez par l'ID AMI approprié
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet-1.id
  vpc_security_group_ids = [aws_security_group.default.id]
  tags = {
    Name = "nginx-proxy"
  }
}

resource "aws_instance" "web1" {
  ami           = "ami-0cbd40f694b804622" # Remplacez par l'ID AMI approprié
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet-1.id
  vpc_security_group_ids = [aws_security_group.default.id]
  tags = {
    Name = "web1"
  }
}

resource "aws_instance" "web2" {
  ami           = "ami-0cbd40f694b804622" # Remplacez par l'ID AMI approprié
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet-1.id
  vpc_security_group_ids = [aws_security_group.default.id]
  tags = {
    Name = "web2"
  }
}

resource "aws_instance" "web3" {
  ami           = "ami-0cbd40f694b804622" # Remplacez par l'ID AMI approprié
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet-1.id
  vpc_security_group_ids = [aws_security_group.default.id]
  tags = {
    Name = "web3"
  }
}

resource "aws_instance" "mysqldb" {
  ami           = "ami-0cbd40f694b804622" # Remplacez par l'ID AMI approprié
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet-1.id
  vpc_security_group_ids = [aws_security_group.default.id]
  tags = {
    Name = "mysqldb"
  }
}
 ```
nano terraform.tfvar
 ```
region = "us-west-1"
vpc_cidr = "10.0.0.0/16"
subnet_cidr = "10.0.1.0/24"
availability_zone = "us-west-1a"
 ```

- terraform init
- terraform plan -var-file terraform.tfvar
- terraform apply -var-file terraform.tfvar

## 05_hello_module

- cd 05_hello_module
- nano main.tf
 ```
provider "aws" {
  region  = "us-east-1"
}

module "vpc" {
   source = "terraform-aws-modules/vpc/aws"
   name = "wordpress"
   cidr = "10.0.0.0/16"
   azs = ["us-east-1a", "us-east-1b"]
   public_subnets = ["10.0.0.0/24", "10.0.1.0/24"]
   private_subnets = ["10.0.2.0/24", "10.0.3.0/24"]
   intra_subnets = ["10.0.4.0/24", "10.0.5.0/24"]
   database_subnets = ["10.0.6.0/24", "10.0.7.0/24"]
   enable_nat_gateway = true
   enable_dns_hostnames = true
}

module "wordpress" {
   source  = "atpoirie/wordpress-ecs/aws"
   version = "1.0.0"
   ecs_service_subnet_ids = module.vpc.private_subnets
   lb_subnet_ids = module.vpc.public_subnets
   db_subnet_group_subnet_ids = module.vpc.database_subnets
}
 ```
- terraform init
- terraform plan
- terraform apply

## 05_hello_github
 ```
provider "github" {
  token = "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"
}

resource "github_repository" "mon_repo" {
  name        = "my_new_repo"
  description = "Créé avec Terraform"
  private     = true
}
 ```
