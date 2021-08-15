# MyIP

Simple data module for getting current external IP address

## Usage example

MODIFIED EXAMPLE :
```
terraform {
  required_version = ">= 0.12"
}

provider "aws" {
  region = "us-east-2"

  # Allow any 2.x version of the AWS provider
  version = "~> 2.0"
}

module myip {
  source  = "4ops/myip/http"
  version = "1.0.0"
}

resource "aws_instance" "example" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]
  key_name= "aws-poc"
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  tags = {
    Name = "terraform-example"
  }
}

resource "aws_security_group" "instance" {

  name = var.security_group_name

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks =  [format("%s/%s",module.myip.address,"32")]
  }
}

variable "security_group_name" {
  description = "The name of the security group"
  type        = string
  default     = "terraform-example-instance"
}

output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP of the Instance"
}

```

Setup modules in `main.tf`:

```terraform
module myip {
  source  = "4ops/myip/http"
  version = "1.0.0"
}

resource aws_security_group allow_ssh {
  name        = "allow_ssh"
  description = "Allow SSH inbound connections"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = concat(compact(var.trusted_hosts), [module.myip.address])
  }
}
```

Also, see [example](/example) directory.
