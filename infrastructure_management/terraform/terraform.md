# Terraform
Writing infrastructure as code is liberating. It's much faster and easier than clicking around web UIs or typing away at a CLI tool's options, and it's a superior to such options, too.

With Terraform, we will define our infrastructure's architecture in version controlled, plain text. We will be able to utilise all of Terraform's key features, such as being able to plan changes before executing them, managing multiple resources across mutliple cloud providers, etc.

## Documentation
Terraform's docs are good. They can [be found here.](https://www.terraform.io/docs/index.html)

## Version
Unless some version specific features or bugs force us into using a specific version, I will always discuss the latest version provided by Hashicorp.

## Source
All the Terraform source code found throughout this page and be found in [this GitHub repository](https://github.com/mrcrilly/terraform-modules). Download it to begin utilising what we discuss in this document.

This source code is generic code and designed to work with AWS alone. Future versions will include other cloud providers.

## Structure
The best structure for a Terraform project is to use modules for each resource you control or develop. A module is simply a folder that contains Terraform state definition files. This folder (module) can then be referenced in other state files and used as a resource. The module can also take in and return variables for variable use cases and consumption.

I will also cover how we can use modules and the Terraform state database (which contains information about live infrastructure) as a dynamic Ansible inventory source. This will require an third party, but open source tool written by me, but it's a simple tool that does a simple job. The tool in question is the [ansible-inventory-terraform](https://github.com/mrcrilly/ansible-inventory-terraform) utility. This utility is documented more on the [Ansible](../ansible/terraform_inventory.md) page.

## Layout
When defining state with Terraform, you can use several files. One of the great things about Terraform is it will read all `.tf` files in the current working directory, combine them, and then run its operations.

This is the basic layout I use, which works regardless of the cloud provider because we will use modules to determine what cloud provider we're using:

```
$ tree
.
├── main.tf
├── networks.tf
├── servers.tf
└── variables.tf
```

I shall briefly explain each file.

### Main
The `main.tf` file is the central point from which we define the provider settings be used and a few other high level, administrative details. I would suggest introducing any modules in here, favouring to split them out into individual components instead.

### Networks
Here we utilise networking modules to outline what network structure(s) we would like to use. As I am mainly focusing on AWS, this would include:

- What VPC(s) I would like;
- What subnets I would like attached to the VPC(s);
- Routing tables;
- And Network ACLs for access control;

These features and resources are rolled up into modules, making the job of deploying them very simple indeed. More on this later.

### Servers
This file does exactly what you would except it to: controls the implementation and management of servers. And again, we utilise Terraform modules to make the deployment of specific services easier and more manageable. 

### Variables
This is where we define custom variables to be used throughout the state files. It can include things like the environment the state pertains to, allowing some modules to be environment-aware, which enables environment specific configurations to be applied.

## Modules
This is one of the most critical aspects of a Terraform project, because using modules will make or break your Terraform implementation and future CI/CD you builds which utilise Terraform. If we can get this part correct, then we will make life easier for ourselves going forward.

Modules will be designed to manage a single AWS resource, such as a VPC, subnet, EC2 instance, Security Group, and so on. We won't however cover the more "vendor locked in" type services.

We will cover the following basic AWS services:

- VPCs;
- Subnets;
- Network ACLs;
- Security Groups;
- EC2 instances;
- Elastic (floating) IPs;
- Additional EBS;
- ... and more to come;

With regards to things like NACLs, SGs, EIPs, and additional EBS devices, we won't develop a module for these resources but instead we will integrate them into modules that manage resources, such as database systems or load balancers, that specifcally need them.

### Documentation
The current Terraform docs on modules is of a high standard, but doesn't make some aspects about modules all that obvious. The official documentation starts [over here](https://www.terraform.io/docs/modules/index.html).

I will try my best to cover the aspects that aren't obvious from the official documentation.

### Layout
The layout for a module is also split out over a collection of files, like so:

```
├── vpc
│   ├── inputs.tf
│   ├── main.tf
│   └── outputs.tf
```

We use our `main.tf` file for the "guts" of the module, and then use `inputs.tf` and `outputs.tf` to keep the module's exposed variables separated.

It's likely we will need to build on this structure in the long run, adding in support for other features as the module deals with more complex resources.

## Windows Server
When dealing with Windows Server 2008 or above, especially in AWS (where my experience of this stems), it's important a custom AMI be made that configures a few WinRM flags. Here are the commands needed to get WinRM working remotely, allowing Teraform to do its stuff.

```
winrm set winrm/config/service/Auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
```

These have been taken directly from [the README of this Go library](https://github.com/masterzen/winrm), which Terraform uses for WinRM connections.