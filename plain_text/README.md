# NFS over plain text connection

## VMs

|Name|IP address|
|:--|:--|
|NFS Server|192.168.56.10|
|NFS Client|192.168.56.11|

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

Then, initialize the VMs with the Ansible Playbook.

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

### Find all VirtualBox VMs and remove them

`.vagrant`ディレクトリの削除などの要因で、VMが起動した状態のまま、VagrantがVMを追跡できなくなることがある。
この状態で新たなVMが起動すると、Ansibleは古いVMを初期化するが、Vagrantは新たなVMに接続する、のような意図しない挙動になるおそれがある。
このような場合、VirtualBoxの管理コマンドで直接削除する。

```shell
VBoxManage list vms

VBoxManage controlvm VM_NAME poweroff
VBoxManage unregistervm VM_NAME
```

他に重要なVMがなければ、以下のエイリアスで、VirtualBox上のすべてのVMを削除できる（意図しないデータ喪失に注意）。

- <https://github.com/hashicorp/vagrant/issues/910#issuecomment-16026322>

```shell
alias shutdown-vms="VBoxManage list vms | cut -f 1 -d ' ' | xargs -I NAME sh -c 'VBoxManage controlvm NAME poweroff ; VBoxManage unregistervm NAME' ; rm -rf ~/VirtualBox\ VMs/*"

shutdown-vms
```
