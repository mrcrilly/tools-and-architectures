# Terraform
Writing infrastructure as code is liberating. It's much faster and easier than clicking around web UIs or typing away at a CLI tool's options, and it's a superior to such options, too.

With Terraform, we will define our infrastructure's architecture in version controlled, plain text. We will be able to utilise all of Terraform's key features, such as being able to plan changes before executing them, managing multiple resources across mutliple cloud providers, etc.

## Documentation
Terraform's docs are good. They can [be found here.](https://www.terraform.io/docs/index.html)

## Layout
When defining state with Terraform, you can use several files. One of the great thing about Terraform is it will read all `.tf` files in the current working directory, combine them, and then run its operations.

This is the basic layout I like to use, but it changes based on the cloud provider. This layout is suitable for AWS:

```
$ tree
.
├── network_acls.tf
├── networks.tf
├── provider.tf
├── routing_tables.tf
├── security_groups.tf
├── servers.tf
└── variables.tf
```

I shall briefly explain each file.

### Network ACLs
The network's traffic flow must be well defined and help prevent malicious traffic going where it should not. On top of this we have Security Groups, coupled with the OS level firewall, to protect individual systems from invasion.

Here is a simple example of a Network ACLs designed to allow SSH for the entire :

```hcl
resource "aws_network_acl" "ssh_bastions" {
  vpc_id = "${aws_vpc.primary.id}"

  tags = {
    Name = "ssh_bastions"
    Environment = "${environment_name}"
  }

  subnet_ids = [
    "${aws_subnet.ssh_bastions.id}"
  ]

  egress {
    rule_no = 2
    protocol = "tcp"
    action = "allow"
    cidr_block = "${aws_vpc.primary.cidr_block}"
    from_port = 22
    to_port = 22
  }

  ingress {
    rule_no = 1
    protocol = "tcp"
    action = "allow"
    cidr_block = "0.0.0.0/0"
    from_port = 22
    to_port = 22
  }
}
```

We use a VPC's ID to attach the network ACL to a network. We then attach the ACL to a subnet, enabling the rule set.

The use of variables here is an indicator of how flexible and powerful Terraform is.

### Networks
Here we define our VPCs, Internet Gateways, and subnets. It's probably the second largest file after the `servers.tf` file. I won't provide a fulle example as that would result in a massively long page, but instead some examples of what a each element in this might look like.

#### VPCs
```hcl
resource "aws_vpc" "primary" {
  cidr_block = "10.3.0.0/16"
  tags {
    Name = "primary_vpc"
    Environment = "${environment_name}"
  }
}
```

With this defined, we can see where the previous `${aws_vpc.primary.cidr_block}` usage comes from.

A brief note on the name `primary` here: I highly recommend using names for VPCs, Gateways, and so, which don't relate to a specific environment. This will allow you to abstract the Terraform state from any given environment.

#### Internet Gateways
```hcl
resource "aws_internet_gateway" "primary" {
  vpc_id = "${aws_vpc.primary.id}"

  tags {
    Name = "internet_gateway"
    Environment = "${environment_name}"
  }
}
```

An Internet Gateway in AWS is very simple in its function and form. I will not discuss it further here.

#### Subnets
```hcl
resource "aws_subnet" "management_ssh_bastions" {
  vpc_id = "${aws_vpc.primary.id}"

  tags {
    Name = "management_ssh_bastions"
    Environment = "${environment_name}"
  }

  cidr_block = "10.3.1.0/24"
}
```

I will discuss subnets in more detail when I discuss architectures, but for now I will leave subnets to do what they're best at: define collision domains and segment our networks into little efficient boxes.

### Provider
This is a file which simply contains authentication details for all relavent providers. However I believe such credentials should be managed from environment variables in `.bash_profile`, securing the state files from holding sensitive secrets whilst being held up in (potentially public) source control.

```hcl
# Creds and region pulled in from environment variables
provider "aws" {}
```

### Routing Tables
Simple enough in their operation and simple to define too.

```hcl
resource "aws_route_table" "public" {
  vpc_id = "${aws_vpc.primary.id}"
  route {
    cidr_block = "${environment_subnet}"
    gateway_id = "${aws_internet_gateway.primary.id}"
  }

  tags {
    Name = "public"
    Environment = "${environment_name}"
  }
}
```

### Security Groups
Essentially firewall rules at the VM layer, as opposed to Network ACLs at the subnet level. Very important to implement, and I implement one per service, not one for a given server's role. Instead, chaining together multiple, simple rules to meet your needs is the better solution.

```hcl
resource "aws_security_group" "all_public_lb" {
  vpc_id = "${aws_vpc.primary.id}"
  name = "all_public_lb"

  tags = {
    Name = "all_public_lb"
    Environment = "${environment_name}"
  }

  ingress {
    from_port = 443
    to_port = 443
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Servers
This will likely be the largest file of them all. It's quite possible you will need to divide this file up further if your estate is big (hundreds of servers and beyond.) If you're an AWS, GCE, or a user of some of feature rich cloud provider, then it's likely you'll use more than servers, utilising fancy cloud services like [Lambda](https://aws.amazon.com/lambda/details/), [ELBs](https://aws.amazon.com/elasticloadbalancing/), and so on. 

When I eventually discuss architecture, I will demonstrate how I dislike cloud services which lock you in, favouring to implement most of their features my self. You might disagree with me on this point, and that's perfectly fine (and expected), but I'm opinionated, and this guide is my opinion.

```hcl
resource "aws_instance" "internet-gateway" {
   count = "2"
   ami = "ami-af4333cf" # CentOS7
   instance_type = "t2.micro"

   key_name = "deployment"

   vpc_security_group_ids = [
     "${aws_security_group.public_internet_in.id}",
     "${aws_security_group.public_ssh_in.id}"
   ]

   subnet_id = "${aws_subnet.public.id}"

   tags {
     Name = "Gateway"
     Environment = "${environment_name}"
   }
 }
```

### Variables
And finally some custom variables. In our example state files, we've done nothing more than use a single variable: `${environment_name}`. It's simple in its definition and use.

```hcl
variable "environment_name" {
  type = "string"
  default = "QE"
}
```

Such variables allow you to change environment specific settings like IP ranges, service names, etc, in one fell swoop.