# ansible-
###Playing with Ansible &amp; some configurations / boilerplate


## 🍀 First things first
- Configure `SSH keys` 
- Install ssh key pair -> `public and private`
- Keep the __private__ on the workstation that the Ansible will use to ssh to other servers
- `Dump/ copy` the **public key** to all the other servers to which the main ansible machine is going to talk to.

## 💾 Inventory file
- Add all the `IP` or `Hostnames` of the server you want ansible to connect to and work with

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
-  
```bash
# Run the playbook
 ansible-playbook --ask-become-pass install_apache.yml
```



