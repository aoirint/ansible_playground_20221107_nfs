# nfs_ansible_playground_20221107

NFS のおためし

## 実行環境

- Ubuntu 20.04 (Host OS)
- VirtualBox 7.0
  - <https://www.virtualbox.org/wiki/Linux_Downloads>
  - ハードウェア仮想化機能 Intel VT-x または AMD-V が有効化された環境が必要
    - KVM等による仮想環境として提供される通常のVPS等の環境、WSL等では動作不能
    - 以下のコマンドで1以上の値が出力されれば、VT-x または AMD-V がサポートされたCPUである（利用には、別途BIOSでの有効化が必要な場合もある）
      - `egrep -c '(vmx|svm)' /proc/cpuinfo`
      - <https://help.ubuntu.com/community/KVM/Installation#Pre-installation_checklist>
- Vagrant 2.3
  - <https://developer.hashicorp.com/vagrant/downloads>
- Ansible 2.12
  - <https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu>

## 何をするか

VagrantでVM（Ubuntu 20.04 with sshd）を立て、
AnsibleでNFSがインストールされたサーバ・クライアント環境を宣言し、適用する。

## 参考

- <https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04-ja>
- <https://developer.hashicorp.com/vagrant/docs/multi-machine>
- <https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html>

## 実行例

```shell
# Remove the VM's SSH server key fingerprint from the host's known_hosts (required for re-created VMs)
ssh-keygen -f "$HOME/.ssh/known_hosts" -R "192.168.56.10"
ssh-keygen -f "$HOME/.ssh/known_hosts" -R "192.168.56.11"

# Generate a ssh key pair (without password) to sign in the VMs
mkdir -p secrets
ssh-keygen -f secrets/key -P ""

# Initialize the VMs
vagrant up

# Execute the playbook
ansible-playbook -i inventory.yaml playbook.yaml

# Open a ssh connection into the server VM
vagrant ssh server

cd /var/nfs/general
sudo touch hoge

exit

# Open a ssh connection into the client VM
vagrant ssh client

cd /nfs/general

# found "hoge"
ls -la

exit

# Remove the VMs for cleaning up
vagrant destroy
```
