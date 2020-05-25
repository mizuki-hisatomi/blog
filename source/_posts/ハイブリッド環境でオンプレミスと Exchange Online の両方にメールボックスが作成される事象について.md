---
title: >
  ハイブリッド環境でオンプレミスと Exchange Online の両方にメールボックスが作成される事象について
date: 2017-07-10
tags: Hybrid
---
こんにちは、Exchange サポートの渡辺です。
今回はハイブリッド環境で同一ユーザーのメールボックスが “オンプレミス” と “Exchange Online” の両方に作成されてしまった際の対処方法についてご紹介します。

&nbsp;
<h3><u><strong>そもそもなぜ両方にメールボックスが作成されてしまうのか</strong></u></h3>
通常、オンプレミスと Exchange Online の双方にメールボックスが作成されることはありませんが、オンプレミスから Exchange Online のメールボックス移動に際して発生することが、過去のお問合せとして報告されています。
&nbsp;
具体的には、Exchange Online へのメールボックスの移動が終わり、その後のメールボックス移動の最終処理でオンプレミス側ではメールボックスからリモート メールボックスへの変換が行われます。
それに伴ってオンプレミス側のディレクトリ情報も更新される動作となっていますが、この際に何らかの要因でディレクトリ情報の更新がスキップされる場合があり、この結果オンプレミスがのメールボックスが残ってしまい、両方にメールボックスが存在する状態が発生します。
&nbsp;
この際、以下のように移動要求自体は CompletedWithWarnings のステータスとなっており、Error 部分には移行の最終処理でクリーンアップ処理がスキップされたため、移動元 (オンプレミス) のメールボックスが残っている可能性がある旨が記録されます。
なお、エラーは発生していますが、メールボックスの移動 (データの移動) 自体はステータス上 Completed で完了している状態となり、前述のとおり Exchange Online への移動自体は完了しているため、メールデータも移動できております。
&nbsp;
**********************
Status                           : CompletedWithWarnings
StatusSummary                 : Failed
State                            : Completed
Error                            : <font color="red">MigrationMRSPermanentException: <span style="background-color: #ffff00">Warning: The mailbox was already moved but failed to</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;update the job state during the final stages of the move. It is possible that the s
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ource mailbox could still be alive since post move cleanup operations were deliberat
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ely not executed.</font>
Report                           : &lt;省略&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017/06/08 2:10:38 [TY1PR0301MB1023] Move has completed and final cleanup has started.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017/06/08 2:10:39 [TY1PR0301MB1023] Target mailbox 'contoso.onmicrosoft.com\121042ca-c198-4fe1-8d
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;25-0b1aefa7e1b5 (Primary)' was successfully reset after the move.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017/06/08 2:10:40 [TY1PR0301MB1023] Waiting for mailbox changes to replicate.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017/06/08 2:11:17 [TY1PR0301MB1023] Request is complete.
**********************
&nbsp;
他にも、以下のような作業を実施した場合には、オンプレミス側でメールボックスが再作成されてしまう可能性がありますのでご注意ください。

・ オンプレミス側でユーザーのディレクトリ情報を、オンプレミスにメールボックスが存在していた時の情報に復元してしまった
・ Exchange Online にメールボックスがある状態で、オンプレミスで Enable-Mailbox を実行してしまった

&nbsp;
<h3><u><strong>両方にメールボックスが作成されてしまった時の動作について</strong></u></h3>
まず、この場合はオンプレミスと Exchange Online のメールボックス両方にログインすることが可能です。
Outlook の場合、Autodiscover の構成次第とはなりますが、基本的にどちらかのメールボックスの情報しか接続できないため、ご注意ください。
どうしてももう一つのメールボックスに接続する必要がある場合は、OWA (OotW) で接続します。
なお、上述のとおりメールボックス データの移行は正常に行えているため、Exchange Online のメールボックスで以前に受信したメール等を確認することもできます。

次に気になるのが、新しいメッセージはどちらのメールボックスに配信されているのかという点です。
答えは、送信者が誰かによって違いがあり、基本的には以下の動作になっています。

   ・ オンプレミスのユーザーから送信されたメール : オンプレミス側のメールボックスに配信される
   ・ Exchange Online のユーザーから送信されたメール : Exchange Online 側のメールボックスに配信される
   ・ 外部から送信されたメッセージ : 最初に経由する環境側のメールボックスに配信される
      (外部からのメールを ExO が最初に受信するメールフローの場合は ExO のメールボックスに、オンプレ側が受信する場合はオンプレのメールボックスに配信される)

