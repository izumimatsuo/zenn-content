---
title: "VagrantのAnsibleプロビジョニングでPlaybookを柔軟に変更できるVagrantfileの書き方"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['vagrant','ansible']
published: true
---
こんにちは。

Vagrantで仮想サーバーを構築するのに、Ansibleプロビジョニングをよく使います。その際、構築したい内容によってAnsible Playbookを変えたいことがあります。でも、そのためにはいちいちVagrantfileを書き変える必要があって面倒です。

この記事では、VagrantのAnsibleプロビジョニングでPlaybookを柔軟に変更して仮想サーバーを構築できるVagrantfileの書き方について説明していきます。

# 前提

この記事の内容は、次の環境で動作を確認しています。

- MacBook Air（13-inch, 2017）
- macOS 11.6
- Vagrant 2.2.18
- VirtualBox 6.1.28
- Python 3.9.8
- Ansible 5.0.1
- Ansible-core 2.12.1

VagrantやVirtualBoxのインストールについては、次の記事が参考になります。

https://qiita.com/tsunemiso/items/d184366b8926bd5a8d00
https://qiita.com/OPySPGcLYpJE0Tc/items/3268aa09c16a25cded0f

また、Ansibleの実行環境は、Pythonのvenvモジュールを利用しています。次の記事が参考になります。

https://maku77.github.io/python/env/venv.html

# Ansibleプロビジョニングで使用するPlaybookを柔軟に変更できるVagrantfileを書く

次のようなVagrantfileを書いていきます。なお、元となるVagrantfileは、別の記事で説明した「[Vagrantで複数台の仮想サーバーを柔軟に起動できるVagrantfileの書き方](https://zenn.dev/izumimatsuo/articles/2021-12-05-vagrant-multi-servers)」のものを再利用しています。

``` ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  ANSIBLE_PLAYBOOK = ENV["ANSIBLE_PLAYBOOK"]
  MAX_OF_SERVERS = (ENV["MAX_OF_SERVERS"] || 1).to_i
  (1..MAX_OF_SERVERS).each do |id|
    config.vm.define "host#{id}" do |host|
      host.vm.hostname = "host#{id}"
      host.vm.network "private_network", ip: "192.168.56.#{10+id}"

      # Ansible Provisioning
      if id == MAX_OF_SERVERS && ANSIBLE_PLAYBOOK
        host.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = ANSIBLE_PLAYBOOK
        end
      end

    end
  end

end
```

次のように、環境変数``ANSIBLE_PLAYBOOK``でPlaybookファイル名を受けとるようにします。

```ruby
  ANSIBLE_PLAYBOOK = ENV["ANSIBLE_PLAYBOOK"]
```

また、``if id == MAX_OF_SERVERS``と``ansible.limit = "all"``で複数の仮想サーバーに対して一括でAnsibleプロビジョニングするようにしています。ちなみに、環境変数``ANSIBLE_PLAYBOOK``が設定されていない場合はプロビジョニングを実行しないようにしています。

```ruby
      # Ansible Provisioning
      if id == MAX_OF_SERVERS && ANSIBLE_PLAYBOOK
        host.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = ANSIBLE_PLAYBOOK
        end
      end
```

# Ansibleプロビジョニングを実行してみる

実際に、簡単なPlaybookを使ってAnsibleプロビショニングを実行してみます。

```
$ cat playbook.yml 
---
- name: setup virtual machine
  hosts: all
  tasks:
    - debug:
        var: ansible_eth1.ipv4.address
```

まず、Ansibleの実行環境を準備します。

```
$ python3 -m venv .venv
$ source .venv/bin/activate

$ pip install ansible
```

環境変数を設定してVagrantで仮想サーバーの起動とAnsibleプロビジョニングを実行します。

- 使用するAnsible Playbookは、playbook.yml
- 起動する仮想サーバーは、2台

```
$ export ANSIBLE_PLAYBOOK='playbook.yml'
$ export MAX_OF_SERVERS=2

$ vagrant up
Bringing machine 'host1' up with 'virtualbox' provider...
Bringing machine 'host2' up with 'virtualbox' provider...

中略

==> host2: Running provisioner: ansible...
    host2: Running ansible-playbook...

PLAY [setup virtual machine] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [host2]
ok: [host1]

TASK [debug] *******************************************************************
ok: [host2] => {
    "ansible_eth1.ipv4.address": "192.168.56.12"
}
ok: [host1] => {
    "ansible_eth1.ipv4.address": "192.168.56.11"
}

PLAY RECAP *********************************************************************
host1                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host2                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

環境変数を再設定するだけで、Vagrantfileをいちいち編集しないでもPlaybookを変更してAnsibleプロビジョニングを実行できるのは便利です。

# まとめ

VagrantのAnsibleプロビジョニングでPlaybookを柔軟に変更して仮想サーバーを構築できるVagrantfileの書き方を紹介しました。インフラの勉強やテストを少しでも効率よく実施できる一助になれば幸いです。

