# Vue3でLINE Login
## 内容
Vue3（実質素のjs）だけでLine Loginをつかってユーザidを取得するサンプル。
認証方式はPKCEでクライアントシークレットをクライアント側に持たなくて済む。 

## 前提
- アクセストークン発行はlambdaとかで別途apiを立ててやる
- LINE devloperコンソールで必要な環境が構築済みであること

## 環境切り替え
.env.<development|production>で環境変数を環境ごとに切り替える。
各変数はxxx.templateを参照してほしい。

## ファイル構成
- src/components/Login.vue ここに認証開始処理を書いている
- .env.development.tempalte 環境変数のテンプレ
- vite-env.d.ts ポート番号とかここで変更する


