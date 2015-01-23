Sgoettschkes/workshopAnsible
============================

This is the git repository for my ansible workshop. Each step of the workshop will be marked with a git tag and instructions within this file.

## Step 1: The setup

You should have Vagrant, Virtualbox and ansible installed. If so, please run `vagrant up` to boot all three boxes. Afterwards, the command `ansible -i hosts all -m ping` shows that all three hosts are available. Awesome, let's move on!

## Step 2: The first task

To get our feets wet, let's update the apt cache on all three machines. Usually, we'd need to ssh into each and run `sudo apt-get update`. But not with ansible! Run `ansible-playbook -i hosts site.yml` and you are done. The site.yml contains the instruction to use the `apt` module to take care of the apt cache updating.

## Step 3: Splitting by group

As webservers and dbservers are different and need seperate steps, we should split the provisioning up! This is why we create a `webservers.yml` and a `dbservers.yml` which can hold specific tasks for each group. Run `ansible-playbook -i hosts site.yml` again. The result is the same as before, but notice how the webservers get executed first and the dbserver afterwards.

## Step 4: Introducing roles

This is nice, but what about reusing tasks? What about repeating common things over various groups? Fear not; There are roles which you can use to bundle tasks (and other things, as we'll see later)! So, we are creating a role "common" (see the `roles/` folder) which has one task, our well known apt cache update. Run `ansible-playbook -i hosts site.yml` once again to find the same result. The output is a bit different as this time, each task execution has the role name written as well. Neat!

## Step 5: We are getting there

If you install nginx, most likely you want to remove the default virtualhost. So let's remove the file and the symlink placed at `/etc/nginx/sites-available/default` and `/etc/nginx/sites-enabled/default`. We are using a new syntax `with_items` for this task. It's like a foreach loop. The `{{ item }}` is replaced with the actual item from the `with_items` list. You'll also notice the `notify` syntax. With them you can instruct ansible to run a handler with this name each time it executes the task. A handler is defined in the `handlers` folder within a role. Run `ansible-playbook -i hosts site.yml` to check it out.
