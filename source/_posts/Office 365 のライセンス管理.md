---
title: >
  Office 365 のライセンス管理
date: 2018-04-02
tags: O365Identity
---
こんにちは。いつも Office 365 を利用いただきまして、ありがとうございます。
今回は、Office 365 のライセンス管理について、ご紹介いたします。

&nbsp;

Office 365 でユーザーにライセンスを付与するには、GUI（Office 365 管理センター）で操作する方法と、PowerShell で操作する方法があります。
少人数の作業であれば前者で問題ありませんが、ユーザーが多い場合や、スクリプトとして実行したいような場合は後者の方法が有効です。

&nbsp;

よくあるお問い合わせのシナリオに沿って、ライセンス管理の PowerShell コマンドをご案内いたします。
ライセンス名やサービスプランを変更することによって応用が可能です。

&nbsp;


## 事前準備（必須）

下記の作業は、どのシナリオにおいても共通です。

▼Office 365 に PowerShell 接続します。
下記の公開情報に従って Azure Active Directory の PowerShell モジュール（MSOnline）をインストールします。

Azure Active Directory の PowerShell モジュール
<a href="https://blogs.technet.microsoft.com/jpazureid/2017/12/04/aad-powershell/">https://blogs.technet.microsoft.com/jpazureid/2017/12/04/aad-powershell/</a>

Office 365 PowerShell への接続
<a href="https://docs.microsoft.com/ja-jp/office365/enterprise/powershell/connect-to-office-365-powershell">https://docs.microsoft.com/ja-jp/office365/enterprise/powershell/connect-to-office-365-powershell</a>

その後、Connect-MsolService コマンドを実行し、管理者権限で Office 365 に接続します。

▼Office 365 テナントに紐づくライセンスの情報を確認します。
Get-MsolAccountSku コマンドを実行し、ライセンスの AccountSkuId を確認します。

コマンド実行例：
<table width="700" border="0">
<tbody>
<tr>
<th><span style="color: #808080">AccountSkuId</span></th>
<th><span style="color: #808080">ActiveUnits</span></th>
<th><span style="color: #808080">WarningUnits</span></th>
<th><span style="color: #808080">ConsumedUnits</span></th>
<th></th>
</tr>
<tr>
<th><span style="color: #808080">------------</span></th>
<th><span style="color: #808080">-----------</span></th>
<th><span style="color: #808080">------------</span></th>
<th><span style="color: #808080">-------------</span></th>
<th></th>
</tr>
<tr>
<th><span style="color: #808080">contoso:STANDARDPACK</span></th>
<th><span style="color: #808080">100</span></th>
<th><span style="color: #808080">0</span></th>
<th><span style="color: #808080">10</span></th>
<th><span style="color: #808080">＜＜＜Office 365 Enterprise E1</span></th>
</tr>
<tr>
<th><span style="color: #808080">contoso:ENTERPRISEPACK</span></th>
<th><span style="color: #808080">100</span></th>
<th><span style="color: #808080">0</span></th>
<th><span style="color: #808080">10</span></th>
<th><span style="color: #808080">＜＜＜Office 365 Enterprise E3</span></th>
</tr>
<tr>
<th><span style="color: #808080">contoso:ENTERPRISEPREMIUM</span></th>
<th><span style="color: #808080">100</span></th>
<th><span style="color: #808080">0</span></th>
<th><span style="color: #808080">10</span></th>
<th><span style="color: #808080">＜＜＜Office 365 Enterprise E5</span></th>
</tr>
</tbody>
</table>
<strong>・・・</strong>


▼ユーザーの UsageLocation を設定します。
ライセンスを割り当てるユーザーは、事前に UsageLocation が設定されている必要があります。設定されていない場合にはライセンス割り当てに失敗します。
そのため、はじめてライセンス割り当てを行うユーザーの場合は、事前に次のコマンドで UsageLocation を設定します。

下記のコマンドを実行し、特定のユーザーの UsageLocation を ”JP” に設定します。

