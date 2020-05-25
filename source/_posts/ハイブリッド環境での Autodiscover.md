---
title: >
 ハイブリッド環境での Autodiscover
date: 2014-11-10
tags: Hybrid

---
こんにちは。Exchange サポートの高橋です。
Exchange Online への移行のため、Exchange のハイブリッド環境に関するお問い合わせが増えております。
Exchange チームとしては、これまでにも Autodiscover についてご紹介してきましたが、今回はハイブリッド環境での Autodiscover の動作についてご案内します。

まず、基本的な Autodiscover の動作については、以下を振り返っていただければと思います。

[Autodiscover を振り返る エピソード 1](/blog/Autodiscover%20を振り返る%20エピソード%201)
[Autodiscover を振り返る エピソード 2](/blog/Autodiscover%20を振り返る%20エピソード%202)

上記ブログでもご紹介のとおり、Outlook クライアントが Autodiscover 接続を行う際、以下の順序で接続が試みられます。

　(1) AD に登録された SCP (ServiceConnectionPoint) に設定された URL
　(2) https://<ユーザーの SMTP ドメイン>/autodiscover/autodiscover.xml
　(3) https&#58;//autodiscover.<ユーザーの SMTP ドメイン>/autodiscover/autodiscover.xml<
　(4) <SMTP ドメイン> のローカルの XML ファイル
　(5) http&#58;//autodiscover.<ユーザーの SMTP ドメイン>/autodiscover/autodiscover.xml へのリダイレクト
　(6) _Autodiscover._tcp.<ユーザーの SMTP ドメイン> (SRV レコード)

Outlook クライアントとしては、接続先のサーバーに関わらず、同じ Autodiscover の動作を行います。
では、どうやって Outlook は Autodiscover により Exchange Online の接続先を自動検出しているのか？
これまでオンプレミス Exchange に接続していた Outlook で、どうやって Exchange Online へ移行したメールボックスへ接続するのか？

答えは、オンプレミス組織に存在する、リモート メールボックスに隠されています。
ハイブリッド環境にて、メールボックスを Exchange Online へ移行すると、オンプレミス組織内では、該当のユーザーはリモート メールボックスと呼ばれる、"自組織内にはメールボックスはもたないがメールが有効なユーザー" へと変更されます。(変更は移行処理の中で自動で行われます。)

上記の通り、Outlook としての動作はメールボックスがどこにあろうと変わりませんので、ドメインに参加したクライアント端末は、まず SCP を参照し、SCP に設定されたオンプレミス Exchange の CAS サーバーに接続を試み、もちろん接続に失敗します。
しかし、Exchange Online へ移行済みのユーザーは、リモート メールボックスの RemoteRoutingAddress パラメーターに設定されたアドレスを基に、再度 Autodiscover を行います。

具体的には、以下の順序で Outlook は Autodiscover を使用して接続を試みます。

ユーザー アカウント例
\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~
プライマリ SMTP アドレス: User1&#64;contoso.com
リモート メールボックスに設定された、RemoteRoutingAddress: User1&#64;contoso.mail.onmicrosoft.com
※ RemoteRoutingAddress の値は、以下のコマンドレット実行にて確認できます。
Get-RemoteMailbox <ユーザー名> | FL RemoteRoutingAddress

ハイブリッド環境での Autodiscover の流れ
\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~
1. プライマリ SMTP アドレスを基に参照した SCP に設定された URL への自動検出
(例: https&#58;//OnpremExCAS.contoso.com/Autodiscover/Autodiscover.xml)
場合によってはこの後も contoso.com の SMTP ドメイン名を基に通常の Autodiscover の流れが続きます。
2. User1&#64;contoso.mail.onmicrosoft.com への URL リダイレクトの自動検出 **RemoteRoutingAddress の値
3. https&#58;//contoso.mail.onmicrosoft.com/autodiscover/autodiscover.xml への自動検出
4. https&#58;//autodiscover.contoso.mail.onmicrosoft.com/autodiscover/autodiscover.xml への自動検出
5. contoso.mail.onmicrosoft.com のローカル自動検出
6. http&#58;//autodiscover.contoso.mail.onmicrosoft.com/autodiscover/autodiscover.xml のリダイレクト チェック
7. https&#58;//autodiscover-s.outlook.com/autodiscover/autodiscover.xml への URL リダイレクトの自動検出
8. https&#58;//pod51057.outlook.com/autodiscover/autodiscover.xml への URL リダイレクトの自動検出
9. http&#58;//autodiscover.contoso.mail.onmicrosoft.com/autodiscover/autodiscover.xml のサービス レコードの検索

ドメインに参加している端末をご利用の場合、上記 1 のプロセスが必ず行われるため、オンプレミス Exchange への接続が発生し、証明書の警告が発生する等のお問い合わせをいただきます。
Outlook がインストールされたクライアント端末で以下のレジストリを設定し、Outlook の SCP 参照の動作を明示的に制御することで、ドメインに参加している端末でも Autodiscover の際に SCP の取得処理をスキップするようになります。

<クライアント端末に設定いただくレジストリ>
HKEY_CURRENT_USER\Software\Microsoft\Office\<Outlook バージョン>\Outlook\AutoDiscover

値の名前 : ExcludeScpLookup
値の型 : REG_DWORD
値 : 0x00000001 (1)

※ Outlook のバージョンは、Outlook 2007 が 12.0、Outlook 2010 が 14.0、Outlook 2013 が 15.0 になります。

なお、SCP 参照の処理をスキップすることで、以下のようなメリット・デメリットが考えられます。

<メリット>
・ オンプレミス Exchange に接続した際、自己署名証明書を使用している場合などに発生する、証明書の警告を除外することが可能
・ SCP 参照処理が減る分、Outlook からの Exchange Online メールボックスへの接続時間の短縮につながる

<デメリット>
・ レジストリは端末単位での有効化になるため、同じ端末を使用してオンプレミス組織のメールボックスにアクセスする際、上記 (2) または (3) の autodiscover の URL を名前解決できない場合には接続に失敗する (組織内の autodiscover 用 DNS レコードが必要となる)
・ 組織全体のクライアントにレジストリを設定する場合、GPO にて配布する必要がある

オンプレミス Exchange への接続を制御したい場合には、レジストリの設定をご検討ください。

Exchange サーバーを構築、運用いただいている皆様において、少しでも参考になりましたら幸いです。
ハイブリッド環境に関するお問い合わせも増えてきておりますので、ハイブリッド環境にまつわるブログも今後ご紹介していけたらと思います。
今後も当ブログおよびサポート チームをよろしくお願いいたします。