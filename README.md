# nfs_ansible_playground_20221107

NFS / NFS over TLS (stunnel) のおためし


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

- nfs-kernel-server 1.3.4-2.5ubuntu3.4: <https://packages.ubuntu.com/focal/nfs-kernel-server>
- stunnel4 5.56-1: <https://packages.ubuntu.com/focal/stunnel4>


## 参考

- <https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04-ja>
- <https://developer.hashicorp.com/vagrant/docs/multi-machine>
- <https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html>

TLS

- <https://www.linuxjournal.com/content/encrypting-nfsv4-stunnel-tls>
- <https://qiita.com/tmiki/items/626d0932f413fdc7d7fe>
- <https://www.stunnel.org/static/stunnel.html>
- <https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html>
- <https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_csr_module.html>
- <https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html>
- <https://stackoverflow.com/questions/35984151/how-to-create-new-system-service-by-ansible-playbook>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html>
- <https://www.tohoho-web.com/ex/openssl.html>
- <https://qiita.com/kunichiko/items/12cbccaadcbf41c72735>
- <https://stackoverflow.com/questions/58839349/how-to-extract-crt-and-key-from-pem-file-on-ansible>
- <https://stackoverflow.com/questions/991758/how-to-get-pem-file-from-key-and-crt-files>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html>
- <https://qiita.com/Gin/items/58df302714f9a28048c5>
- <http://blog.livedoor.jp/cadmean/archives/23581393.html>
- <http://blog.livedoor.jp/cadmean/archives/23650174.html>
- <https://etherarp.net/securing-connections-with-stunnel/index.html>


## このリポジトリにおけるNFSディレクトリの設定

- /nfs/general: 所有者が`nobody:nogroup`に固定
  - クライアントのrootユーザがファイルを作成しても、所有者はサーバ側の`nobody:nogroup`になる
- /nfs/home: 所有者をクライアントが変更できる（`no_root_squash`）
  - クライアントのrootユーザがファイルを作成すると、所有者はサーバ側の`root:root`になる


## このリポジトリの使い方

3種類のNFS環境をそれぞれディレクトリに分けて用意しています。
それぞれのディレクトリに入って、実行例のコマンドを実行したり、パケットを監視したりする使い方を想定しています。

TODO: パケット監視の方法について追記

### plain_text: 平文で通信するNFS

通信路およびすべてのホストが信頼できるプライベートネットワークやVPNでは、最もオーバーヘッドの少ないこの方式でよいと思われます。

### tls_server_auth: サーバー認証によるTLSで通信するNFS

※ 著者はセキュリティやNFSサービスの専門家ではないため、実際にはなにか有用な利用法があるかもしれません。

通信路は信頼できないが、任意のホストの接続は許すという中途半端な想定であり、読み取り専用パブリックアクセスの提供など、特別なケース以外では、この方式をとるべきではないように思われます。

### tls_server_auth_and_client_auth: サーバー認証およびクライアント認証によるTLS（mTLS）で通信するNFS

ゼロトラストネットワークでは、通信路を暗号化し、限られたホストの接続のみを許す、この方式をとるべきでしょう。


## 2022年11月に公開されたOpenSSL v3.0.0～v3.0.6の脆弱性（CVE-2022-3602, CVE-2022-3786）について

Ubuntu 22.04など、OpenSSL 3系に依存したstunnelを使用している環境に応用した場合、これらの脆弱性の影響を受ける可能性があります。
OpenSSLを、対策されたv3.0.7以降に相当するバージョンに更新しましょう。

- <https://www.jpcert.or.jp/at/2022/at220030.html>
- <https://www.ipa.go.jp/security/ciadr/vul/alert20221102.html>

## 実行例（共通）

作業ディレクトリを各環境のディレクトリに変更してから、実行してください。

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

### 秘密鍵・証明書入りPEMファイルを一括で作成

- <https://www.linuxjournal.com/content/encrypting-nfsv4-stunnel-tls>

```shell
openssl req -newkey rsa:4096 -x509 -days 3650 -nodes -out nfs-tls-server.pem -keyout nfs-tls-server.pem

openssl req -newkey rsa:4096 -x509 -days 3650 -nodes -out nfs-tls-client.pem -keyout nfs-tls-client.pem
```

### 秘密鍵、証明書署名要求、証明書をそれぞれ作成

- <https://www.tohoho-web.com/ex/openssl.html>

```shell
# 秘密鍵の生成
openssl genrsa 4096 > nfs-tls-server.key

# 証明書署名要求の作成
openssl req -new -key nfs-tls-server.key > nfs-tls-server.csr

# 証明書の作成
cat nfs-tls-server.csr | openssl x509 -req -days 3650 -signkey nfs-tls-server.key > nfs-tls-server.crt

# ---

openssl genrsa 4096 > nfs-tls-client.key
openssl req -new -key nfs-tls-client.key > nfs-tls-client.csr
cat nfs-tls-client.csr | openssl x509 -req -days 3650 -signkey nfs-tls-client.key > nfs-tls-client.crt
```