Set-MsolUser -UserPrincipalName &lt;対象ユーザーの UPN&gt; -UsageLocation "JP"

下記のコマンドを実行し、csv ファイル（後述）に記載されたユーザーの UsageLocation を ”JP” に一括設定します。

  Import-Csv &lt;csv ファイル名&gt; | ForEach-Object {Set-MsolUser -UserPrincipalName $_.UserPrincipalName -UsageLocation "JP"}

\- 参考情報
UsageLocation は、ISO 3166-1 alpha-2 (A2) の 2 文字の国/地域コードを指定します。
<a href="https://www.iso.org/obp/ui/#search/code/">https://www.iso.org/obp/ui/#search/code/</a>


## 事前準備（任意）</h4>
csv ファイルを使用してライセンスの一括割当・変更を行う場合は、下記を事前準備として実施ください。

▼全ユーザーの UserPrincipalName 一覧を csv ファイルに出力します。

Get-MsolUser -all | Select-Object UserPrincipalName | export-csv &lt;出力先の csv ファイル&gt; -Encoding UTF8 -NoTypeInformation

▼特定ライセンスが付与されているユーザーの UserPrincipalName 一覧を csv ファイルに出力します。

Get-MsolUser | where {$_.Licenses.AccountSkuID -contains "&lt;対象ライセンスの AccountSkuId&gt;"} | Select-Object UserPrincipalName | export-csv &lt;出力先の csv ファイル&gt; -Encoding UTF8 -NoTypeInformation


コマンド実行例：※E1 ライセンスが付与されているユーザーの一覧を c:\temp\users.csv ファイルに出力する場合

<span style="color: #000000">Get-MsolUser | where {$_.Licenses.AccountSkuID -contains "contoso:STANDARDPACK"} | Select-Object UserPrincipalName | Export-Csv "C:\Temp\users.csv" -encoding UTF8 -NoTypeInformation</span>

csv ファイルは、下記の形式で作成されます。
ライセンス割当・変更を行う対象ユーザーが事前に分かっている場合は、手動作成でも問題ありません。
---------------------------------------------------
UserPrincipalName
testuser01@contoso.onmicrosoft.com
testuser02@contoso.onmicrosoft.com
・・・
---------------------------------------------------


ユーザーにライセンスを付与する際、特定のサービスプランのみ有効化する場合は、下記を事前準備として実施ください。

▼作業対象ライセンスに含まれるサービスプランの情報を確認します。

(Get-MsolAccountSku | where {$_.AccountSkuId -eq "&lt;確認したいライセンスの AccountSkuId&gt;"}).ServiceStatus

コマンド実行例：　※Office 365 Enterprise E3 の場合（サービスプランは、2018/03/28 時点の情報です）

(Get-MsolAccountSku | where {$_.AccountSkuId -eq "contoso:ENTERPRISEPACK"}).ServiceStatus

