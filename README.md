# Ansible Playbook for Kubernetes 

## Add ansible inventory
This is set in the _ansible_host_. I set the hosts in `/etc/hosts`. All my hosts are on the same subnet.
``` hosts-file
# Static table lookup for hostnames.
# See hosts(5) for details.

10.80.1.98     kubenode1.local kubenode1
10.80.1.97     kubenode2.local kubenode2
10.80.1.96     kubenode3.local kubenode3
10.80.1.95     kubenode4.local kubenode4
```

And include them in the `hosts.yml`. The location of the file is configured in the `ansible.cfg` file.

``` ini
[defaults]
inventory=./hosts.yml
```

```yml
all:
  children:
    nodes:
      hosts:
        kubenode1:
          ansible_user: kubedeveloper
          apiserver_address: 10.80.1.98
        kubenode2:
          ansible_user: kubedeveloper
        kubenode3:
          ansible_user: kubedeveloper
        kubenode4:
          ansible_user: kubedeveloper
    master:
      hosts:
        kubenode1:
```

If for example you want to set the IP in the `hosts.yml` file, set it like this:

```yml
all:
  children:
    nodes:
      hosts:
        kubenode1:
          ansible_host: 10.80.1.98
          ansible_user: kubedeveloper
          apiserver_address: 10.80.1.98
        kubenode2:
          ansible_host: 10.80.1.97
          ansible_user: kubedeveloper
        kubenode3:
          ansible_host: 10.80.1.96
          ansible_user: kubedeveloper
        kubenode4:
          ansible_host: 10.80.1.95
          ansible_user: kubedeveloper
    master:
      hosts:
        kubenode1:
```

And if you're using one of the nodes as the _ansible_host_, set the `ansible_connection` as local:

```yml
all:
  children:
    nodes:
      hosts:
        kubenode1:
          ansible_connection: local
          ansible_user: kubedeveloper
          apiserver_address: 10.80.1.98
        kubenode2:
          ansible_host: 10.80.1.97
          ansible_user: kubedeveloper
        kubenode3:
          ansible_host: 10.80.1.96
          ansible_user: kubedeveloper
        kubenode4:
          ansible_host: 10.80.1.95
          ansible_user: kubedeveloper
    master:
      hosts:
        kubenode1:
```
## Passwordless SSH access.
This is for easier access from the _ansible host_ to each _nodes_.

### Generate SSH keypair
Optional, only if you don't have a key already.

### Copy the SSH key from the ansible_hosts to each nodes

``` shell
ssh-copy-id <username>@<node_ip>
```

### Test ansible connection to nodes

This should be run at the root of this repository (at the location of the `ansible.cfg` file).

``` shell
ansible -m ping all
```

Make sure this command is able to ping all the nodes before running this playbook.

## Storing Password
Define the text file containing the password. For example `.secret` inside `ansible.cfg`, for example:

```ini
[defaults]
vault_password_file=.secret
```

Ansible will search for password inside this file, this password is used for accessing the (ansible vault)[https://docs.ansible.com/ansible/latest/vault_guide/index.html]

### Storing each nodes become_password
In `Ansible`, the variable `ansible_become_password` is used to store the `sudo` password. Since there's a possibility that each nodes have different password, it is better to store all the password in one file, and encrypt it, see (the official guide for building your inventory)[https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html].

#### In case all your nodes using the same password

##### Create the directory
Since I'm not grouping my nodes in a specific directory (automatically under `all` group), we'll create a `group_vars/all` directory. create two files named `vars` and `vault`.

The `vars` file:

```yml
ansible_become_pass: "{{ vault_ansible_become_pass }}"
```

And the `vault` file:
```yml
vault_ansible_become_pass: <the-actual-sudo-password>
```

And then encrypt the `vault` file using:

```shell
ansible-vault encrypt group_vars/all/vars
```
