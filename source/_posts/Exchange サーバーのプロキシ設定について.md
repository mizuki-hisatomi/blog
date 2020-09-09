---
title: >
  Exchange サーバーのプロキシ設定について
date: 2020-09-09
tags: Exchange
---

## 概要
Exchange サーバーもハイブリッド環境や外部組織と連携する際などにはウェブ プロキシを設定する必要があります。具体的に必要となる設定についてご案内します。

## 詳細
設定箇所は以下となります。

- Exchange サーバーの InternetWebProxy 

  これは Get-ExchangeServer の InternetWebPorxy となります。

  ```PowerShell
  Set-ExchangeServer <対象サーバー> -InternetWebProxy <プロキシ URI>
  ```

  例:

  ```PowerShell
  Set-ExchangeServer ExSvr1 -InternetWebProxy http://myproxy:8080
  ```

- WinHttp の既定のプロキシ
  
  Exchange サーバーの制御下にない、 OS 側コンポーネントの動作としてインターネット接続が必要になる場合もあります。例えば、公的証明機関から発行された証明書の失効リストを確認するような場合です。この時には、WinHttp の既定のプロキシ設定が利用されるため適切に設定しておく必要があります。

  以下のように設定を確認できます。

  ```
  netsh winhttp show proxy
  ```

  設定を変更する場合には管理者権限で起動した cmd で以下のように実行します。

  ```
  netsh winhttp set proxy proxy-server="<プロキシ URI>" bypass-list="<プロキシ除外リスト>"
  ```

  例:
  ```
  netsh winhttp set proxy proxy-server="http://myproxy:8080" bypass-list="*.foo.com;<local>"
  ```

注1. プロキシ サーバー側では Exchange サーバーからの接続リクエストに対して認証要求をしないよう設定する必要があります。

注2. InternetWebProxy については `http://<プロキシ>` の形式で指定する必要があります。netsh のほうはホスト名のみの指定でも問題はありませんが、異なっていると混乱しやすいため上記例では統一しています。