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

## 06_hello_github
 
 nano main.tf
 ```
provider "github" {
  token = "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"
}

resource "github_repository" "mon_repo" {
  name        = var.nom_du_repo
  description = "Créé avec Terraform"
  private     = true
}

variable "nom_du_repo" {
  description = "Nom du dépôt GitHub"
  type        = string
  default     = "test"
}
 ```
nano terraform.py
 ```
import subprocess
import sys
nom_du_repo = sys.argv[1]
subprocess.run(['terraform', 'apply', '-var', f'nom_du_repo={nom_du_repo}'])
 ```
 ```
sudo python3 terraform.py my_repo
 ```

## 07_hello_vultr

 ```
provider "vultr" {
  api_key = "SILLVA2A6J3F6S4SKKSNXAPFNZFMWNFF2MRA"
}

resource "vultr_firewall_group" "my_firewallgroup" {
    description = "base firewall"
}

resource "vultr_firewall_rule" "ssh_rule" {
    firewall_group_id = vultr_firewall_group.my_firewallgroup.id
    protocol          = "tcp"
    ip_type           = "v4"
    subnet            = "0.0.0.0"
    subnet_size       = 0
    port              = "22"
    notes             = "ssh"
}

resource "vultr_firewall_rule" "http_rule" {
    firewall_group_id = vultr_firewall_group.my_firewallgroup.id
    protocol          = "tcp"
    ip_type           = "v4"
    subnet            = "0.0.0.0"
    subnet_size       = 0
    port              = "80"
    notes             = "http"
}

resource "vultr_instance" "example" {
  region       = "fra"
  plan         = "vc2-1c-1gb"
  os_id        = "387"
  hostname     = "exo-7"

  user_data = <<-EOF
    "#!/bin/sh
    "sudo apt update"
    "sudo apt install -y apt-transport-https ca-certificates curl software-properties-common curl"
    "sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg"
    "sudo echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable' | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null"
    "sudo apt update"
    "sudo apt install -y docker-ce docker-ce-cli containerd.io"
    "sudo systemctl start docker"
    "sudo docker pull satzisa/html5-speedtest"
    "sudo docker run -d -p 80:80 satzisa/html5-speedtest"
  EOF
}
 ```
output.tf
 ```
output "ip" {
  value = vultr_instance.example.main_ip
}
 ```

## 08_hello_wordpress

 ```
provider "vultr" {
 api_key = "SILLVA2A6J3F6S4SKKSNXAPFNZFMWNFF2MRA"
}

resource "vultr_firewall_group" "my_firewallgroup" {
   description = "base firewall"
}

resource "vultr_firewall_rule" "ssh_rule" {
   firewall_group_id = vultr_firewall_group.my_firewallgroup.id
   protocol          = "tcp"
   ip_type           = "v4"
   subnet            = "0.0.0.0"
   subnet_size       = 0
   port              = "22"
   notes             = "ssh"
}

resource "vultr_firewall_rule" "http_rule" {
   firewall_group_id = vultr_firewall_group.my_firewallgroup.id
   protocol          = "tcp"
   ip_type           = "v4"
   subnet            = "0.0.0.0"
   subnet_size       = 0
   port              = "80"
   notes             = "http"
}

resource "vultr_instance" "example" {
 region       = "fra"
 plan         = "vc2-1c-1gb"
 os_id        = "387"
 hostname     = "exo-8"

  user_data = <<-EOF
    "#!/bin/sh
    "sudo apt update"
    "sudo apt install -y apt-transport-https ca-certificates curl software-properties-common curl"
    "sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg"
    "sudo echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable' | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null"
    "sudo apt update"
    "sudo apt install -y docker-ce docker-ce-cli containerd.io"
    "sudo systemctl start docker"
    "sudo docker pull mysql:latest"
    "sudo docker pull wordpress:latest"
    "sudo docker network create wordpress_network"
    "sudo docker run -d --name mysql-container --network wordpress_network -e MYSQL_ROOT_PASSWORD=your_mysql_root_password -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=your_mysql_password mysql:latest"
    "sudo docker run -d --name wordpress-container --network wordpress_network -p 80:80 -e WORDPRESS_DB_HOST=mysql-container -e WORDPRESS_DB_NAME=wordpress -e WORDPRESS_DB_USER=wordpress -e WORDPRESS_DB_PASSWORD=your_mysql_password wordpress:latest"
  EOF
}
 ```
output.tf
 ```
output "ip" {
  value = vultr_instance.example.main_ip
}
 ```
