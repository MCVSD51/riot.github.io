---
layout: ja
title: アプリケーション設計
redirect_from: "/ja/guide/application-design"
---

{% include ja/guide-tabs.html %}

## ポリシーではなく、ツール

Riotには、カスタムタグとイベントシステム(observable)、ルータがバンドルされています。これらが、クライアントサイドアプリケーションを構築するために必要な、最も基本的な要素だと考えています。

1. カスタムタグ: ユーザインターフェースのため
2. イベントシステム: モジュール性のために
3. ルータ: URLと「戻るボタン」のため

Riotは、厳格なルールを押しつけるよりも、創造性を発揮するためのベーシックなツールだけを提供します。この柔軟なアプローチにより、開発者に設計上の選択の余地を大きく残しています。

また、なるべくファイルサイズとAPIの数という点でも、基本要素はミニマルであるべきだと考えています。基本的なものはシンプルであるべきで、そうであれば認知的な負荷も最低限で済みます。


## Observable

Observableはイベントを送ったり受け取ったりするための汎用的なツールです。依存性や「結合」を避けてモジュールを分離するために、よく使われるパターンのひとつです。イベントを使うことで、大きなプログラムは小さく簡単なユニットに分割できます。モジュールは追加することも、削除することも、アプリケーションの他の部分に影響を与えずに変更することもできます。

よくあるやりかたは、アプリケーションをひとつのコアと複数のエクステンションに分けることです。コアは何かが起きるとイベントを送ります。新しいアイテムが追加されたり、既存のアイテムが削除されたり、あるいはサーバから何かが読み込まれたり。

Observableを使うことで、エクステンションはイベントを検知して、それらに反応することができるようになります。コアが関知していなくても、コアを拡張できるわけです。これを「疎結合」と言います。

これらのエクステンションはカスタムタグ(UIコンポーネント)の場合も、非UIモジュールの場合もあります。

一度コアとイベントを注意深くデザインしてしまえば、開発チームのメンバーは他の部分に煩わされずにシステムの開発を進めることができます。

[Observable API](/ja/api/observable/)


## ルーティング

ルータはURLと「戻るボタン」を扱うための、汎用的なツールです。Riotのルータは、最小の実装になっています。次のことが可能です:

1. URLのハッシュ部分を変更
2. ハッシュの変更を通知
3. 現在のハッシュを調べる

ルーティングのロジックはどこにでも置くことができます。カスタムタグでも、非UIモジュールでも構いません。ルータを、各部分に指示を出すための、アプリケーションの中心的な要素と位置付けるアプリケーションフレームワークもあります。一方で、URLイベントをキーボートイベントと同じように扱って、アプリケーションの全体的なアーキテクチャを変えないというアプローチもあります。(訳注: どういった設計にするかは、開発者に委ねられています)

ロケーションバーには常にURLが表示されているわけですから、どんなブラウザアプリケーションにもルーティングは必要です。

[ルータAPI](/ja/api/route/)


## モジュール性

カスタムタグはアプリケーションのビュー部分を作ります。モジュール化されたアプリケーションでは、これらのタグは互いについて関知せず、分離されているべきです。理想的には、外側のHTMLレイアウトにかかわらず、同じタグを別プロジェクトでも使えるはずです。

もし、2つのタグがお互いを「知って」いると、互いに依存した「密結合」を呼び込んでしまいます。そうなると、システムを壊さずにタグを自由に動かすことができなくなります。

結合を避けるには、互いを直接呼び出すよりもイベントに登録するようにします。必要なのは、`riot.observable`かそれと同等のpub/subシステムです。

このイベントシステムは、シンプルにAPIから、Facebook Fluxのようなより大きな設計にまで対応できます。

### Riotアプリケーションの設計例

これは、ユーザログインを実現するための、非常に簡略化したRiotアプリケーションの骨格です。

```javascript
// Login API
var auth = riot.observable()

auth.login = function(params) {
  $.get('/api', params, function(json) {
    auth.trigger('login', json)
  })
}


<!-- login view -->
<login>
  <form onsubmit="{ login }">
    <input name="username" type="text" placeholder="username">
    <input name="password" type="password" placeholder="password">
  </form>

  login() {
    opts.login({
      username: this.username.value,
      password: this.password.value
    })
  }

  // any tag on the system can listen to login event
  opts.on('login', function() {
    $(body).addClass('logged')
  })
</login>
```

そして、アプリケーションをマウントします。

```html
<body>
  <login></login>
  <script>riot.mount('login', auth)</script>
</body>
```

上記のセットアップでは、システムは互いを知っている必要がありません。シンプルに「ログイン」イベントを検知して、それぞれに求められたことを果たします。

Observableは疎結合な(モジュール化された)アプリケーションのための、クラシックな構築要素です。