---
title: >
  Export-CliXml について
date: 2015-08-23
tags: Exchange
---
Exchange 2007 以降では PowerShell を利用して設定情報を出力することができますが、Format-List などで出力するよりも Export-CliXml を使うほうが便利な場合があります。


ご存じの方も多いかと思いますが、PowerShell では豊富なコマンドレットが用意されておりフィルタやソートはもちろん、カスタム オブジェクトを作成してデータを処理することなどが容易になっています。このような機能を利用する場合には Format-List 等ではなく Export-CliXml を活用いただければと思います。Export-CliXml で出力したデータは、Import-Clixml を使って PowerShell のオブジェクトとして復元でき、利用者は自由にデータの操作ができるようになります。

例えば、マシン上の全プロセスの状態を出力するために Get-Process | fl * と実行するとリスト形式で各プロセスの情報が大量に出力されますが、これではデータを利用する際には不便です。例えば結果を特定のプロパティ値でソートしたりすることが簡単にはできません。

代わりに Export-Clixml を使うことでより効率的に利用することができます。
シンプルな Export-Clixml の利用方法は以下です。


```PowerShell
コマンド | Export-Clixml <出力先ファイル>
```


例 1:
```PowerShell
Get-Process | Export-Clixml Processes_Snapshot.xml
```

例 2:
```PowerShell
Get-DistributionGroup | Export-Clixml DistributionGroup_List.xml
```

例えば、上記例 1 のように Export-Clixml で出力した xml ファイルをトラブルシュート等のために利用する場合、以下のように Import-Clixml で取り込んだ後に PowerShell のコマンドレットを利用して必要な情報を迅速に確認できます。
ここでは例として、全プロセスから "Exchange" という文字列を含むものをフィルタし、且つ CPU 利用率を降順でソートして上位 5 つのプロセスをリストしています。


```PowerShell
$snapshot = Import-Clixml Processes_Snapshot.xml
$snapshot | where name -like *Exchange* | sort -Property cpu -Descending | select -First 5 | ft -AutoSize

Handles NPM(K)  PM(K)  WS(K) VM(M)    CPU(s)   Id ProcessName

------- ------  -----  ----- -----    ------   -- -----------
   1124    174 778352 409552  1895 14,100.17 2092 Microsoft.Exchange.Store.Worker
   1788     68 130692 102200   877  1,667.59 1716 Microsoft.Exchange.Diagnostics.Service
    652     57  76584  70748   797    485.73 8964 Microsoft.Exchange.EdgeSyncSvc
   2024    140 209372 170504  1328    418.11 4664 Microsoft.Exchange.ServiceHost
   1916    130 280820 130476  1207    205.13 5016 msexchangerepl
```


情報を出力したマシンではなくとも、PowerShell が利用できるマシンであれば Import-Clixml を利用して上記のように解析することができますので、トラブルシュート等の際にご活用いただければと思います。

参考:  
Export-Clixml  
https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/export-clixml