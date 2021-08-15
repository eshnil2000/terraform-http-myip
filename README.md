# MyIP

Simple data module for getting current external IP address

## Usage example

MODIFIED EXAMPLE :
```

module myip {
  source  = "4ops/myip/http"
  version = "1.0.0"
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
