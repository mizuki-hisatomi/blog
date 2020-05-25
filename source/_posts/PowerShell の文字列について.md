---
title: >
  PowerShell の文字列について
date: 2016-10-12
tags: Exchange
---
日頃から PowerShell をご利用いただいている場合にはご存じかと思いますが、PowerShell では " と ' はどちらも文字列を表現する際に利用されますが、これらは異なる動作をします。
具体的には " (ダブル クォーテーション) はその中で $ 記号を見つけた場合にはそれを変数として評価しその値で置き換えますが、' (シングル クォーテーション) はそういった動作はせずそのまま文字通り文字列として扱います。

例えば、以下のような場合 "hello $name" の値は、まず $name の値である 'world' に置きかえられるため、"hello world" となります。一方、'hello $name' は文字通りの "hello $name" という文字列となります。

```PowerShell
　$name = 'world'
　"hello $name"  # hello world
　'hello $name'  # hello $name
```

なお、ドット (.) は変数名の一部とはならないため、オブジェクトのプロパティ値を評価させる場合には、$() で囲ってあげる必要があります。

例えば以下の例で、対象ユーザーの PrimarySmtpAddress の値に置き換えたい場合には $mailbox.PrimarySmtpAddress を評価させるために $($mailbox.PrimarySmtpAddress) と記載する必要があります。[意図した結果とならない例] のようにそのまま $mailbox.PrimarySmtpAddress と記載すると $mailbox のみが評価されてしまいます。

```PowerShell
　$mailbox = Get-Mailbox "user01"

　# 意図した結果とならない例:
　"PrimarySmtpAddress is $mailbox.PrimarySmtpAddress" # PrimarySmtpAddress is user01.PrimarySmtpAddress

　# 意図した結果となる例:
　"PrimarySmtpAddress is $($mailbox.PrimarySmtpAddress)" # PrimarySmtpAddress is user01@contoso.local
```

上記の話は一般的な PowerShell の話ですが、Exchange にどのように関係するかというと、Message-ID には '$' が含まれる場合があるということです。

例えば以下のように $ が含まれる Mesasage-ID を "" で囲んでコマンドレットを実行しても意図した結果は得られません。

```PowerShell
　Get-MessageTrackingLog -MessageId:"<9AEF1E3F63BB44C$96149F133C253F20@test>"
```

なぜなら `"<9AEF1E3F63BB44C$96149F133C253F20@test>"` の値は実際には、`"<9AEF1E3F63BB44C@test>"` となるためです (この例の場合、`$96149F133C253F20` が変数として評価されますがこのような変数は定義されていないので、値は $null となります)。

この場合には、' ' を利用することで正しいコマンドとなります。

```PowerShell
　Get-MessageTrackingLog -MessageId:'<9AEF1E3F63BB44C$96149F133C253F20@test>'
```

上記のように文字列中に $ が含まれる可能性がある場合などには、" と ' を正しく使い分ける必要があるためご注意ください。
※本情報の内容（添付文書、リンク先などを含む）は、作成日時点でのものであり、予告なく変更される場合があります。