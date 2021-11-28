---
title: "iPadとvimでブログ記事を作成する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['zenn','ipad','vim']
published: false
---
# はじめに

はじめての記事投稿となります。

この記事では、Zennでブログをはじめるにあたり、iPadとVimを使って作業ができる環境を作った内容を共有したいと思います。

# 環境

iPadだけで完結したかったけど、Zenn CLIが動かなかったため、GitHubリポジトリ連携などの設定は、MacBookを併用しています。

- MacBook Air (13-inch,2017)/macOS 11.6
- iPad mini (第5世代)/iPadOS 15.1
- 外付けキーボード(iClever IC-BK20)

# GitHubリポジトリ連携

ZennのコンテンツをGitHubリポジトリで管理するため、Zenn公式の記事にそってMacBookにて設定します。

https://zenn.dev/zenn/articles/connect-to-github

# iPadでVimを使えるようにする

いろいろと調べた結果、以下の方法があることがわかった。

- iVimなどの、Vimクローンアプリケーションを使う。
- iSHなどの、Linuxエミュレータアプリケーションを使う。

今回は、Alpine Linuxが動作するiSHというLinuxエミュレータアプリケーションを選んだ。
記事を編集するために必要な、git vimが簡単に揃えられることを期待しました。

https://apps.apple.com/jp/app/ish-shell/id1436902243

## iSHのインストール

Apple Storeからインストールして起動します。

![]()

バージョンを確認しておきます。

```
# cat /etc/alpine-release                                                              
3.12.0
```

## apkの更新

```
# apk update
# apk upgrade
```

## timezoneの設定

```
# apk add tzdata
# cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
# apk del tzdata
```

## shellの入れ替え

```
# apk add bash bash-completion
# sed -e '/^root/s/\/bin\/ash/\/bin\/bash/' /etc/passwd
```

## git vimのインストール

```
# apk add git vim
```

