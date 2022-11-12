## Usage

### Initialize the VMs

Generate a ssh key pair (without password) to sign in the VMs.

```shell
mkdir -p secrets
ssh-keygen -f secrets/ssh_key -P ""
```

Create the VMs with the Vagrantfile.

```shell
vagrant up
```

Initialize the VMs with the Ansible Playbook.

```shell
ansible-playbook -i inventory.yaml playbook.yaml
```

### Create a file in the NFS directory on the server

Open a ssh connection to the server VM.

```shell
vagrant ssh server
```

Create a file in the NFS directory.

```shell
cd /var/nfs/general

sudo touch hoge
```

### Check the file in the NFS directory on the client

Open a ssh connection to the client VM.

```shell
vagrant ssh client
```

Go to the NFS directory and you will see the shared file `hoge`.

```shell
cd /nfs/general

ls -la
```

### Remove the VMs for cleaning up

```shell
vagrant destroy -f
```
