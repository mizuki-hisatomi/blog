---
title: >
  Export-Office365LicenseStatus のご紹介
date: 2018-05-29
tags: O365Identity
---
今回は Office 365 の管理者の皆様に、ライセンス付与状況を出力する PowerShell スクリプトをご紹介します。

Office 365 のライセンスを確認する場合、GUI ツールとしては Office 365 管理センターが利用できます。また、CUI ツールとしては MSOnline (Azure AD v1) モジュールの Get-MsolUser コマンドも利用できます。しかしながらどちらを利用しても、ライセンスに含まれる各サービス プランのステータスまで含めて一覧表示する機能が標準では用意されていません。

Get-MsolUser コマンドの出力内容を整形する PowerShell スクリプトを書くことで実現が可能ですが、Get-MsolUser コマンドの出力は複雑な構造となっているため、スクリプティングに慣れていないと実装が難しい状況です。例えば Get-MsolUser コマンドの Licenses 属性は、ユーザーに複数のライセンスが割り当てられていれば値が配列になっています。さらに Licenses 属性の子属性として ServiceStatus 属性があり、当該ライセンスに含まれている各サービス プランのステータスに関する情報が入れ子になっています。

このようなスクリプティングに関するお問い合わせを頻繁にお客様より頂くため、これまでのお問い合わせでのお客様のご要望を基に、私たちは <a target="_blank" href="https://github.com/Microsoft/Export-Office365LicenseStatus" rel="noopener">Export-Office365LicenseStatus</a> を GitHub に公開しました。この PowerShell スクリプトを使用すると、Office 365 の各ユーザーに割り当てられているライセンスや、ライセンスに含まれている各サービス プランのステータスを一覧表示することができます。CSV 出力機能も備えており、出力された CSV ファイルを Excel で開いてさらなる集計を行うことも可能です。

実行例を交えながら使い方をご紹介します。
<h2 style="margin-top: 40px;margin-bottom: 20px">ダウンロード方法</h2>
スクリプトは GitHub の <a target="_blank" href="https://github.com/Microsoft/Export-Office365LicenseStatus/releases" rel="noopener">releases</a> ページからダウンロードできます。最新の Export-Office365LicenseStatus.zip をダウンロードしてください。
<h2 style="margin-top: 40px;margin-bottom: 20px">準備</h2>
<ol>
 	<li>スクリプトの実行には MSOnline (Azure AD v1) モジュールが必要です。インストールされていない場合は<a target="_blank" href="https://blogs.technet.microsoft.com/jpazureid/2017/12/04/aad-powershell/" rel="noopener">こちらのページ</a>を参考にしてインストールしてください。</li>
 	<li>ダウンロードした Export-Office365LicenseStatus.zip を展開します。</li>
 	<li>Export-Office365LicenseStatus.ps1 のプロパティをクリックします。</li>
 	<li>(Windows 10) [全般] タブの [セキュリティ] の [許可する] のチェックをオンにします。[許可する] が表示されていない場合は特に操作は必要ありません。
(Windows 7) [全般] タブの [セキュリティ] の [ブロックの解除] をクリックします。[ブロックの解除] が表示されていない場合は特に操作は必要ありません。</li>
 	<li>[OK] をクリックしてプロパティを閉じます。</li>
 	<li>管理者権限で Windows PowerShell を起動します。</li>
 	<li>以下のコマンドを実行します。

```PowerShell
Get-ExecutionPolicy
```

</li>
 	<li>実行結果が RemoteSigned / Bypass / Unrestricted 以外の値だった場合は、以下のコマンドを実行します。
確認が表示されたら「Y」を入力してください。

```PowerShell
Set-ExecutionPolicy RemoteSigned
```

</li>
</ol>
<h2 style="margin-top: 40px;margin-bottom: 20px">基本的な使い方</h2>
用途に応じて複数の使い方がありますので、いくつかのパターンをご紹介します。まずは Windows PowerShell を起動して Export-Office365LicenseStatus.ps1 を保存したフォルダーに移動しておいてください。なお Windows PowerShell を Azure Active Directory に接続していない (Connect-MsolService コマンドを実行していない) 状態でスクリプトを実行すると、認証ウィンドウが表示されますので、Office 365 テナントの全体管理者ユーザーの資格情報を入力してください。
<h3 style="margin-top: 30px;margin-bottom: 20px">全ユーザーの情報をコンソール出力する</h3>

```PowerShell
.\Export-Office365LicenseStatus.ps1 -All $true
```

<h3 style="margin-top: 30px;margin-bottom: 20px">特定のユーザーの情報をコンソール出力する</h3>

```PowerShell
\Export-Office365LicenseStatus.ps1 -UserPrincipalName User01@contoso.onmicrosoft.com
```

<h3 style="margin-top: 30px;margin-bottom: 20px">特定のユーザーの情報を CSV に出力する</h3>

```PowerShell
.\Export-Office365LicenseStatus.ps1 -UserPrincipalName User01@contoso.onmicrosoft.com -CsvOutputPath C:\temp\License.csv
```

<h3 style="margin-top: 30px;margin-bottom: 20px">ライセンスが割り当てられていないユーザーも含むすべてのユーザーの情報を CSV に出力し、指定されたパスにファイルが存在しても上書きする</h3>

```PowerShell
.\Export-Office365LicenseStatus.ps1 -All $true -CsvOutputPath C:\temp\License.csv -ExportNoLicenseUser -Force
```

<h2 style="margin-top: 40px;margin-bottom: 20px">出力イメージ</h2>
コンソール出力の場合は、以下のように出力されます。ここでは見やすいようにスクリプトの実行結果を ft (Format-List) で表示しています。ちなみに ProvisioningStatus が Disabled になっているサービス プランは無効化されており、それ以外のものは有効化されています。
<a target="_blank" href="media/2018/05/2018052801b.png">

![](2018052801b.png)
</a>

CSV 出力の場合は、Excel で表示すると以下のようになります。Excel のフィルタ機能もしくはピボット テーブル等で整形ください。
<a target="_blank" href="media/2018/05/2018052802.png" rel="noopener">

![](2018052802.png)
</a>

このほかにもパラメーターの組み合わせで異なる使い方が可能ですので、必要に応じてお試しください。

ライセンスの割り当てなどの管理を行う方法については<a target="_blank" href="https://blogs.technet.microsoft.com/exchangeteamjp/2018/04/02/office365-license-management/" rel="noopener">こちら</a>の記事をご参照ください。

なお Export-Office365LicenseStatus は弊社の製品・サービスではないため、スクリプト自体に関するサポートは弊社からは提供されません。スクリプト自体に対するフィードバックは GitHub の <a target="_blank" href="https://github.com/Microsoft/Export-Office365LicenseStatus/issues" rel="noopener">issues</a> に投稿していただくようお願いいたします。ベスト エフォートとはなりますが弊社のエンジニアや有志の GitHub ユーザーが対応します。Export-Office365LicenseStatus はオープン ソースのため、読者の皆様からのプル リクエストも歓迎します。

今後も当ブログおよびサポート チームをよろしくお願いいたします。
※ 本情報の内容 (添付文書、リンク先などを含む) は、作成日時点でのものであり、予告なく変更される場合があります。

<span style="color: #ff6600">小間 竜太郎</span>