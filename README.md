# Terraform
 Assignment 1 :Terraform Script
# Define the provider (AWS in this case)
provider "aws" {
  region = "us-east-1"  # Modify the region as needed
}

# Create a VPC
resource "aws_vpc" "example_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Create a public subnet
resource "aws_subnet" "public_subnet" {
  vpc_id     = aws_vpc.example_vpc.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

# Create a private subnet
resource "aws_subnet" "private_subnet" {
  vpc_id     = aws_vpc.example_vpc.id
  cidr_block = "10.0.2.0/24"
}

# Create a security group for the EC2 instance
resource "aws_security_group" "instance_sg" {
  name        = "instance_sg"
  description = "Security group for EC2 instance"
}

# Define inbound rule for SSH access
resource "aws_security_group_rule" "ssh_inbound" {
  type        = "ingress"
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]  # Open to the world (for demonstration, make more secure for production)
  security_group_id = aws_security_group.instance_sg.id
}

# Define outbound rule for all traffic
resource "aws_security_group_rule" "all_traffic_outbound" {
  type        = "egress"
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
  security_group_id = aws_security_group.instance_sg.id
}

# Create an EC2 instance in the public subnet
resource "aws_instance" "example_instance" {
  ami           = "ami-0c55b159cbfafe1f0"  # Use a suitable AMI for your region
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public_subnet.id
  key_name      = "your_key_pair_name"  # Replace with your SSH key pair name
  root_block_device {
    volume_size = 8
    volume_type = "gp2"
  }
  tags = {
    Name    = "Assignment Instance"
    purpose = "Assignment"
  }
  security_groups = [aws_security_group.instance_sg.id]
}

# Output the public IP address of the EC2 instance
output "instance_public_ip" {
  value = aws_instance.example_instance.public_ip
}
