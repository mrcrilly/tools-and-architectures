# Packer
**This page is still underconstruction. Some parts are out of date at this point in time.**

A simple tool with a few powerful tricks up its sleeves. There really isn't much to document here when it comes to Packer, because [the documentation](https://www.packer.io/docs/) covers it well enough. Instead I will demonstrate, with an example, how I utilise Ansible with Packer.

## Building AMIs
Using AMIs (or "golden images" as they're so often called) is an excellent way to speed up deployments of virtual machines. This is because you can pre-populate them with relavent software, configuration, and details.

Let's build an AMI, based on a CentOS 7 base AMI.

```json
{
  "variables":
  {
    "name": "elasticsearch"
    "version": "1-0-0",
    "source_ami": "ami-af4333cf",
    "aws_region": "us-east-1"
  },
  "builders":
  [
    {
      "type": "amazon-ebs",
      "access_key": "{{env `AWS_ACCESS_KEY`}}",
      "secret_key": "{{env `AWS_SECRET_KEY`}}",
      "region": "{{user `aws_region`}}",
      "source_ami": "{{user `source_ami`}}",
      "instance_type": "t2.micro",
      "ssh_username": "centos",
      "ami_name": "{{user `name`}}-{{user `version`}}"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "sudo yum install -y epel-release",
        "sudo yum install -y ansible"
      ]
    },
    {
      "type": "ansible-local",
      "playbook_file": "bootstrap.yml",
      "role_paths": [
        "roles/elasticsearch",
        "roles/telegraf",
        "roles/sudo",
        "roles/fail2ban"
      ]
    },
    {
      "type": "shell",
      "inline": [
        "sudo yum erase ansible -y",
        "sudo rm -rf /tmp/packer-provisioner-ansible-local/"
      ]
    }
  ]
}
```

Let's walk over this.

### Variables
The variables are just like thye in Terraform or Ansible: global state that can be repeated but centrally controlled and easily managed. They're useful here for various things, such as the AMI name and version. They're pretty self explanatory really.

We use two types of variable lookups here: `user` and `env`. Both are externally mutable, but only the `user` defined variables have a default defined. However like their environment variable cousins, they can be externally defined from the CLI, making your Packer build files very flexible indeed.

### Builders
All this should be very simple to follow and understand also. There's nothing special going on here, really. See [this page]() for more information.

We are however, using some pre-defined variables to make the process of building our AMI more flexible. The use of `name` seems obsolete, but it allows you to define a single file as a template and easily update it for a new service.

### Provisioners
Here is where we get some serious work done. With a provisioner, we define how the AMI is configured. Obviously Ansible is our tool of choice and as such, it's what we use here in conjunction with a simple `shell` provisioner.

Packer allows you to run Ansible as a provisioner in two ways:

1. From your local hot via an SSH proxy to the remote VM;
1. Or by uploading the Ansible code to the VM and running it locally;

I prefer the latter method as it's easier to work with, frankly. Also, correctly defined Roles allow for the service it manages to standup with everything in `defaults/main.yml`, without external configuration or management. This means a Role is capable of instilling defaults and configuration that puts us in a good state.

Before we can use Ansible, however, we have to install it remotely. That's where the `shell` provider comes in: it both installs Ansible for us and also removes it. It also clears the uploaded Ansible files as they present a minor security risk should they be left laying around.