&nbsp;
<table width="699" border="0">
<tbody>
<tr>
<th><span style="color: #808080">ServicePlan</span></th>
<th><span style="color: #808080">ProvisioningStatus</span></th>
</tr>
<tr>
<th><span style="color: #808080">-----------</span></th>
<th><span style="color: #808080">------------------</span></th>
</tr>
<tr>
<th><span style="color: #808080">RMS_S_ENTERPRISE</span></th>
<th><span style="color: #808080">Success ＜＜＜Azure Rights Management</span></th>
</tr>
<tr>
<th><span style="color: #808080">EXCHANGE_S_ENTERPRISE</span></th>
<th><span style="color: #808080">Success ＜＜＜Exchange Online (Plan 2)</span></th>
</tr>
<tr>
<th><span style="color: #808080">FLOW_O365_P2</span></th>
<th><span style="color: #808080">Success ＜＜＜Flow for Office 365 Plan 2</span></th>
</tr>
<tr>
<th><span style="color: #808080">FORMS_PLAN_E3</span></th>
<th><span style="color: #808080">Success ＜＜＜Microsoft Forms (Plan E3)</span></th>
</tr>
<tr>
<th><span style="color: #808080">PROJECTWORKMANAGEMENT</span></th>
<th><span style="color: #808080">Success ＜＜＜Microsoft Planner</span></th>
</tr>
<tr>
<th><span style="color: #808080">Deskless</span></th>
<th><span style="color: #808080">Success ＜＜＜Microsoft StaffHub</span></th>
</tr>
<tr>
<th><span style="color: #808080">STREAM_O365_E3</span></th>
<th><span style="color: #808080">Success ＜＜＜Microsoft Stream for O365 E3 SKU</span></th>
</tr>
<tr>
<th><span style="color: #808080">TEAMS1</span></th>
<th><span style="color: #808080">Success ＜＜＜Microsoft Teams</span></th>
</tr>
<tr>
<th><span style="color: #808080">INTUNE_O365/th&gt;</span></th>
<th><span style="color: #808080">Success ＜＜＜Mobile Device Management for Office 365</span></th>
</tr>
<tr>
<th><span style="color: #808080">OFFICESUBSCRIPTION</span></th>
<th><span style="color: #808080">Success ＜＜＜Office 365 ProPlus</span></th>
</tr>
<tr>
<th><span style="color: #808080">SHAREPOINTWAC</span></th>
<th><span style="color: #808080">Success ＜＜＜Office Online</span></th>
</tr>
<tr>
<th><span style="color: #808080">POWERAPPS_O365_P2</span></th>
<th><span style="color: #808080">Success ＜＜＜PowerApps for Office 365 Plan 2</span></th>
</tr>
<tr>
<th><span style="color: #808080">SHAREPOINTENTERPRISE</span></th>
<th><span style="color: #808080">Success ＜＜＜SharePoint Online (Plan 2)</span></th>
</tr>
<tr>
<th><span style="color: #808080">MCOSTANDARD</span></th>
<th><span style="color: #808080">Success ＜＜＜Skype for Business Online (Plan 2)</span></th>
</tr>
<tr>
<th><span style="color: #808080">SWAY</span></th>
<th><span style="color: #808080">Success ＜＜＜Sway</span></th>
</tr>
<tr>
<th><span style="color: #808080">BPOS_S_TODO_2</span></th>
<th><span style="color: #808080">Success ＜＜＜To-Do (Plan 2)</span></th>
</tr>
<tr>
<th><span style="color: #808080">YAMMER_ENTERPRISE</span></th>
<th><span style="color: #808080">Success ＜＜＜Yammer Enterprise</span></th>
</tr>
</tbody>
</table>
&nbsp;

▼PowerShell 上で変数 disabledplans を作成し、無効とするサービス プランの情報を変数に格納します。
下記のコマンドを実行し、割り当てるライセンスのうち、無効にするサービスを指定し、変数 disabledplans に格納します。
$disabledplans = New-Object System.Collections.Generic.List[string]
$disabledplans.Add("&lt;無効にする ServicePlan&gt;")
$disabledplans.Add("&lt;無効にする ServicePlan&gt;")
・・・
$licenses = New-MsolLicenseOptions -AccountSkuId "&lt;割り当てるライセンスの AccountSkuId&gt;" -DisabledPlans $disabledplans


コマンド実行例：
※contoso:ENTERPRISEPACK ライセンスを割り当てる際に、Microsoft Teams、Sway、Yammer Enterprise を無効にする場合
<span style="color: #000000">$disabledplans = New-Object System.Collections.Generic.List[string]
$disabledplans.Add("TEAMS")
$disabledplans.Add("SWAY")
$disabledplans.Add("YAMMER_ENTERPRISE")
$licenses = New-MsolLicenseOptions -AccountSkuId "contoso:ENTERPRISEPACK" -DisabledPlans $disabledplans</span>



## シナリオ A. 新規ユーザーにライセンスを割り当てます。
▼特定のユーザーに対して、全てのサービスプランを有効化してライセンスを付与します。

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -AddLicenses "&lt;付与ライセンスの AccountSkuId&gt;"

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -AddLicenses "&lt;付与ライセンスの AccountSkuId&gt;"}


