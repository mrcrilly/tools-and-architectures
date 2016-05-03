# Ansible
Now owned by RedHat, Ansible is a very flexible, powerful, and fast configuration management tool. It's architecture is very simple and when combined with a few key tools, it will be the only tool you need to define your golden images for VMs.

Throughout this guide, we will use only Ansible to configure systems. No manual steps will be defined, only Ansible Playbooks and Roles.

All Playbooks and Roles will only focus on RHEL based distributions. There are many other fine Linux distributions out there, but one must be selected and I've selected CentOS. 

## Version
This guide assumes the reader is using Ansible `v2.*`, the current latest stable major version at the time of writing.

## Documentation
The Ansible documentation is relatively good, but does lack some details. Review [the documentation](http://docs.ansible.com/ansible/index.html) to learn the basics of Ansible if you're not familiar.

## Structure
The structure is so simple, we will do nothing more than simply outline it below:

```
$ tree -L 1
.
├── README.md
├── playbooks
└── roles
```

Yup, that's it. The `roles/` directory holds our roles, and the `playbooks/` directory our playbooks. There is no `ansible.cfg` present here because it is loaded from, among other possible locations, the current working directory, which in this case will be our configuration repository explained later.

## Playbooks
A playbook can do multiple things for us. It can wrap a set of tasks which we can then repeatedly use on any number of systems. It can also "import" and use roles, which we will discuss below, which often leads to a cleaner playbook. A playbook, unlike a role, has a unique job: it brings together tasks and roles and defines what host patterns they're applied to. We won't cover everything here, but here is an example playbook utilising some roles, tasks, and keeping the structure clean and tidy:

```yaml
---

- name: Bootstap everything
  hosts: all
  become: true
  roles:
    - bootstrap
    - users
    - security

- name: Setup our web server
  hosts: webserver
  become: true
  pre_tasks:
    - name: Some GET request because example
      get_url:
        url: http://internal-system

  roles:
    - { role: nginx, nginx: client_facing_nginx }
    - { role: postgresql, postgresql: postgresql_clean }
    - { role: magicmidget, magicmidget: client_facing_magicmidget }
```

This playbook has multiple plays in it. It demonstrates a clean approach to tabs, a lack of string quotes, and simple layout.

There is very little to be said about playbooks. They're mostly a free-for-all.

## Roles
A role is a managed resource that can be applied to a system, or an entire estate of systems, easily. It is self-sufficient, but also allows for all aspects of its behaviour to be configured or simply ignored.

When building a new role, it should follow a very simple philosophy: **design it to manage a single resource and do one job (and only one job) really well**. It will be easier to use, update, and retire as time passes.

###Standards
All roles should be self-sufficient. At minimum they should contain a working configuration in the `defaults/main.yml` file and it should be possible to stand up a role without any additional setup or changes being made to the code. This kind of design allows anyone to easily test the role in a VM (perhaps using Vagrant) and make sure it does what it claims in a consistent, repeatable manner.

To achieve this, roles should have the following logic and conditions in place:

- A complete, working configuration in `defaults/main.yml`;
- A set of configuration flags that allows the user to ignore certain aspects of the role;
- All metadata in place so ownership, versions, etc, can be tracked;
- A complete `README.md` demonstrating usage and explaining the configuration (at minimum);
- (Optional) Handlers for managing service restarts on configuration file changes;
- (Optional) A `Vagrantfile` others can use to quickly test the role in a safe environment;

With the above rules in place, our roles will be high quality and very flexible.

### Structure
The structure of a role's directory should simply match the structure given to us by `ansible-galaxy init role_name`. That way it's always consistent. Of course as the role author you are free to delete any unused directories.

```
.
├── README.md
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   ├── configuration.yml
│   ├── firewall.yml
│   ├── main.yml
│   └── software.yml
└── templates
    └── vault.service
```

For anyone who has followed Ansibe's documentation, the above structure should be simple enough and easy to follow. However we will now address two very important parts about a role:

- The `tasks/` structure;
- The configuration YAML data structure;

### Tasks
The `tasks/` folder structure is an important part of making the role easier to understand, use, and maintain. This is how it's broken down:

- `software.yml`: anything related to installing software, be it a dependency or the software your role is managing;
- `configuration.yml`: anything related to configuring third party software, your software, or the system state minus a few things (see below);
- `firewall.yml`: manage the system's firewall to allow networking access to your resource(s);
- `users.yml`: manage system level user access as needed. It's a good idea to create a system user with which to install and run the software's process(es);
- `selinux.yml`: configure any SELinux (or GRSec) related fields here;

This is more of a pattern rather than a hard rule on what the files should be called or limited to. You can name the files anything you want, and have any number of them because we just have one philosophy to follow here: split out the logic of your role into individual playbooks so they can be maintained easily and selected or bypassed with tags and conditions.

From with the `main.yml` file, we simply include the above files, but with an additional set of features in place, tags and when.

The order of inclusion is somewhat unimportant, but keep it consistent. Also to put it simply: as long as everything needed by the software is being dealt with before turning on the service/process, then it doesn't really matter.

We use this structure because it should be easy for anyone coming to your role to find the relevant information quickly, and modify it with ease.

```yaml
---
- include: users.yml
  tags:
    - magicmidget-users
  when: magicmidget.manage.users

- include: software.yml
  tags:
    - magicmidget-software
  when: magicmidget.manage.software

- include: configuration.yml
  tags:
    - magicmidget-configuration
  when: magicmidget.manage.configuration

- include: firewall.yml
  tags:
    - magicmidget-firewall
  when: magicmidget.manage.firewall

- include: selinux.yml
  tags:
    - magicmidget-selinux
  when: magicmidget.manage.selinux
```

Now, the tags. These are very important because they open up a world of pleasure. Consider these two calls to `ansible-playbook`:

`ansible-playbook magicmidget.yml -i inv.ini -e @environment.yml`

Versus

`ansible-playbook magicmidget.yml -i inv.ini -e @environment.yml --tags magicmidget-configuration`

In the event you need to update the configuration for your `magicmidget` installations, across your entire network, which of the above two executions is going to be fastest? Of course it's the latter, because the use of tags here means we can focus in on a single playbook within the role. This means only that playbook's code is executed against the network.

The `when:` clauses are also very important. These clauses allow us to optionally ignore parts of the role, without having to use tags, from within the role's very own configuration. This could be because you install a particular piece of software in a certain way, and so you don't want the role to manage that for you, but the configuration management is a great convenience, so you do want that part. It allows the role to become more flexible and perform only what the user wants.

Using tags and when will dramatically cut down the execution time of your roles when used correctly.

### Role Configuration
When outlining the configuration for a role, it's important to remember that a role might be used more than once in a playbook or a network. Therefore, roles need to offer two things:

- A configuration format that can be repeated multiple times in a single scope for multiple installations;
- An interface to set the configuration for the role, essentially parameterizing the role;

Therefore, we construct our role's configuration as such:

```yaml
---
magicmidget:
  manage:
    users: true
    software: true
    configuration: true
    firewall: true
    selinux: true

  software:
    repository: http://something.com/magicmidget.rpm
    version: v1.0.1

  user:
    name: magicmidget
    group: magicmidget
    uid: 5001
    gid: 5001

  configuration:
    local_file: "templates/magicmidget.conf.j2"
    remote_file: "etc/magicmidget/magicmidget.conf"

  firewall:
    - port: "8080/tcp"
    - port: "8081/udp"

  selinux:
    - key: "allow.this"
      value: true
```

This is not a final or static structure. That is, the keys we see here, such as user, configuration, can be omitted or added to as you, the role author, see fit. However there are important aspects about this data structure:

- It's very easy to find particular parts of the configuration quickly;
- It's a repeatable structure;
- It can be very consistent across all your roles;

There are some negatives to using a top level key and defining a block this way: without setting `hash_behaviour` to `merge` (the default is `replace`), you have to repeat the entire block each and every time you need to apply the configuration in a different way. This can suck, but has a positive flip side in that the configuration is complete and clear, and no inheritance rules need to be followed or trace to understand or determine where the configuration is coming from.

### Example
Consider our fictional(?) `magicmidget` software, mentioned above a few times. It can be utilised in more than one way within a network, so let's make two sets of configuration for it:

```yaml
client_facing_magicmidget:
  manage:
    users: false # client manages the users on their server
    software: true
    configuration: true
    firewall: false # client has hardware firewall; doesn't want OS level f/w
    selinux: false # client uses GRSec, which we don't support

  software:
    repository: http://something.com/magicmidget.rpm
    version: v1.0.1

  user:
    name: client_mm_user # client provided user and uid/gid
    group: client_mm_user
    uid: 6789
    gid: 6789

  configuration:
    local_file: "templates/magicmidget.conf.j2"
    remote_file: "etc/magicmidget/magicmidget.conf"

  firewall:
    - port: "8080/tcp"
    - port: "8081/udp"

  selinux:
    - key: "allow.this"
      value: true

internal_facing_magicmidget:
  manage:
    users: true
    software: true
    configuration: true
    firewall: true
    selinux: true

  software:
    repository: http://something.com/magicmidget.rpm
    version: v2.0.0

  user:
    name: magicmidget
    group: magicmidget
    uid: 5001
    gid: 5001

  configuration:
    local_file: "templates/magicmidget_internal.conf.j2"
    remote_file: "etc/magicmidget/magicmidget.conf"

  firewall:
    - port: "8080/tcp"
    - port: "8081/udp"

  selinux:
    - key: "allow.this"
      value: true
```

With this multi-instance configuration, we can now use our role, passing in our configuration as a parameter:

```yaml
---
- hosts: all
  become: true
  roles:
    - { role: magicmidget, magicmidget: client_facing_magicmidget }
    - { role: magicmidget, magicmidget: internal_facing_magicmidget }
```

We have a parameterized role we can repeatedly use over and over within our network at this point.

## Configuration
We've looked at how to correctly define and write a role's structure, but it's very likely you'll need to override the configuration in the role's `defaults/main.yml` file and replace it with your own, suited to your environment. Keeping this configuration in a separate repository to your Ansible code is important for two primary reasons:

- Security: keeping your configuration separate from your code means you can hide one, but share the other;
- Sharing: and you can share your code, knowing it run well, is high quality, and others can use it, contribute it, and so forth;

These two key factors mean we separate out our configuration into a separate repository.

### Structure
Below is an example of a simple structure you can use in configuration repository:

```
.
├── README.md
├── ansible.cfg
├── checks
├── environments
├── files
├── group_vars
├── inventories
├── services
├── templates
└── users
```

The reason the `ansible.cfg` file is at this location is simple: we executed our `ansible-playbook` calls from this directory, so having the configuration present in the current working directory is ideal. The `README.md` file outlines the same information as this document.

The short of it is:

- `environments/`: environment specific configuration, with per-environment sub-directories for further refinement;
- `files/`: for static files, such as SQL dumps to be imported or just configuration files, that don't require templating or dynamic content;
- `group_vars/`: for global configuration applicable or to be made available to all environments;
- `inventories/`: literally for Ansible inventory files, which are used by the `world.yml` to build environments and also by Ansible its self to find systems;
- `templates/`: the same as files, but for dynamic files such as configuration files that are environment sensitive;

All other directories are either self-explanatory or for undocumented use.
