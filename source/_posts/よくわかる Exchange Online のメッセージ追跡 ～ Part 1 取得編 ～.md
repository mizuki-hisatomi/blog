---
title: >
  よくわかる Exchange Online のメッセージ追跡 ～ Part 1 取得編 ～
date: 2016-02-23
tags: EXO
---
みなさんこんにちは、Exchange Server サポートの 杉山 卓弥 です。Exchange Online で送受信されているメッセージを調査する場合、管理者にてメッセージ追跡を実行する機会は非常に多いかと思います。そこで、メッセージ追跡に役立つ情報を順次 Exchange Server サポートよりご紹介していきます。Part 1 では、Exchange Online のメッセージ追跡の実行方法、取得方法について使用頻度が高い順にご説明していきます。

<span style="color: #3366ff">Exchange Server サポートからのお願い</span>
<span style="color: #3366ff">********************************************************</span>
<span style="color: #3366ff">弊社サポートにメッセージ配信に関するお問い合わせをいただく際、メッセージ追跡ログをあらかじめ送付いただくことで、すぐに調査を開始することができます。また、事象の早期解決に結びつくことも多くございますので、事前の HistoricalSearch (詳細版) の取得にご協力をいただけますようお願いいたします。</span>
<span style="color: #3366ff">********************************************************</span>

&lt;取得方法&gt;
A. HistoricalSearch (詳細版)
B. MessageTrace (詳細版)
C. MessageTrace (要約版)
D. HistoricalSearch (要約版)

&lt;事前準備&gt;
Windows Powershell から Exchange Online に接続します。

Title: リモート PowerShell による Exchange への接続
URL: <a target="_blank" href="https://technet.microsoft.com/library/jj984289(v=exchg.160).aspx" rel="noopener">https://technet.microsoft.com/library/jj984289(v=exchg.160).aspx</a>

==================
A. HistoricalSearch (詳細版)
==================
使用頻度 : ★★★
詳細度 : ★★★
実行時間 : ★☆☆ (長い)
取得可能期間 : ★★★ (過去 90 日)

特徴 :
過去 7 日以上経過した単一または複数のメッセージをイベント単位で最も詳細に調査する場合に有効です。弊社サポートで特定のメッセージを調査する際にもご依頼をしている方法です。

取得方法 :
1. メッセージ追跡 (HistoricalSearch) を開始します。

&lt;送信者、受信者を指定して実行する場合&gt;
Start-HistoricalSearch -ReportTitle &lt;任意の実行名&gt; -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -SenderAddress &lt;送信者メールアドレス&gt; -RecipientAddress &lt;受信者メールアドレス&gt; -ReportType MessageTraceDetail -NotifyAddress &lt;通知メールアドレス&gt;

&lt;メッセージ ID を指定して実行する場合&gt;
Start-HistoricalSearch -ReportTitle &lt;任意の実行名&gt; -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -MessageID "&lt;メッセージ ID&gt;" -ReportType MessageTraceDetail -NotifyAddress &lt;通知メールアドレス&gt;

<span style="color: #ff0000"># パラメーターの StartDate および EndDate は協定世界時 (UTC) を基準としています。日本標準時 (JST) での受信時刻とは異なりますので、例えば、2016/02/22 (JST) に受信した場合は、StartDate を 2016/02/21、EndDate を 2016/02/23 と指定することをお勧めします。</span>

2. 実行後に表示された JobId を控えておき、以下のコマンドを実行して Status が Done となっていることを確認します。
Status が Done となるまでには数時間かかる場合があるので、以下のコマンドは Start-HistoricalSearch でのメッセージ追跡開始後、しばらく時間が経ってから実行することをお勧めします。

Get-HistoricalSearch -JobId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" | FL

3. 該当の JobId の Status が Done となったら、FileUrl に記載されている URL より CSV 形式のメッセージ追跡ログをダウンロードします。
URL アクセス時に証明書の警告画面が表示された場合には、[キャンセル] を押して処理を進めます。