▼一部のサービスのみ有効化してライセンスを付与します。

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -AddLicenses "&lt;付与ライセンスの AccountSkuId&gt;" -LicenseOptions $licenses

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -AddLicenses "&lt;付与ライセンスの AccountSkuId&gt;" -LicenseOptions $licenses}

## シナリオ B. ユーザーからライセンスを削除します。
▼特定のユーザーに対して、ライセンスを削除します。

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -RemoveLicenses &lt;"削除ライセンスの AccountSkuId"&gt;

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -RemoveLicenses "&lt;削除ライセンスの AccountSkuId&gt;"}


## シナリオ C. 同一ライセンス内で、有効にするサービスプランを変更します。
▼特定のユーザーに対して、サービスプランを変更します。　※AddLicenses オプションは使用しません。

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -LicenseOptions $licenses

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -LicenseOptions $licenses}

## シナリオ D. ライセンス変更を行います。
▼特定のユーザーに対して、ライセンス変更を行います（変更後は、全てのサービスプランを有効化）

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -RemoveLicenses "&lt;変更前の AccountSkuId&gt;" -AddLicenses "&lt;変更後の AccountSkuId&gt;"

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -RemoveLicenses "&lt;変更前の AccountSkuId&gt;" -AddLicenses "&lt;変更後の AccountSkuId&gt;"}

▼特定のユーザーに対して、ライセンス変更を行います（変更後は、一部のサービスのみ有効化）

Set-MsolUserLicense -UserPrincipalName &lt;UPN&gt; -RemoveLicenses "&lt;変更前の AccountSkuId&gt;" -AddLicenses "&lt;変更後の AccountSkuId&gt;" -LicenseOptions $licenses

▼csv ファイルで指定したユーザーに対して、一括で処理するには、下記のコマンドを実行します。

Import-Csv -Path &lt;csv ファイルのパス名&gt; | ForEach-Object {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -RemoveLicenses "&lt;変更前の AccountSkuId&gt;" -AddLicenses "&lt;変更後の AccountSkuId&gt;" -LicenseOptions $licenses}

## 事後確認
▼作業対象ユーザーのライセンス付与状況を確認します。

Get-MsolUser -UserPrincipalName &lt;UPN&gt; | Select-Object -ExpandProperty Licenses

▼サービスプランまで確認するには、下記のコマンドを実行します。

Get-MsolUser -UserPrincipalName &lt;UPN&gt; | Select-Object -ExpandProperty Licenses | Select-Object -ExpandProperty ServiceStatus

ユーザーのライセンス保持状況を確認するには、[Office 365 管理センター] - [アクティブなユーザー] にて csv ファイルをエクスポートする方法が有効です。

[サンプルスクリプト](/blog/Export-Office365LicenseStatus%20のご紹介)も公開しておりますので、ご活用ください。



## 参考情報
Office 365 PowerShell を使用してライセンスをユーザー アカウントに割り当てる
<a href="https://technet.microsoft.com/ja-jp/library/dn771770.aspx">https://technet.microsoft.com/ja-jp/library/dn771770.aspx</a>

Office 365 PowerShell を使ったサービスへのアクセスを無効にする
<a href="https://technet.microsoft.com/ja-jp/library/dn771769.aspx">https://technet.microsoft.com/ja-jp/library/dn771769.aspx</a>

Windows PowerShell コマンドでの新規ユーザー一括作成手順 (ライセンス付与、パスワード同時設定)
<a href="https://answers.microsoft.com/ja-jp/msoffice/forum/msoffice_o365admin-mso_manage/windows-powershell/b0dbbafd-ec0d-401f-9d88-b6b4fc72f6f6">https://answers.microsoft.com/ja-jp/msoffice/forum/msoffice_o365admin-mso_manage/windows-powershell/b0dbbafd-ec0d-401f-9d88-b6b4fc72f6f6</a>


今後も Office 365 サービスに関する有益な情報を発信してまいりますので、弊社サポート ブログをよろしくお願いいたします。