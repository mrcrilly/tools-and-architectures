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
├── subnets.tf
└── variables.tf
```

I shall briefly explain each file.

### Network ACLs
The network's traffic flow must be well defined and help prevent malicious traffic going where it should not. On top of this we have Security Groups, coupled with the OS level firewall, to protect individual systems from invasion.

Here is a simple example of a Network ACLs designed to allow SSH for the entire :

```hacl
resource "aws_network_acl" "ssh_bastions" {
  vpc_id = "${aws_vpc.primary.id}"

  tags = {
    Name = "ssh_bastions"
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

Use a VPC's ID to attach the network ACL to a network. We then attach the ACL to a subnet, enabling the rule set.

The use of variables here is an indicator of how flexible and powerful Terraform is.

