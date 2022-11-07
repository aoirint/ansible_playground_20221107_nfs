# nfs_ansible_playground_20221107

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
