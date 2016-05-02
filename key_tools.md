# Key Tools
Some tools are central to daily operations. For me, these tend to be:

* Ansible
* Packer
* Terraform
* Vagrant (and by extention, VirtualBox)

I use these almost daily. Obviously as a DevOp, I mainly manage infrastructure and its configuration, so it's no surprise these tools play a critical part in my tool belt. Because of this, a lot of the tools that are discussed in this guide are implemented and managed using these key tools.

## Workflow
For those familiar with these tools, their workflow and place within should be obvious, but for the uninitiated:

1. Ansible Roles/Playbooks define how a tool is configured;
1. Packer uses Ansible and a cloud provider account to build images;
1. Terraform consumes those images and builds our infrastructure;
1. Ansible comes into play again and configures the infrastructure built by Terraform;

This work flow can be heavily automated, and this will be demonstrated in time.

## Infrastructure
Before we can build anything, we need infrastructure. I like to use a two layered system for building infratucture:

1. Development
1. Stage/Production

For development, I use VirtualBox. For stage or production, I favour AWS.

When defining Terraform state in the future, when I discuss architectures, I will provide two state files: one for development using VirtualBox and a second for AWS for use in production.

If you use a different cloud provider, perhaps DigitalOcean, you'll find Terraform [supports a lot of providers.](https://www.terraform.io/docs/providers/index.html) You'll be expected to write the state files you'll need, but I'm more than hapy to help you out if you struggle.

## Monitoring Driven Infrastructure (MDI)
As I've previously stated, theory is a type of tool. I would like to present a theory here, briefly, regarding how I intend to develop infrastructure going forward.

I like to call it Monitoring Driven Infrastructure, a phrase I've coined off of the back of [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) and one I like to use to describe how infrastructure should be developed. Simply put: I believe your monitoring solution should be developed and put in place first, with all checks and thresholds defined, followed by your actual infrastructure housing your application(s). Your application and all resources supporting it should be refined to turn your monitoring dashboard of choice from red to green.