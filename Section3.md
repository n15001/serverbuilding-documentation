Section 3 Ansibleによる自動化とテスト
======
https://github.com/cloneko/serverbuilding/blob/master/Section3.md

3-0 Ansibleのインストール
-----
```
~ ❯❯❯ sudo apt install ansible
~ ❯❯❯ ansible --version                                                                                           ⏎
ansible 2.0.0.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
```
##Ansibleのテスト  
カレントディレクトリにhostsファイルを作って、接続先にsection2で作ったサーバのIPを書く。  
`echo 192.168.56.130 > hosts`
```
~ ❯❯❯ ansible -i hosts 192.168.56.130 -m ping
192.168.56.130 | UNREACHABLE! => {
    "changed": false, 
    "msg": "ERROR! SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue", 
    "unreachable": true
}
```
鍵認証にするみたい  
http://qiita.com/HamaTech/items/21bb9761f326c4d4aa65
```
~ ❯❯❯ sssh-keygen -t rsa  
~ ❯❯❯ scat ~/.ssh/id_rsa.pub | ssh vagrant@192.168.56.130 'cat >> .ssh/authorized_keys'  
~ ❯❯❯ sssh vagrant@192.168.56.130  
[vagrant@localhost ~]$ sudo vi /etc/ssh/sshd_config  
PasswordAuthentication yes を noに変更  
[vagrant@localhost ~]$ systemctl restart sshd

~ ❯❯❯ ansible -i hosts 192.168.56.130 -m ping -u vagrant --private-key="~/.ssh/id_rsa"
192.168.56.130 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
いちいちオプション付けるの面倒なので、指定しておく。
```
~ ❯❯❯ sudo vi ~/.ssh/config #追記する
Host 192.168.56.*
  User vagrant
  IdentityFile /home/n15001/.ssh/id_rsa
  IdentitiesOnly yes

~ ❯❯❯ ansible -i hosts 192.168.56.130 -m ping                                                                         ⏎
192.168.56.130 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
ansibleでパッケージのインストールができた
```
~ ❯❯❯ ansible -i hosts 192.168.56.130 -m yum -s -a name=telnet 
192.168.56.130 | SUCCESS => {
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "読み込んだプラグイン:fastestmirror\nLoading mirror speeds from cached hostfile\n * base: ftp.iij.ad.jp\n * extras: download.nus.edu.sg\n * updates: ftp.iij.ad.jp\n依存性の解決をしています\n--> トランザクションの確認を実行しています。\n---> パッケージ telnet.x86_64 1:0.17-59.el7 を インストール\n--> 依存性解決を終了しました。\n\n依存性を解決しました\n\n================================================================================\n Package          アーキテクチャー バージョン              リポジトリー    容量\n================================================================================\nインストール中:\n telnet           x86_64           1:0.17-59.el7           base            63 k\n\nトランザクションの要約\n================================================================================\nインストール  1 パッケージ\n\n総ダウンロード容量: 63 k\nインストール容量: 113 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  インストール中          : 1:telnet-0.17-59.el7.x86_64                     1/1 \n  検証中                  : 1:telnet-0.17-59.el7.x86_64                     1/1 \n\nインストール:\n  telnet.x86_64 1:0.17-59.el7                                                   \n\n完了しました!\n"
    ]
}
```
3-1 ansibleでWordpressを動かす(2)を行なう
-----
vagrantってデフォルトで鍵作るのね
ansible用に新しく立ち上げる
```
~/k/ansible-server ❯❯❯ vagrant init
~/k/ansible-server ❯❯❯ vagrant up
~/k/ansible-server ❯❯❯ vagrant ssh
[vagrant@localhost ~]$ sudo chmod 600 ~/.ssh/authorized_keys
~/k/ansible-server ❯❯❯ echo 192.168.56.131 >> ~/hosts
~/k/ansible-server ❯❯❯ sudo etc/ansible/ansible.cfg
[defaults]
hostfile = /home/n15001/hosts
remote_user = vagrant
private_key_file=/home/n15001/.vagrant.d/insecure_private_key
~/k/ansible-server ❯❯❯ ansible 192.168.56.131 -m ping                                                       ⏎
192.168.56.131 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
```
~ ❯❯❯ sudo nano ansible-wpingyml                                                                              ⏎
---
- hosts: 192.168.56.131
  remote_user: vagrant
  tasks:
   - name: pingしてみる #taskの名前
     ping:
~ ❯❯❯ ansible-playbook ansible-wp.yml --check
PLAY ***************************************************************************
TASK [setup] *******************************************************************
ok: [192.168.56.131]
TASK [pingしてみる] ****************************************************************
ok: [192.168.56.131]
PLAY RECAP *********************************************************************
```

**Playbook**：https://github.com/n15001/serverbuilding-documentation/blob/master/ansible-wp.yml

インストール済みかどうかで条件分岐や、nginx設定ファイルの書き換えをどうしようか

```
~ ❯❯❯ sudo ansible-playbook ansible-wp.yml --ask-sudo-pass
SUDO password: 
[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and make sure become_method is 'sudo' (default). 
This feature will be removed in a future release. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ok: [192.168.56.132]

TASK [pingてすと] *****************************************************************
ok: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [yum.conf proxy] **********************************************************
changed: [192.168.56.132]

TASK [nginx repo] **************************************************************
changed: [192.168.56.132]

TASK [nginx install] ***********************************************************
changed: [192.168.56.132]

TASK [php install] *************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [phpの設定ファイルかきかえ] **********************************************************
changed: [192.168.56.132] => (item={u'regexp': u'user = apache', u'line': u'user = nginx'})
changed: [192.168.56.132] => (item={u'regexp': u'group = apache', u'line': u'group = nginx'})

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [nginx設定ファイルかきかえ] *********************************************************
changed: [192.168.56.132]
 [WARNING]: Consider using template or lineinfile module rather than running sed

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [command] *****************************************************************
changed: [192.168.56.132]

TASK [WordPress ダウンロード] ********************************************************
changed: [192.168.56.132]
 [WARNING]: Consider using get_url module rather than running wget


TASK [wordpress 解凍] ************************************************************
changed: [192.168.56.132]
 [WARNING]: Consider using unarchive module rather than running tar

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [MariaDB を常時起動にする (サービスを有効化)] *********************************************
changed: [192.168.56.132]

TASK [nginxを常時起動にする (サービスを有効化)] ************************************************
changed: [192.168.56.132]

TASK [php-fpm を常時起動にする (サービスを有効化)] *********************************************
changed: [192.168.56.132]

TASK [wordpress用のデータベース作成] ***************************************
changed: [192.168.56.132]

TASK [データベース弄れるユーザ作成] **************************************
changed: [192.168.56.132]

TASK [wp-config-sampleをコピー] ****************************************************
changed: [192.168.56.132]

TASK [wp-configの書き換え] **********************************************************
changed: [192.168.56.132]

PLAY RECAP *********************************************************************
192.168.56.132             : ok=25   changed=23   unreachable=0    failed=0   
```
なんとか自動化はできた、コマンド叩いてから150秒くらいで終了する。  
`http://サーバーのIP/wordpress` にアクセスすればwordpressの設定画面が出てくる。
![ansibleで自動化したやつ](https://raw.githubusercontent.com/n15001/serverbuilding-documentation/master/Screenshot%20from%202016-05-31%2014-27-07.png "ansibleで自動化したやつ")

3-1-2? VagrantfileからAnsibleを呼び出す
-----
```
~/k/ansible-test ❯❯❯ sudo nano Vagrantfile    #追記する                                                                 ⏎
config.vm.provision "ansible" do |ansible|
    ansible.playbook = "/home/n15001/ansible-wp.yml"
    ansible.inventory_path = "/home/n15001/hosts"
    ansible.limit = 'all'
  end

~/k/ansible-test ❯❯❯ sudo vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'CentOS7'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: ansible-test_default_1464676139548_28147
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Remote connection disconnect. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
==> default: Machine booted and ready!
GuestAdditions 5.0.20 running --- OK.
==> default: Checking for guest additions in VM...
==> default: Configuring and enabling network interfaces...
==> default: Mounting shared folders...
    default: /vagrant => /home/n15001/kaihatu/ansible-test
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and 
make sure become_method is 'sudo' (default). This feature will be removed in a 
future release. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ok: [192.168.56.132]

TASK [pingてすと] *****************************************************************
ok: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [yum.conf proxy] **********************************************************
changed: [192.168.56.132]

TASK [nginx repo] **************************************************************
changed: [192.168.56.132]

TASK [nginx install] ***********************************************************
changed: [192.168.56.132]

TASK [php install] *************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [phpの設定ファイルかきかえ] **********************************************************
changed: [192.168.56.132] => (item={u'regexp': u'user = apache', u'line': u'user = nginx'})
changed: [192.168.56.132] => (item={u'regexp': u'group = apache', u'line': u'group = nginx'})

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [nginx設定ファイルかきかえ] *********************************************************
changed: [192.168.56.132]
 [WARNING]: Consider using template or lineinfile module rather than running
sed

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
changed: [192.168.56.132]

TASK [command] *****************************************************************
changed: [192.168.56.132]

TASK [WordPress ダウンロード] ********************************************************
changed: [192.168.56.132]
 [WARNING]: Consider using get_url module rather than running wget

TASK [wordpress 解凍] ************************************************************
changed: [192.168.56.132]

TASK [file] ********************************************************************
 [WARNING]: Consider using unarchive module rather than running tar

changed: [192.168.56.132]

TASK [MariaDB を常時起動にする (サービスを有効化)] *********************************************
changed: [192.168.56.132]

TASK [nginxを常時起動にする (サービスを有効化)] ************************************************
changed: [192.168.56.132]

TASK [php-fpm を常時起動にする (サービスを有効化)] *********************************************
changed: [192.168.56.132]

TASK [wordpress用のデータベース作成] *****************************************************
changed: [192.168.56.132]

TASK [データベース弄れるユーザ作成] **********************************************************
changed: [192.168.56.132]

TASK [wp-config-sampleをコピー] ****************************************************
changed: [192.168.56.132]

TASK [wp-configの書き換え] **********************************************************
changed: [192.168.56.132]

PLAY RECAP *********************************************************************
192.168.56.132             : ok=25   changed=23   unreachable=0    failed=0   
```