注意点 :

1) 1 日経過しても Status が Done とならない場合には、再度新しく HistoricalSearch を作成、開始いただけますようお願いいたします。
2) 手順 3 にて CSV 形式のメッセージ追跡ログをダウンロードできる回数には制限があります。2018年3月時点では ダウンロード回数が20 回を超えると以下のエラーとなります。
----- ここから ----
503
サーバーがビジー状態です
サービスはビジー状態です。しばらくしてからもう一度やり直してください。
----- ここまで ----
エラーとなる場合は、再度手順 1 からメッセージ追跡の手順を実施ください。

&nbsp;

================
B. MessageTrace (詳細版)
================
使用頻度 : ★★☆
詳細度 : ★★☆
実行時間 : ★★★ (短い)
取得可能期間 : ★☆☆ (過去 7 日)

特徴 :
単一または複数のメッセージをイベント単位で調査する場合に有効です。管理者にてコネクタの経由状況やトランスポート ルールの適用状況を確認することができます。

取得方法 :
&lt;送信者、受信者を指定して実行する場合&gt;
Get-MessageTrace -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -SenderAddress &lt;送信者メールアドレス&gt; -RecipientAddress &lt;受信者メールアドレス&gt; | Get-MessageTraceDetail | Export-Csv -Path C:\&lt;フォルダー名&gt;\&lt;ファイル名&gt;.csv -Encoding UTF8 -NoTypeInformation

&lt;メッセージ ID を指定して実行する場合&gt;
Get-MessageTrace -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -MessageId "&lt;メッセージ ID&gt;" | Get-MessageTraceDetail | Export-Csv -Path C:\&lt;フォルダー名&gt;\&lt;ファイル名&gt;.csv -Encoding UTF8 -NoTypeInformation

注意点 :
パラメーターの StartDate および EndDate は協定世界時 (UTC) を基準としています。日本標準時 (JST) での受信時刻とは異なりますので、例えば、2016/02/22 (JST) に受信した場合は、StartDate を 2016/02/21、EndDate を 2016/02/23 と指定することをお勧めします。

================
C. MessageTrace (要約版)
================
使用頻度 : ★★☆
詳細度 : ★☆☆
実行時間 : ★★★ (短い)
取得可能期間 : ★☆☆ (過去 7 日)

特徴 :
多数のメッセージの最終配信ステータスや送信元 IP などを調査する場合に有効な方法です。管理者にてメッセージの配送状況を一覧形式で確認することができます。

取得方法 :
&lt;送信者、受信者を指定して実行する場合&gt;
Get-MessageTrace -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -SenderAddress &lt;送信者メールアドレス&gt; -RecipientAddress &lt;受信者メールアドレス&gt; | Export-Csv -Path C:\&lt;フォルダー名&gt;\&lt;ファイル名&gt;.csv -Encoding UTF8 -NoTypeInformation

&lt;メッセージ ID を指定して実行する場合&gt;
Get-MessageTrace -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -MessageId "&lt;メッセージ ID&gt;" | Export-Csv -Path C:\&lt;フォルダー名&gt;\&lt;ファイル名&gt;.csv -Encoding UTF8 -NoTypeInformation

注意点 :
パラメーターの StartDate および EndDate は協定世界時 (UTC) を基準としています。日本標準時 (JST) での受信時刻とは異なりますので、例えば、2016/02/22 (JST) に受信した場合は、StartDate を 2016/02/21、EndDate を 2016/02/23 と指定することをお勧めします。

================
D. HistoricalSearch (要約版)
==================
使用頻度 : ★☆☆
詳細度 : ★☆☆
実行時間 : ★☆☆ (長い)
取得可能期間 : ★★★ (過去 90 日)

