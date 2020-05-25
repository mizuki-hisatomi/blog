---
title: >
  アカウントを乗っ取られた際の対応について
date: 2018-07-19
tags: O365Identity
---

Office 365 をご利用いただいているお客様からよく寄せられるご質問のひとつとして、アカウントが乗っ取られた（ハッキングされた）際にどう対処すればよいかという質問があります。
最も一般的なシナリオは、組織のメンバーがフィッシング詐欺の被害者になり、攻撃者にアカウントのパスワードを取得されてしまったというものです。
このブログではアカウント乗っ取りの発覚から、乗っ取りが確定した場合の対応、そして再びアカウントを乗っ取られないようにするための予防策についてご紹介したいと思います。

## 1. 問題の発覚
アカウントが乗っ取られた際によく見られるアクションとしては、例えば以下のようなものがあります。

- メッセージがなくなっている、もしくは削除されている
- 乗っとられたアカウントからメールを受け取ったユーザーがいるが、乗っ取られたアカウントの送信済みフォルダーには送信済みのメールがない
- 管理者やユーザーが設定した覚えのない受信トレイ ルールや転送ルールが設定されており、内容としては、見知らぬアドレスへ転送するものや、[メモ]/[迷惑メール]/[RSS フィード] フォルダーへアイテムを移動す- るようなルールが設定されている
- Global Address List 上のユーザー名が変更されている
- 乗っ取られたユーザーの外部へのメール送信がブロックされており、メールを送信すると "550 5.1.8 Access denied, bad outbound sender AS(XXXXXXX)" という NDR を受信する

## 2. 管理者による調査
Security & Compliance Center や Azure ポータルから、対象のアカウントのアクティビティの詳細を確認し、アカウントが想定していない動きをしていないかを確認します。 一般的な確認項目を以下に記載します。

### Azure ポータル
対象のアカウントにログインしているアクセス元 IP/サインインの場所
サインインに成功/失敗した回数

### O365 管理センター
ユーザー > アクティブなユーザー で対象のアカウントを選択し、[メールの設定] に見覚えのない転送設定が作成されていないか確認
ユーザー > アクティブなユーザー より、管理者アカウントが増えていないか確認

### Security/Compliance センター
統合監査ログ（UnifiedAuditLog) で対象アカウントのアクションを確認
※統合監査ログはあらかじめ有効化されている必要があり、事象が発生してから有効化しても遡ってログを確認することはできません。弊社では、来年の夏までにすべてのテナントで既定で統合監査ログが有効化されるように準備を進めています。


参考情報:
Title: Office 365 の監査ログの検索を有効または無効にする
Url: https://support.office.com/ja-jp/article/office-365-の監査ログの検索を有効または無効にする-e893b19a-660c-41f2-9074-d3631c95a014?ui=ja-JP&rs=ja-JP&ad=JP

※統合監査ログからメールボックス監査ログ（MailboxLogin/アイテムの Create/Delete 等のアクティビティ）を確認するためには、あらかじめ Exchange Online 側で対象のメールボックスの監査ログが有効化されており、かつ、監査対象に適切なアクティビティが定義されている必要があります。

参考情報:
Title: Office 365 でメールボックスの監査を有効にする
Url: https://support.office.com/ja-jp/article/office-365-でメールボックスの監査を有効にする-aaca8987-5b62-458b-9882-c28476a66918?ui=ja-JP&rs=ja-JP&ad=JP

## 3. 対処および再発防止のためのアクション
再発防止策としては一般的に以下のアクションを実施いただくようご案内しております。

- 対象のアカウントに設定されている不要な代理アクセスを削除する
- 外部ドメインへの転送を制限する
- 多要素認証（MFA) を有効化する
- StrongPassword(複雑なパスワード)が設定されるよう強制する
- メールボックス監査ログ/統合監査ログを有効化する

各アクションの詳細については、以下のブログをご参照ください。

Title: How to fix a compromised (hacked) Microsoft Office 365 account
Section: Remediate affected account and improve your security posture
Url: https://blogs.technet.microsoft.com/office365security/how-to-fix-a-compromised-hacked-microsoft-office-365-account/

なお、管理者アカウントについては被害にあった場合の影響が大きいため、あらかじめ多要素認証の有効化を強く推奨させていただきます。
またIDのセキュリティ保護については以下の公開情報についても併せてご参照下さい。

Title: Azure AD でのハイブリッドおよびクラウド デプロイ用の特権アクセスをセキュリティで保護する
Url: https://docs.microsoft.com/ja-jp/azure/active-directory/users-groups-roles/directory-admin-roles-secure?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json

Title: Azure Active Directory Identity Protection
Url: https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection

Title: Azure AD Privileged Identity Management とは
Url: https://docs.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/pim-configure

## リンク集
いずれも英語の記事となりますが、ご紹介したリンク以外にも、アカウント乗っ取りに対処する上で有用な記事を記載いたします。

Title: Finding Illicit Activity The Old Fashioned Way
Url: https://blogs.technet.microsoft.com/office365security/finding-illicit-activity-the-old-fashioned-way/

Title: Responding to a Compromised Email Account in Office 365
Url: https://docs.microsoft.com/en-us/office365/enterprise/responding-to-a-compromised-email-account

以下のブログは O365 セキュリティ チームの公式ブログとなりますので、ご購読いただければ幸いです。

Title: Securing Office 365
Url: https://blogs.technet.microsoft.com/office365security/

今後とも Office 365 をよろしくお願いいたします。