この状態では上記の動作となっているため、メールボックスの移動後に配信されたメッセージが、オンプレミスまたは、Exchange Online のどちらかにしか存在しない場合があるので注意が必要です。

&nbsp;
<u><strong><h3>両方にメールボックスが作成されてしまった時の対処方法</strong></u></h3>
メールボックスがオンプレミスと Exchange Online の両方に作成されてしまった場合の対処方法としては、オンプレミスのメールボックスを削除するか、Exchange Online のメールボックスを削除するかの二択となります。
この状況が発生するシナリオとしては、上述のとおり基本的に Exchange Online へ移動を試みている際に、オンプレミスのメールボックスが削除されなかった” というパターンが多いので、ここではオンプレミスのメールボックスを削除する方法をご案内します。
&nbsp;
基本的な流れとしては以下の通りですが、作業中 (2,3 の作業中) は対象のメールボックスに対するメッセージが配信されず、送信者に配信不能通知が応答される可能性がありますのでご注意ください。
&nbsp;
<strong><big>0. オンプレミスのメールボックスに届くメールを Exchange Online に転送するように構成する
1. 削除対象のメールボックス内のメール アイテムを PST にエクスポートする
2. ディレクトリ同期の停止
3. メールボックスの削除 (無効化)
4. ディレクトリ同期の再開と、手順 1 でエクスポートした情報のインポート</strong></big>
&nbsp;
以下、各手順の詳細です。

&nbsp;
<u><strong><big>0. オンプレミスのメールボックスに届くメールを Exchange Online に転送するように構成する</strong></u></big>
この後の手順では、オンプレミスのメールボックス内のアイテムをエクスポートして、オンプレミス側を削除する手順となっているのですが、エクスポートしてから削除されるまでに受信したアイテムについては、Exchange Online に引き継ぐことができなくなってしまいます。
そのため、事前にオンプレミスのメールボックスに届いたメールは Exchange Online に転送されるように構成します。
転送する方法は様々あり、仕分けルールやトランスポート ルール、メールボックスの配信オプションなどが使用できます。
配信オプションを利用する場合は、オンプレミス側で以下のコマンドを実行することで、Exchange Online 側のメールボックスへの転送を設定できます。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Set-Mailbox -Identity &lt;対象のメールボックス&gt; -ForwardingSmtpAddress &lt;ExO のメールボックス固有 (onmicrosoft.com ドメイン等) のメール アドレス&gt;
&nbsp;
ここでポイントとなるのは、転送先が Exchange Online 固有のメールアドレスである必要があるという点です。
Exchange Online のメールボックスの場合は基本的に onmicrosoft.com ドメインなど Exchange Online 側にしか設定されていないメールアドレスがあるため転送するという方法が利用できますが、逆にオンプレミス側のメールボックスを残したいというシナリオの場合、オンプレミス側固有のメールアドレスが設定されているケースは少ないかもしれません。
&nbsp;
この場合は転送するという方法が利用できないため、必要に応じて Exchange Online 側で新しいメールを受信しない  (送信者には NDR を応答する) ように構成する必要があります。
こちらも方法は様々ですが、例えば Exchange Online 側のメールボックスが受信できるサイズの上限を 0 Byte とするのも 1 つの方法です。
※ ディレクトリ同期環境でも Exchange Online 側で以下のコマンドを実行して Exchange Onlinega 側のメールボックスだけに設定を行うことが可能です。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Set-Mailbox -Identity &lt;対象のメールボックス&gt; -MaxReceiveSize 0B
&nbsp;
もちろんオンプレミス固有のメールアドレスがある場合は、先に説明した方法と同じようにオンプレミスに対して転送する設定をいただければ問題ありません。

&nbsp;
<u><strong><big>1. 削除対象のメールボックス内のメール アイテムを PST にエクスポートする</strong></u></big>
削除を行うオンプレミスのメールボックス内に含まれるデータを PST ファイルにエクスポートします。
PST ファイルにエクスポートする方法はいくつかありますが、環境によって実施できないものなどがありますのでお気をつけください。
この手順はメールボックス移動を行った後に、オンプレミスのメールボックスに配信されたメッセージを取得するためのものですので、特に必要なデータが無い場合は、本手順を実施いただかなくても問題はありません。