特徴 :
過去 7 日以上経過した多数のメッセージのイベント概要や送信元 IP を調査する場合に有効な方法です。管理者にてメッセージの配送状況を一覧形式で確認することができます。

取得方法 :
1. メッセージ追跡 (HistoricalSearch) を開始します。

&lt;送信者、受信者を指定して実行する場合&gt;
Start-HistoricalSearch -ReportTitle &lt;任意の実行名&gt; -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -SenderAddress &lt;送信者メールアドレス&gt; -RecipientAddress &lt;受信者メールアドレス&gt; -ReportType MessageTrace  -NotifyAddress &lt;通知メールアドレス&gt;

&lt;メッセージ ID を指定して実行する場合&gt;
Start-HistoricalSearch -ReportTitle &lt;任意の実行名&gt; -StartDate yyyy/mm/dd -EndDate yyyy/mm/dd -MessageID "&lt;メッセージ ID&gt;" -ReportType MessageTrace -NotifyAddress &lt;通知メールアドレス&gt;

<span style="color: #ff0000"># パラメーターの StartDate および EndDate は協定世界時 (UTC) を基準としています。日本標準時 (JST) での受信時刻とは異なりますので、例えば、2016/02/22 (JST) に受信した場合は、StartDate を 2016/02/21、EndDate を 2016/02/23 と指定することをお勧めします。</span>

2. 実行後に表示された JobId を控えておき、以下のコマンドを実行して Status が Done となっていることを確認します。
Status が Done となるまでには数時間かかる場合があるので、以下のコマンドは Start-HistoricalSearch でのメッセージ追跡開始後、しばらく時間が経ってから実行することをお勧めします。

Get-HistoricalSearch -JobId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" | FL

3. 該当の JobId の Status が Done となったら、FileUrl に記載されている URL より CSV 形式のメッセージ追跡ログをダウンロードします。
URL アクセス時に証明書の警告画面が表示された場合には、[キャンセル] を押して処理を進めます。

注意点 :
1) 1 日経過しても Status が Done とならない場合には、再度新しく HistoricalSearch を作成、開始いただけますようお願いいたします。
2) 手順 3 にて CSV 形式のメッセージ追跡ログをダウンロードできる回数には制限があります。2018年3月時点では ダウンロード回数が20 回を超えると以下のエラーとなります。
----- ここから ----
503
サーバーがビジー状態です
サービスはビジー状態です。しばらくしてからもう一度やり直してください。
----- ここまで ----
エラーとなる場合は、再度手順 1 からメッセージ追跡の手順を実施ください。

&nbsp;

&lt;参考リンク&gt;
Title: メッセージの追跡を実行し、結果を表示する
URL: <a target="_blank" href="https://technet.microsoft.com/ja-jp/library/jj200712(v=exchg.160).aspx" rel="noopener">https://technet.microsoft.com/ja-jp/library/jj200712(v=exchg.160).aspx</a>

Title: Start-HistoricalSearch
URL: <a target="_blank" href="https://technet.microsoft.com/ja-jp/library/dn621132(v=exchg.160).aspx" rel="noopener">https://technet.microsoft.com/ja-jp/library/dn621132(v=exchg.160).aspx</a>

Title: Get-MessageTrace
URL: <a target="_blank" href="https://technet.microsoft.com/ja-jp/library/jj200704(v=exchg.160).aspx" rel="noopener">https://technet.microsoft.com/ja-jp/library/jj200704(v=exchg.160).aspx</a>

Title: Get-MessageTraceDetail
URL: <a target="_blank" href="https://technet.microsoft.com/ja-jp/library/jj200681(v=exchg.160).aspx" rel="noopener">https://technet.microsoft.com/ja-jp/library/jj200681(v=exchg.160).aspx</a>

*************************************************************************************
本記事は 2016 年 2 月 22 日時点で執筆されたものであり、ご紹介したコマンドの動作、機能は今後変更される場合がございます。
*************************************************************************************