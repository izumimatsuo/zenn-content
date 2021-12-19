---
title: "VagrantとAnsibleでつくった仮想サーバーをTestinfraで検証する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['vagrant','ansible','testinfra']
published: true
---
こんにちは。

VagrantとAnsibleで仮想サーバを構築した際に、想定したとおりに構築できたかを検証したいですよね。その際、難しい設定が必要だったり長いコマンドを都度で入力するのは面倒です。できれば簡単に検証を実施したいです。

この記事では、Ansibleと連携できるTestinfraを使って簡単にVagrantで構築した仮想サーバーの検証をおこなう方法について説明していきます。

# 前提

この記事の内容は、次の環境で動作を確認しています。

- MacBook Air（13-inch, 2017）
- macOS 11.6
- Vagrant 2.2.18
- VirtualBox 6.1.28
- Python 3.9.8
- Ansible 5.0.1
- Ansible-core 2.12.1
- Testinfra 6.5.0

VagrantやVirtualBoxのインストールについては、次の記事が参考になります。

https://qiita.com/tsunemiso/items/d184366b8926bd5a8d00
https://qiita.com/OPySPGcLYpJE0Tc/items/3268aa09c16a25cded0f

また、AnsibleとTestinfraの実行環境は、Pythonのvenvモジュールを利用しています。次の記事が参考になります。

https://maku77.github.io/python/env/venv.html

# Ansibleの設定ファイルを用意する

Testinfraは、Ansibleの設定ファイルを使って連携させると便利ですので、次のような設定ファイルを用意します。

```ini
$ cat ansible.cfg
[defaults]
inventory = .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory

[ssh_connection]
ssh_args = -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o ControlMaster=auto -o ControlPersist=3600s
```

Ansibleプロビジョニングを実行すると、Vagrantは自動でAnsible用のインベントリを作成します。これをTestinfraでも利用できるように、``[defaults]``セクションで設定します。

また、sshの初回接続で対話モードとならないようにします。具体的には、``UserKnownHostsFile=/dev/null``と``StrictHostKeyChecking=no``を``[ssh_connection]``セクションに設定します。

ちなみに、``ControlMaster=auto``と``ControlPersist=3600s``は、Ansibleを効率良く実行するための設定となります。

# Testinfraを実行してみる

簡単なテストケースを用意します。

```python
$ cat test_playbook.py
def test_distribution(host):
    assert 'centos' == host.system_info.distribution
```

テスト対象なる仮想サーバーは、次のVagrantfileとAnsible Playbookでプロビジョニングします。詳しくは、別の記事で説明した「[VagrantのAnsibleプロビジョニングでPlaybookを柔軟に変更できるVagrantfileの書き方](https://zenn.dev/izumimatsuo/articles/2021-12-12-vagrant-provision-ansible)」を参考にしてください。

``` ruby
$ cat Vagrantfile
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

```
$ cat playbook.yml 
---
- name: setup virtual machine
  hosts: all
  tasks:
    - debug:
        var: ansible_eth1.ipv4.address
```

ここまでで、次のようなファイルが用意できています。

```
$ tree
.
├── Vagrantfile
├── ansible.cfg
├── playbook.yml
└── test_playbook.py
```

次に、AnsibleとTestinfraの実行環境を準備します。

```
$ python3 -m venv .venv
$ source .venv/bin/activate

$ pip install ansible pytest-testinfra
```

環境変数を設定してVagrantで仮想サーバーの起動とAnsibleプロビジョニングを実行します。

```
$ export ANSIBLE_PLAYBOOK='playbook.yml'

$ vagrant up
Bringing machine 'host1' up with 'virtualbox' provider...

中略

==> host1: Running provisioner: ansible...
    host1: Running ansible-playbook...

PLAY [setup virtual machine] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [debug] *******************************************************************
ok: [host1] => {
    "ansible_eth1.ipv4.address": "192.168.56.11"
}

PLAY RECAP *********************************************************************
host1                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

最後にTestinfraを実行してみます。

```
$ pytest -v --hosts='ansible://all' test_playbook.py 
============================================== test session starts =========================================================

中略

test_playbook.py::test_distribution[ansible://host1] PASSED                                                           [100%]
============================================== 1 passed in 2.12s ===========================================================
```

問題無いことを検証できました。

対象となる複数の仮想サーバーとのssh接続設定を都度用意しなくてもテストを実行できるのは便利です。

# まとめ

VagrantとAnsibleプロビジョニングで構築した仮想サーバーをTestinfraで簡単に検証する方法を紹介しました。インフラの勉強やテストを少しでも効率よく実施できる一助になれば幸いです。

## 参考

https://koh-sh.hatenablog.com/entry/2019/10/06/012243
https://testinfra.readthedocs.io/en/latest/