&nbsp;
Outlook を使用して PST ファイルにエクスポートする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
上述の通り、構成によってはオンプレミスのメールボックスに接続できない場合がありますので注意が必要です。
手順は以下の公開情報をご確認ください。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Title : メール、連絡先、予定表を Outlook の .pst ファイルにエクスポートまたはバックアップする
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://support.office.com/ja-jp/article/Export-or-backup-email-contacts-and-calendar-to-an-Outlook-pst-file-14252b52-3075-4e9b-be4e-ff9ef1068f91">https://support.office.com/ja-jp/article/Export-or-backup-email-contacts-and-calendar-to-an-Outlook-pst-file-14252b52-3075-4e9b-be4e-ff9ef1068f91</a>

&nbsp;
メールボックスのエクスポート要求を作成して、エクスポートする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
管理者によって実施する手順となります。
手順等は以下の公開情報をご確認ください。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Title : メールボックスのインポート要求とエクスポート要求
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://technet.microsoft.com/ja-jp/library/ee633455(v=exchg.150).aspx">https://technet.microsoft.com/ja-jp/library/ee633455(v=exchg.150).aspx</a>

&nbsp;&nbsp;&nbsp;&nbsp;Title : メールボックスのエクスポート要求を作成する
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://technet.microsoft.com/ja-jp/library/ff459227(v=exchg.141).aspx">https://technet.microsoft.com/ja-jp/library/ff459227(v=exchg.141).aspx</a>
&nbsp;&nbsp;&nbsp;&nbsp;※ Exchange 2010 の公開情報とはなりますが、Exchange 2013 や 2016 でも同様です。

&nbsp;
インプレース電子情報開示を使用して、エクスポートする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
こちらも管理者によって実施する手順となります。
手順等は以下の公開情報を参考にしていただければと思いますが、バージョンによって手順が異なるのでご注意ください。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Title : インプレース電子情報開示 (eDiscovery)
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://technet.microsoft.com/ja-jp/library/dd298021(v=exchg.150).aspx">https://technet.microsoft.com/ja-jp/library/dd298021(v=exchg.150).aspx</a>

&nbsp;
<u><strong><big>2. ディレクトリ同期の停止</strong></u></big>
後続の手順でオンプレミスのメールボックスを削除することになりますが、作業途中でディレクトリ同期が行われた場合に、場合によっては Exchange Online のメールボックスまで削除されてしまう場合があります。
基本的には再接続されるため問題はありませんが、可能であればディレクトリ同期を一時的に停止して作業を実施いただければと思います。
※ 理論的にはディレクトリ同期の間隔 (既定では 30 分) 内で作業 (後述の手順 3 の 2,3,4 の手順) を実施いただければ影響はありませんので、必須ではありません。
&nbsp;
AAD Connect を利用している環境では、ディレクトリ同期を実行しているサーバーで Powerhsell を起動し、以下のコマンドを実行することで、定期的に実行されるディレクトリ同期を停止することができます。
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Set-ADSyncScheduler -SyncCycleEnabled $False

&nbsp;
※補足
削除対象のメールボックスで訴訟ホールドが有効な場合は対象のメールボックスを削除できないため、事前にホールドを無効化する必要があります。
また、訴訟ホールドを無効化しても反映までに時間を要する (最大 60 分) ため、ディレクトリ同期を停止する前に事前に実施することをお勧めします。
なお、訴訟ホールドの有効/無効の設定はオンプレミスと Exchange Online で独立しているので、オンプレミス側で削除対象のメールボックスの設定を変更しても Exchange Online のメールボックスでは訴訟ホールドが有効なままとなりますのでご安心ください。
訴訟ホールドを無効にする場合は、以下のコマンドを実行します。

&nbsp;&nbsp;&nbsp;&nbsp;Set-Mailbox -Identity &lt;対象のメールボックス&gt; -LitigationHoldEnabled $False

&nbsp;
<u><strong><big>3. メールボックスの削除 (無効化)</strong></u></big>
不要なメールボックスを削除 (無効化) します。
&nbsp;
1. 事前に EXO、オンプレ双方にて以下のコマンド実行し、現在の設定内容 (特に EmailAddresses の値) を控えます。
   後述の手順で使用しますので、出力結果はテキスト ファイル等にコピーしておきます。

&nbsp;&nbsp;&nbsp;&nbsp;Get-Mailbox &lt;対象のメールボックス&gt; | fl

