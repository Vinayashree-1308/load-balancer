Provider "aws" {
  access_key = "AKIAZXVJ7ROTW5QMOAFJ"
  secret_key = "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"
}

variable "3" {
  description = "Number of instances to create and attach to load balancer"
}
variable "vpc-0df44c1054f754ecf" {
  description = " VPC ID where EC2 are going create "
}

##############################################################
# Data sources to get VPC, subnets and security group details
##############################################################
data "aws_vpc" "AND-VPC" {
  id = "${var.vpc-0df44c1054f754ecf}"
}

data "aws_subnet_ids" "AND-VPC" {
  vpc_id = var.vpc-0df44c1054f754ecf
}

data "aws_security_group" "AND-VPC" {
  vpc_id = data.aws_vpc.AND-VPC.vpc-0df44c1054f754ecf
  
}

##############################################################
#Creation of Load balancer 
module "elb" {
name = "AND-Load-Balancer"
availability_zones = ["us-west-2a", "us-west-2b", "us-west-2c"]


#Declaring Subnets under specific VPC( VPC Name: AND-VPC )
subnets         = data.aws_subnet_ids.AND-VPC.vpc-0df44c1054f754ecf
security_groups = [data.aws_security_group.AND-VPC.vpc-0df44c1054f754ecf]

#Declaring the listners

listener = [
    {
      instance_port     = "80"
      instance_protocol = "http"
      lb_port           = "80"
      lb_protocol       = "http"
    },
    {
      instance_port     = "8080"
      instance_protocol = "http"
      lb_port           = "8080"
      lb_protocol       = "http"
},
]
health_check = {
    target              = "HTTP:80/"
    interval            = 30
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
  }

# ELB attachments
  number_of_instances = var.3
  instances           = module.ec2_instances.ami-07f3715a1f6dbb6d9
}

##############################################################
# Tagging EC2 instances to load balancer
##############################################################
module "ec2_instances" {
instance_count = var.3

  name                        = "AND-Instances"
  ami                         = "ami-07f3715a1f6dbb6d9"
  instance_type               = "t2.micro"
  vpc_security_group_ids      = [data.aws_security_group.AND-VPC.vpc-0df44c1054f754ecf]
  subnet_id                   = element(tolist(data.aws_subnet_ids.AND-VPC.vpc-0df44c1054f754ecf), 0)
  associate_public_ip_address = true
}
