---
title: >
  オンプレミスの Exchange Server 環境に新規サーバーを追加する際の留意事項について
date: 2017-11-01
tags: Exchange
---
こんにちは。Exchange サポートの益森です。
 
以前、以下のブログにて、Outlook クライアントが追加構築中のサーバーに接続してしまう動作およびその回避方法を案内しておりますが、最近、ハイブリッド環境の構築など、オンプレミス Exchange Server 環境に新規サーバーを追加するケースが増えているため、改めて、簡単にオンプレミスの Exchange Server 環境に新規サーバーを追加する際の留意事項についてご案内をさせていただきます。
 
Title : Active Directory サイトを活用した Exchange 導入
URL : <a href="https://jpmessaging.github.io/blog/Active%20Directory%20サイトを活用した%20Exchange%20導入/">https://jpmessaging.github.io/blog/Active%20Directory%20サイトを活用した%20Exchange%20導入/</a>
 
まず、上記ブログに記載の通り、Outlook クライアントが追加構築中のサーバーに接続してしまった場合、証明書に関するポップアップが表示される可能性がございます。
 
また、環境構成等にもよりますが、Outlook クライアントが意図せず追加構築中のサーバーに接続してしまうことで、Outlook のプロファイルの情報が書き換わってしまい、通常とは異なる設定、ネットワーク経路で Exchange Server に接続することに起因して、最悪の場合、Outlook クライアントが Exchange Server に接続できなくなるという障害が発生してしまう可能性もございます。(※)
 
※追加サーバーの構築中に Outlook を起動しているだけで追加構築中のサーバーに接続してしまう可能性があるため、後述の回避策を実施することをお勧めしております
 
Outlook クライアントが意図せず追加構築中のサーバーに接続してしまう動作の詳細は上記ブログをご確認いただければと思いますが、簡潔に申し上げますと、Outlook クライアントと追加構築中の Exchange Server が同じ AD サイトに所属しているためです。
 
そのため、上記ブログに記載の通り、追加構築をする Exchange Server については、一旦、Exchange Server をインストールする前に Outlook クライアントが存在しない AD サイトに所属させ、Exchange Server のインストール、仮想ディレクトリ等の設定変更、テストを行った後で Outlook クライアントと同じ ADサイトに所属させることで、Outlook クライアントが意図せず追加構築中のサーバーに接続してしまうことを回避できます。
