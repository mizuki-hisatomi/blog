---
title: >
  参照 DC-GC の固定について
date: 2013-01-07
tags: Exchange
---
Exchange 2007/2010 では、Exchange サーバーが参照可能なドメイン コントローラとグローバル カタログ サーバー (以下 DC/GC) について、Get/Set-ExchangeServer を用いて確認と設定をすることができます。

```
[PS] C:\>Get-ExchangeServer E2010-1-A -Status | fl current*, static*

CurrentDomainControllers        : {DC-1-A.a.local}
CurrentGlobalCatalogs           : {DC-1-A.a.local}
CurrentConfigDomainController   : DC-1-A.a.local
StaticDomainControllers         : {}
StaticGlobalCatalogs            : {}
StaticConfigDomainController    :
StaticExcludedDomainControllers : {}
```

上記の CurrentDomainControllers と CurrentGlobalCatalogs  に表示される DC/GC が、使用可能と認識されている DC/GC となります。
何らかの理由により参照する DC/GC を固定したい場合には上記の Static*** に対象の DC/GC を指定しますが、この場合指定した DC/GC のすべてが利用できなくなった場合にもその他の DC/GC へフェールバックすることはありませんので、ご注意ください。

Title: Set-ExchangeServer
URL: http://technet.microsoft.com/ja-jp/library/bb123716.aspx