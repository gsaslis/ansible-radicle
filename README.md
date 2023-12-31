# An Ansible Playbook to Deploy Radicle

This directory provides the necessary steps that deploy a **Public** Radicle Node (together with its accompanying HTTP API), using Ansible, and running the necessary system services suitable for an "always on" deployment of Radicle.

## Requirements

 
1. Setup Ansible on your machine: https://docs.ansible.com/ansible/latest/intro_installation.html 
1. Install python requirements (consider using bundled `extensions/setup/setup.sh`)
1. Update roles (consider using `extensions/setup/role_update.sh`)
1. Create ssh keys for access to your VM
```bash
$ ssh-keygen -t ed25519 -f <path_to_private_key> -N <path_to_public_key>
```
5. Spin up a VM, ensuring you set up SSH access with the key you created above.
5. If you want a nice domain for your node, now is a good time to add an 'A' record pointing to the static IP of your VM, in your domain's DNS settings.  
5. With all that in hand, you can now create necessary files: 

```
# <project_path>/production.ini
radicle1 ansible_ssh_host=$host_or_ip ansible_ssh_port=$ssh_port ansible_ssh_user=$ssh_user ansible_ssh_private_key_file=<path_to_private_key> (from step 3)

[radicle]
radicle1


[all:vars]
environment=prod
ansible_python_interpreter=/usr/bin/python3
```


```yaml
# <project_path>/vars/secret_vars.yaml
rad_passphrase: "never_commit_sensitive_values"
rad_domain: your.radicle.domain.here
rad_node_alias: your.radicle.domain.here
concourse_domain: your.concourse.domain.here
concourse_port: 8080
concourse_postgres_user: your_db_username
concourse_postgres_password: your_db_password
concourse_user: your_username
concourse_password: your_password
# generate below with e.g. `openssl rand -hex 32`
concourse_client_secret: some_long_secret
radicle_node_port: 7777
radicle_httpd_port: 8888
# the options to pass to the `radicle-node` binary, to define behaviour of your seed node, e.g.
radicle_node_options: --force
ssh_public_key: <the_path_to_the_public_key_that_should_be_added_to_the_authorized_keys_of_the_newly_created_user> # `<path_to_public_key> from step 4
firewall_allowed_tcp_ports:
  - "22" # (ssh port - better use another value) 
  - "7777" # (radicle-node port)
  - "8888" # (radicle-httpd port) 
security_ssh_port: 22 # (ssh port)
```

## Install Radicle Services


Run the radicle-docker-ansible playbook, like so: 

```bash
cd ansible/plays
ansible-playbook --ask-vault-pass -i ../production.ini 0_setup.yml
ansible-playbook --ask-vault-pass -i ../production.ini 1_hardening.yml
ansible-playbook --ask-vault-pass -i ../production.ini 2_radicle.yml
```

## Upgrade your Node

The radicle playbook will back up your `.radicle` storage folder and 
will install the latest binaries available. 

```bash
cd ansible/plays
ansible-playbook --ask-vault-pass -i ../production.ini 2_radicle.yml
```


That's it!!   


Enjoy!!! 


