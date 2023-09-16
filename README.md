# ansible-
###Playing with Ansible &amp; some configurations / boilerplate


## 🍀 First things first
- Configure `SSH keys` 
- Install ssh key pair -> `public and private`
- Keep the __private__ on the workstation that the Ansible will use to ssh to other servers
- `Dump/ copy` the **public key** to all the other servers to which the main ansible machine is going to talk to.
```bash
# comand to copy ssh public key from the main workstation to the server in order for the main server to talk to the target server
ssh-copy-id -i ~/.ssh/ansible.pub 172.0.0.23 
```

## 💾 Inventory file
- Add all the `IP` or `Hostnames` of the server you want ansible to connect to and work with
- Dynamic Inventory is also supported in Ansible
- For Dynamic just get the server info from the cloud provider.
  

## ⏱️ Connection check
- Inorder to check the connection to all the servers from the main where ansible is installed 

```bash
# this will create a connection and check if everything is fine and we are able to connect
# here the path till ssh private is given and the key name is ansible (private key) & invenory contains all the server ips to which using this key the ansible main machine going to connect.
ansible all --key-file ~/.ssh/ansible -i inventory - ping

# To list all the hosts
ansible all --list-hosts

# Lists all the data/information of all the servers
# m is the module flag
ansible all -m gather_facts

# Lists all the data of a specific server
ansible all -m gather_facts --limit 172.0.0.12


```

## ⌨️ AdHoc Commands 
- Some commands which require sudo previliges to work with.
- The sudo thing can be achieved using the below syntax.
```bash
#Tell ansible to use sudo (become)
ansible all -m apt -a update_cache=true --become --ask-become-pass

#Install a package named tmux via the apt module
# Equivalent to sudo apt-get install tmux 
ansible all -m apt -a name=tmux --become --ask-become-pass

#Install a package via the apt module, and also make sure it’s the latest version available
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass

#Upgrade all the package updates that are available
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```

## :open_book: Playbooks 
- Before Ansible runs any playbooks or before running any commands on servers it will `First Gather Facts` and do the further things. 
- To disable fact gathering for a play, set the gather_facts key to no
- Write roles its like modules in Terraform 

```bash
# Run the playbook
 ansible-playbook --ask-become-pass install_apache.yml
```

## :open_book: Targeting specific Nodes
- Lets say you 3 servers one for front end, second one contains some business logic as in REST API, and last one is a DB server
- So, we would have to target specific servers for specific configurations
- Need to create groups in inventory file eg one for db_servers, file_servers, web_servers

```shell
# inventory_group file 
[web_servers]
172.16.250.132
172.16.250.248
 
[db_servers]
172.16.250.133

[file_servers]
172.16.250.134


# changes in the yml file 
---
 
 - hosts: all
   become: true
   tasks: # here we are updating all the systems before any new operations are to be performed

   - name: install updates (CentOS)
     dnf:
       update_only: yes
       update_cache: yes
     when: ansible_distribution == "CentOS"
 
   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
  # here we are targeting specific servers mentioned in the invernoty file grouping
 - hosts: web_servers
   become: true
   tasks:
 
   - name: install apache and php for Ubuntu servers
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache and php for CentOS servers
     dnf:
       name:
         - httpd
         - php
       state: latest
     when: ansible_distribution == "CentOS"

```

## :safety_pin: Tags
- Used to run `SPECIFIC TASK` in the playbook instead of running all the tasks for testing just one change in one task
- So, we dont need to run all the tasks just for one change in other task
- We `Tag` all the tasks and then based on that tag we can run that particular task and skip all the other tasks 
  
  
```bash
# List the available tags in a playbook
ansible-playbook --list-tags site_with_tags.yml

# Examples of running a playbook but targeting specific tags
ansible-playbook --tags db --ask-become-pass site_with_tags.yml
ansible-playbook --tags centos --ask-become-pass site_with_tags.yml
ansible-playbook --tags apache --ask-become-pass site_with_tags.yml
```

## :open_file_folder: File Management & downloading Binaries
- You can copy files from Local System to all the other servers 
- Also, if we want we can download binaries like Terraform, Unzip, etc on a workstation locally or other servers.
  
```shell

# Exmaple of copying file from local to all the web servers 
- hosts: web_servers
   become: true
   tasks:   
    - name: copy html file for site
        tags: apache,apache,apache2,httpd
        copy:
        src: default_site.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: 0644

# Exmaple of downloading binaries in local workstation
- hosts: workstations
   become: true
   tasks:

   - name: install unzip
     package:
       name: unzip
 
   - name: install terraform
     unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip # download link address
      dest: /usr/local/bin # here the binary will be downloaded
      remote_src: yes # to tell it is downloaded
      mode: 0755
      owner: root
      group: root


# Run the playbook 
ansible-playbook --ask-become-pass file_management.yml

```

## :magic_wand: Services
- It is a module that can be used to start, stop, restart, or reload services on remote hosts. This is useful for managing the state of services, especially when deploying updates or changes.

```bash

# changes related to start httpd automatically and also enabling it if in case the server goes down to restart httpd automatically
- name: start and enable httpd (CentOS)
     tags: apache,centos,httpd
     service:
       name: httpd
       state: started
       enabled: yes
     when: ansible_distribution == "CentOS"


# If the configuration file is changed and needs a restart so this is how it can be done 
- name: change e-mail address for admin
     tags: apache,centos,httpd
     lineinfile: #module to be used to change a line 
       path: /etc/httpd/conf/httpd.conf # path to the file which is to be changed 
       regexp: '^ServerAdmin' # line begin with ServerAdmin
       line: ServerAdmin somebody@somewhere.net #replace that with this line 
     when: ansible_distribution == "CentOS"
     register: httpd # variable declare that stores the state of httpd whether its changed or not
 
- name: restart httpd (CentOS)
     tags: apache,centos,httpd
     service:
       name: httpd
       state: restarted 
     when: httpd.changed  # this play check if changed then run this which will restart the server 

```

## Variables
- Registered Variables -> One can capture the output of a command by using the register statement.
- The output is saved into a variable that could be used later for either debugging purposes or in order to achieve something else, such as a particular configuration based on a command's output.
 
## :: Task Iteration with Loops
- Need to look into it.

## :genie_man: Ansible Handlers
- They are executed only once.
- Executed at the very end.

## :ninja: Anisble Roles
- Create Roles basically an Ansible PlayBook only
- Add all the plays(tasks) in that particular role
- Eg You can add some Bootstrapping scripts as Plays like an apt-get update 
- These types of roles can be used to set up a server which has nothing on it.
- So, this can actually download all the latest packages required by the system.








