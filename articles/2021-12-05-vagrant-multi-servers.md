---
title: "Vagrantで複数台の仮想サーバーを柔軟に起動できるVagrantfileの書き方"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['vagrant']
published: true
---

こんにちは、はじめての記事投稿となります。

Vagrantでは、Vagrantfileを定義することで複数台の仮想サーバーを起動して、インフラの勉強をしたりテストすることができます。その際、状況によって起動する仮想サーバーの台数を変えたいことがあります。でも、そのためにはいちいちVagrantfileを書き変える必要があって面倒です。

この記事では、起動したい仮想サーバーの台数を柔軟に変更できるようにする、Vagrantfileの書き方について説明していきます。

# 前提

この記事の内容は、以下の環境で動作を確認しています。

- MacBook Air (13-inch, 2017)
- macOS 11.6
- Vagrant 2.2.18
- VirtualBox 6.1.28

また、VagrantやVirtualBoxのインストールについては、以下の記事が参考になります。

https://qiita.com/tsunemiso/items/d184366b8926bd5a8d00
https://qiita.com/OPySPGcLYpJE0Tc/items/3268aa09c16a25cded0f

# Vagrantfileに複数の仮想マシンを定義する書き方

ほかのブログなどで紹介されている書き方だと、以下のようになります。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.vm.define "host1" do |host|
    host.vm.hostname = "host1"
    host.vm.network "private_network", ip: "192.168.56.11"
  end

  config.vm.define "host2" do |host|
    host.vm.hostname = "host2"
    host.vm.network "private_network", ip: "192.168.56.12"
  end

end
```

これだと起動したい仮想サーバーの台数を変更するには、``config.vm.difine "hostX" do``〜``end``の一連の定義を追記したり、削除する必要があって面倒です。

# 起動したい仮想サーバーの台数を柔軟に変更できる書き方

起動したい仮想サーバーの台数を環境変数にすることと、一連の定義をループ処理とすることで対応します。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  MAX_OF_SERVERS = (ENV["MAX_OF_SERVERS"] || 1).to_i
  (0...MAX_OF_SERVERS).each do |i|
    config.vm.define "host#{i+1}" do |host|
      host.vm.hostname = "host#{i+1}"
      host.vm.network "private_network", ip: "192.168.56.1#{i+1}"
    end
  end

end
```

環境変数「MAX_OF_SERVERS」を「10」台に設定して仮想サーバーを起動するには、以下のようにします。

```
$ export MAX_OF_SERVERS=10

$ time vagrant up
Bringing machine 'host1' up with 'virtualbox' provider...
Bringing machine 'host2' up with 'virtualbox' provider...
Bringing machine 'host3' up with 'virtualbox' provider...
Bringing machine 'host4' up with 'virtualbox' provider...
Bringing machine 'host5' up with 'virtualbox' provider...
Bringing machine 'host6' up with 'virtualbox' provider...
Bringing machine 'host7' up with 'virtualbox' provider...
Bringing machine 'host8' up with 'virtualbox' provider...
Bringing machine 'host9' up with 'virtualbox' provider...
Bringing machine 'host10' up with 'virtualbox' provider...

中略

real	7m44.957s
user	1m3.005s
sys	0m53.508s
```
```
$ vagrant status
Current machine states:

host1                     running (virtualbox)
host2                     running (virtualbox)
host3                     running (virtualbox)
host4                     running (virtualbox)
host5                     running (virtualbox)
host6                     running (virtualbox)
host7                     running (virtualbox)
host8                     running (virtualbox)
host9                     running (virtualbox)
host10                    running (virtualbox)
```

わたしの環境では、8分弱で10台の仮想サーバーを起動できました。なお、デフォルト(1台)の定義にもどすには、環境変数をリセットすることでもどせます。

```
$ vagrant destroy -f

$ unset MAX_OF_SERVERS

$ vagrant status
Current machine states:

host1                     not created (virtualbox)
```

# まとめ

環境変数を利用して、Vagrantで複数台の仮想サーバーを柔軟に起動できるVagrantfileの書き方を紹介しました。インフラの勉強やテストを少しでも効率よく実施できる一助になれば幸いです。

