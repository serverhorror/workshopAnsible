Sgoettschkes/workshopAnsible
============================

This is the git repository for my ansible workshop. Each step of the workshop will be marked with a git tag and instructions within this file.

## Step 1: The setup

You should have Vagrant, Virtualbox and ansible installed. If so, please run `vagrant up` to boot all three boxes. Afterwards, the command `ansible -i hosts all -m ping` shows that all three hosts are available. Awesome, let's move on!

## Step 2: The first task

To get our feets wet, let's update the apt cache on all three machines. Usually, we'd need to ssh into each and run `sudo apt-get update`. But not with ansible! Run `ansible-playbook -i hosts site.yml` and you are done. The site.yml contains the instruction to use the `apt` module to take care of the apt cache updating.