※ 設定されている内容が多いパラメーターは出力が省略されてしまう場合がありますので、事前に以下のコマンドを実行してください。
　 これによって、出力結果が省略されなくなります。

&nbsp;&nbsp;&nbsp;&nbsp;$FormatEnumerationLimit=-1
&nbsp;
2. オンプレミスで Exchange 管理シェルを起動し、以下のコマンドを実行して、オンプレミスのメールボックスを無効化します。
   これにより、対象のユーザーに付与されている Exchange 属性が削除されます。
   実行後は、メールボックスが存在している AD サイト内すべての DC に無効化された情報が複製されていることをご確認ください。

&nbsp;&nbsp;&nbsp;&nbsp;Disable-Mailbox -Identity &lt;対象のメールボックス&gt;
&nbsp;
3. 続いて以下のコマンドを実行して、リモート メールボックスを有効化します。

&nbsp;&nbsp;&nbsp;&nbsp;Enable-RemoteMailbox -Identity &lt;メールボックス&gt; -RemoteRoutingAddress &amp;ltmail.onmicrosoft.com ドメインのアドレス&gt;

&nbsp;&nbsp;&nbsp;&nbsp;例 :
&nbsp;&nbsp;&nbsp;&nbsp;Enable-RemoteMailbox -Identity user01 -RemoteRoutingAddress user01@contoso.mail.onmicrosoft.com
&nbsp;
※ 補足
設定にも依存いたしますが、一般的には &lt;テナント&gt;.mail.onmicrosoft.com ドメインがオンプレミス/Exchange Online 間のハイブリッド関連機能 (空き時間情報の確認やメール転送等) で使用される SMTP ドメインとなるため、こちらを RemoteRoutingAddress として設定します。
mail.onmicrosoft.com を付与しない構成となっている場合は、代わりにハイブリッド関連機能で使用するドメインのアドレスを付与いただければと思います。

&nbsp;
<u><strong><big>4. ディレクトリ同期の再開と、手順 1 でエクスポートした情報のインポート</strong></u></big>
ディレクトリ同期を再開します。
&nbsp;
1. ディレクトリ同期実行サーバーで以下のコマンドを実行し、スケジュールによる同期を有効化します。

&nbsp;&nbsp;&nbsp;&nbsp;Set-ADSyncScheduler -SyncCycleEnabled $True
&nbsp;
2. 続いて以下のコマンドを実行し、手動で同期を開始します。

&nbsp;&nbsp;&nbsp;&nbsp;Start-ADSyncSyncCycle
&nbsp;
3. ディレクトリ同期を再開後、組織外やオンプレミスからのメッセージ等が正常に Exchange Online 側のメールボックスに配信されるかご確認ください。
問題ないようでしたら、手順 1 でエクスポートした PST ファイルを基に、Exchange Online のメールボックスにインポートを実施します。
&nbsp;
手順はエクスポート時と同様に、Outlook から実施する方法と管理者で実施する方法があります。
詳細は以下の公開情報をご確認ください。

&nbsp;&nbsp;&nbsp;&nbsp;Title : Outlook .pst ファイルからメール、連絡先、予定表をインポートする
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://support.office.com/ja-jp/article/Import-email-contacts-and-calendar-from-an-Outlook-pst-file-431a8e9a-f99f-4d5f-ae48-ded54b3440ac">https://support.office.com/ja-jp/article/Import-email-contacts-and-calendar-from-an-Outlook-pst-file-431a8e9a-f99f-4d5f-ae48-ded54b3440ac</a>

&nbsp;&nbsp;&nbsp;&nbsp;Title : ネットワーク アップロードを使用して PST ファイルを Office 365 にインストールする
&nbsp;&nbsp;&nbsp;&nbsp;URL : <a href="https://support.office.com/ja-jp/article/Use-network-upload-to-import-PST-files-to-Office-365-103f940c-0468-4e1a-b527-cc8ad13a5ea6">https://support.office.com/ja-jp/article/Use-network-upload-to-import-PST-files-to-Office-365-103f940c-0468-4e1a-b527-cc8ad13a5ea6</a>
&nbsp;
手順は以上となります。
それほど頻繁に発生する事象ではありませんが、もし発生してしまった場合は、こちらの対処方法を参考としていただけますと幸いです。

&nbsp;
※本情報の内容（添付文書、リンク先などを含む）は、作成日時点でのものであり、予告なく変更される場合があります。

