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
// TODO Publier une petite instance AWS EC2
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

- terraform init
- terraform plan
- terraform apply

## 03_hello_apache

- cd ~
- ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
- cd 03-hello_apache
- nano main.tf

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

- terraform init
- terraform plan
- terraform apply
