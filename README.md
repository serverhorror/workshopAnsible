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

## Step 6: Templates and variables

Now it's time to introduce templates. They can be placed anywhere on the host and the variables inside are replaced by ansible with the value set for one host or one group. In our case, we'll place the gninx.conf file onto the webservers and the variable within is replaced with the value from `group_vars/webservers`. Variables can be placed in many places, but that's a story for another time. Run our command (you should know it by now!) and you'll be able to connect to [http://192.168.33.150](http://192.168.33.150) in your browser and see a response.

## Step 7: register, ignore_errors and when

Our webserver should run PHP code. So let's add a new role with tasks that install PHP and composer (the PHP package manager). PHP installation through apt is straightforward and nothing new for you. But look at those tasks to install composer. The first one is not even installing composer but just checking if it's already there (using `which composer.phar`). The register syntax is used to store the result of a task in a variable. `ignore_errors` means that if this command fails, nothing bad happens. The ansible run is not stopped and the task will be marked as "ok" internally. And finally there is `changed_when` which can be used to change the way ansible detects if a task changed anything. The registered variable is used in the next task with the `when` syntax. It means the task is only executed if the `when` statement evaluates to true. So, if `which composer.phar` fails, we install composer.

## Step 8: Know your modules

Do you know that you can install composer with 2 lines instead of ~10 as we just did? The `shell` module we used to install composer takes an option `creates`. You can specify a filesystem path. If it exists, the task won't run.

So why did I not tell you this at first? For one, I wanted to show you how to work with `register` and `when`. And I also wanted to make a point to learn about the different modules. For every task, look at the modules available (see [http://docs.ansible.com/list_of_all_modules.html](http://docs.ansible.com/list_of_all_modules.html)). Many common tasks are already covered. Also have a look at the options for the modules you are using, because they can make the difference between 10 ugly lines and 2 pretty clear lines of yaml code.

## Step 9: Get the facts straight!

Working with information from a remote host to set it up is not something that is uncommon. You may have a heterogeneous setup with different operating systems or different hardware. Running specific commands only on specific systems can be tricky. Luckily, Ansible got us covered: Through so-called "Facts" it registers variables holding information about the system the command is run. We can use this in combination with `when` or in templates.

Before we get to use facts, let's look at the `hosts` file. We now use an alias for each system. Our "common" role has a new file, `tasks/network.yml`. Ansible always uses the `tasks/main.yml` when executing tasks, so adding an `include` line there does the trick of also executing the tasks in the `network.yml`. In this file, we first set the hostname to the variable `inventory_hostname`. It contains the alias from our hosts file. The other tasks replaces each line that matches the regex we specified with the content we specified. It ensures that our current hostname is known to the system and maps to our localhost.

The third task shows how powerful Ansible is when different features get combined. We are again working on the hosts file. This time, we want to make sure that each host knows each other by his hostname. You see `with_items` not being a list of key/value pairs but rather a dictionary. Each dictionary contains the ip for each host from the `hosts` file and the hostname which was extracted from the Facts.

## Step 10: More about roles

A role can be made highly configurable and therefor shareable with some easy additions. Look at the nginx role. Notice the additional directories that where added. The first one is called `defaults` and the `main.yml` in it holds the default values for variables. These defaults can be overwritten by defining the same variable name somewhere else. The second directory is `files`, which is nearly the same as templates only that files in here are not run through the Jinja2 templating enginge and are therefor not changed. The `meta` directory holds various information about a role. Almost all of them are only needed if you wanna share the role on [Ansible Galaxy](https://galaxy.ansible.com/). I only added the `dependencies` key which can be used to define dependencies of this role. The roles it depends on are then run before any task of this particular role.

The last directory is `vars`, which can be used to store variables **Which should not be overwritten**. So, `defaults` holds variables that can be overwritten, vars holds the ones which cannot be overwritten outside of the role. They can be useful to hold defaults which are then overwritten by tasks inside the role. Or to indicate to people copying the role how they can change certain behaviour.
