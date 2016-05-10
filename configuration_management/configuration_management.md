# Configuration Management (CM)
The art of configuring and managing and entire network is something to be learned and developed over a long period of time. It's a skill to be able to manage a network manually, making changes to individual files and updating scripts and bits of code holding everything together. I do admire the system administrator who can navigate their vast nest of cronjobs which reboot services at 0300 hours due to memory leaks, not to mention print their SSH logs out onto a dot matrix printer for "redundancy."

CM replaces your shell scripts in for-loops with an actual solution that is idempotent and safe. Sure it might not be as flexible as a shell script, what is, but it's most certainly a better option.

CM is the art of writing code to explain the state in which you want your network, and its individual components, to be in. It's a way of changing a single line in a single file, and having 1,000 VMs updated in a minutes. All the while that very same code is version controlled and being tested in a CI environment when changes are made. It's time to stop breaking stuff.

## Ansible
This is where Ansible comes in. It's going to be our CM tool of choice, and you'll see why in the next section.

