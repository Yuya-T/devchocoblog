+++
date = "2019-10-08T10:24:42+09:00"
draft = false
title = "Web開発ほぼ未経験でChrome Extensionを開発した話"
categories = [ "Web", "Chrome Extension" ]
tags = [ ]
math = true
+++

# Chrome Extension とは
Chromeにインストールする拡張機能です(そのまま)．
インストールすると右上にアイコンが追加されたりしますね．

# 開発の流れ
1. `manifest.json` を記述する<br/>
   これは拡張機能の名前やバージョン，機能などを定義するファイルです．
   [お作法](https://developer.chrome.com/extensions/manifest)に従って書きます．
2. コードを書く<br/>
   Chrome上で動かすので基本的にWebの技術しか使いません．
   画面が必要ならフロントエンドの開発と全く同じ要領で開発できます．
   当然，ReactだろうとVeu.jsだろうと何でも使えます．
   バックグラウンド動作はスクリプトを書くのと全く変わりません．
   ただし，少し違うのはChrome独自のAPIを使えることです．
   何ができるのかは[ドキュメント](https://developer.chrome.com/extensions/api_index)をご覧ください．
   通知を出したり定時実行したり，かなり豊富なAPIが用意されています．
3. アプリストアで公開する

以上！
超簡単！
(簡単だったとは言ってない)



## 今回使った技術
今回作った拡張機能は以下の技術を使って開発しました．

- TypeScript
- React
- Sass

アーキテクチャーとして，Reactと相性の良いFluxを採用しました．


# ハマったポイント
- `npm install @types/hoge` が必要<br/>
  TypeScriptを使っているので型が必要だが，JavaScriptで書かれたライブラリには型宣言がない．
  そのため上記コマンドで型を導入する必要がある．
  これを行わないと，ライブラリは認識されないためコンパイルが失敗する．

- `tsconfig.json` の設定<br/>
  Node.jsを使っているなら `"moduleResolution": "node"` に設定するのを忘れないように．
  さもないとライブラリが認識されない．
  これで軽く $n$ 日は溶かした．

- `manifest.json` にURLを指定する<br/>
  アクセス先のドメインを指定していなかったので `chrome.identity.launchWebAuthFlow` で拡張機能へ戻ってこれなかった．
  CORS のエラーで落ちる．
  `hogehoge.com` へアクセスするなら `*://*.hogehoge.com/*` を `permissions` に追加すればサブドメインなどへもアクセス可能．

- `manifest.json` に独自データを載せられる<br/>
  APIキーなどを記述しておくと便利だった．
  コード中には記述したくないので悩んでいた．
  
- cookieの削除<br/>
  基本的には `chrome.cookies.remove` で削除できる．
  しかし `chrome.identity.launchWebAuthFlow` を実行した時のような別ウィンドウの場合は不可能．
  OAuth などの認証系ならログアウト用のURLへ(存在すれば)アクセスすれば解決できる．
  
- `Alarm` の発火時に毎回APIへアクセスしてユーザー・ステータスを取得したい<br/>
  `chrome.alarms.onAlarm.addListener` を `BackgroundPage` に追加すればよい．
  これによってChromeの起動時に毎回リスナーが設定される．
  `Alarm` の発火はフロントエンド側ではなくバックエンド側なので，このような実装になる．

# 感想
Webのフロントエンド開発が未経験の状態だったので，Reactに慣れるのに少し時間がかかってしまった．
TypeScriptとSassも当然初めてだったが，こちらはネット上の記事をサラッと読むだけで書けた．

Chrome Extension独自のお作法というのは `manifest.json` くらいしかないうえ，この書き方も決まりきったものなのでドキュメントを読めばすぐに分かる．
Webのフロントエンド開発者ならパパッと手軽に開発できると思う．
これを機に過去に自作したユーティリティ系をChrome Extension化しようかなと思った．
