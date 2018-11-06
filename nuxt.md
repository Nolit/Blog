## 発表の流れと目的

## nuxtjsについて

## セットアップ
```npm create nuxt-app プロジェクト名```
これ一つで出来ます

## プロジェクトの説明

### 初めに
package.jsonを見てください
scriptsにdev/build/start/generateが登録されています
それぞれについては後述しますが、一先ず下記のコマンドを実行してください
``` npm run dev ```
このコマンドでdevモードでアプリをサーバ上に立ち上げます
pages/index.vueが表示されています
タイトルの部分を書き換えてみてください
デフォルトでwebpackのHMR(Hot Module Replacement)が設定されており、ブラウザをリロードしなくても変更が反映されます

### ページ遷移
では今からSPAっぽいことをやっていきます
従来のvuejsアプリケーションでは、VueRouterというライブラリを別途入れていました
Nuxtjsの場合、デフォルトで同梱されているので追加のインストールは不要です

ToDo: 従来のVueRouterの書き方をだす

nuxtjsではこれまでのようにルーティングファイルを書かなくてよいです
pagesディレクトリへ規約に従ってファイルを作るとビルド時にルーティングファイルを生成してくれます
それではログインページを作成してみます
pages/loginディレクトリに下記のようなindex.vueファイルを作成してください
```
<template>
    <div>
        ログインページです
    </div>
</template>
```
/loginへアクセスするとページが表示されるはずです
ToDo: 生成されたルーティングを見せる

次にTOPページに二つのリンクを作成します(一つは検証用です)
<a href="/login">ログイン(aタグ)</a>
<nuxt-link :to="'/login'">
    ログイン
</nuxt-link>
また、ログインページにもTOPへ戻るリンクを作成します
<nuxt-link :to="'/'">
    トップへ
</nuxt-link>

通常のaタグではdocumentの読み込みを行うためSPAの挙動になりません
nuxt-link、もしくはrouter-linkを使用してください
現在はどちらも同じ機能ですが、将来的にはパフォーマンスを考慮した機能がnuxt-linkに提供されるそうです
それではTOPページログインページを遷移してみます
開発者ツールで見てみると、aタグはdocumentの読み込みを行うのに対して、nuxt-linkだとページコンポーネントのみの読み込みとなっています
TOPページへ再度戻る時にはコンポーネントの読み込みすら行いません(キャッシュを使用)
この辺りにSPAを感じてください

### テンプレート
次にページの共通化を行います
NuxtjsではBladeでいう継承機能が存在します
layouts/default.vueが既に作成されているので見てください
pagesディレクトリにあるファイルはデフォルトでこのファイルを継承します
templateタグ内にあるnuxtタグに子テンプレートの内容が埋め込まれます
defaultテンプレートにTOPページへのリンクを作成します
```
<nuxt-link :to="'/'" class="button--grey">
    TOP
</nuxt-link>
```

ログインページからTOPへのリンクを削除します
TOPページ、ログインページどちらでもリンクが表示されているはずです

### 設定
次にnuxtjsに関する設定を見ていきます
nuxt.config.jsを開いてください
丁寧にコメントが書かれているので大体分かるかもしれません
いくつかの項目は後述しますが、まずはタイトルを変えたいと思います
head内にあるtitleがページのタイトルになります
変更をしてみて確認してください
ただ実際の開発では、サイト全体の名前と各ページのタイトルを表示するのが一般的かと思います
今から「ページタイトル - アプリケーション名」の形式で表示出来るように設定します
nuxt.config.js内のtitleキーを、titleTemplateに変更してください
値を"%s - サンプルアプリ"にしてください
%sに各ページのタイトルを埋め込みます
pages/index.vueを開いてください
scriptタグ内でexportしている箇所を下記のように変更します
```
export default {
  components: {
    Logo
  },
  head () {
    return {
      title: "トップページ"
    }
  }
}
```
pages/login/index.vueにも下記を加えてください
```
<script>
export default {
  head () {
    return {
      title: "ログインページ"
    }
  }
}
</script>
```
アクセスしてみるとページによってタイトルが変わっているのを確認できます
その他headに設定できる項目については下記を参照してください
https://github.com/declandewet/vue-meta#recognized-metainfo-properties



ご覧の通り、webpackの設定は完全に隠蔽されています
ToDo: 設定の上書き方法の記す



## 余談

```npm install -g @vue/cli @vue/cli-init```

@vue/cli・@vue/cli-init、名前から分かりそうですが簡単に捕捉します
@vue/cliはvuejsでアプリを作る時、必要なプロジェクトのセットアップを行ってくれます
(例: パッケージのインストール、webpackの設定など)
vue createコマンドを実行すると、デフォルトモードとマニュアルモードが選べます
デフォルトでもvuejsアプリが作れますし、マニュアルモードではTypeScript/Vuexなどが入れれます
最新バージョンは3系で、良く記事に出てくるvue-cliパッケージは2系で非推奨です
3系からBeta版としてguiコマンドが導入されており、パッケージの選択やビルド・サーバの起動等、単体テスト、e2eテストなんかもブラウザから出来ます
```vue gui```
次にvue/cli-initですが、一言で表すならテンプレート機能です
vue createのようにパッケージをそれぞれ選ぶのではなく、用意されたテンプレートからプロジェクトのセットアップが出来ます
2系ではvue/cliに同梱されてましたが、3系からは別になったようです

今回は

vue init {template}

Beta版のvue uiはすごいらしい
セットアップ時の項目がGUIから選べて、buildやserveも実行できる



---
theme : "white"
transition : "default"
---

# reveal.js 試してみた

Markdown で書いてみた

---

## 縦スクロール

キーボードの ↓ を押す
Vim使いなら J を押す

--

## これは

- reveal.jsを利用した   
- デモになります。

---

## 終わり

ありがとうございました