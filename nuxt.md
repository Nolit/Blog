## 発表の流れと目標
初めに今回の発表の目標ですが、「Nuxt.jsを利用して簡単なToDoアプリ」を作るということを目標にします。  
作成したアプリをS3へ配置して動かすところまでやります。  
その中で、Nuxtjsとはどういったものなのかを感じて貰えれば幸いです。  

## nuxtjsとは？
一言で言いますと、ユニバーサルなVuejsアプリケーションのフレームワークです  
universalとは「共通の」という意味がありますが、ここではクライアントもサーバーも同じくvuejsを使う・・・ぐらいに考えてください  

## 利用者数の推移
利用者数の推移を調べました  

![ダウンロード数](./%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BC%E3%83%89%E6%95%B0.JPG?raw=true)

npmパッケージのダウンロード数をnpm trendsで見てます  
一年間の推移ですが、一週間で7200ダウンロードから先週は67,500まで増加しています  
先週については、Vuejsの大きいカンファレンスがあったのでその影響かもしれませんが、利用者が増えているのは間違いないと思います  


## 導入実績
SNSでキラキラしてる人達が使ってるnoteはご存知でしょうか  
文章や写真なんかのコンテンツを無償、有償で共有するサービスです  
そのnoteをAngularからnuxtjsへ移行すると発表がありました  
現在はnuxtjsで作ったテスト用のページを順次公開しているようです  
https://note.mu/konpyu/n/n9b7bf4343514  

## 経緯
そんな伸びを感じるNuxtjsも今は色々な機能がありますが、当初はSSRを支援する目的で作られました(公式より)  
SPAアプリを作る時、最初から存在する全てのページを読み込むのは現実的ではありません  
なので新しいページが必要になる時にjsでページを読み込みますが、ページの読み込み->ajaxによるデータの取得->レンダリング  
というプロセスを踏むので、初回の描画が遅くなってしまいました。  
そのせいで、SEOであったりユーザビリティといった部分が問題となりました。  
解決方法として、「ajaxによるデータの取得->レンダリング」の部分をサーバーサイドで行う動きが出てきました  
これがSSRで、そういった背景がありNuxtjsではSSRをより簡単に行う為、開発されました  
補足としてReactのNext.jsが先に生まれて、その後にNuxt.jsが続いた形です  

## 導入
という風にSSRを目的として作られたフレームワークなのですが、  
単純にSPAアプリを作る上でもメリットがあると思っていて、今日はSSRはやらずにその辺りをやっていきます  
Nuxtはフレームワークというだけあって、規約に基づいて開発をします  
例えば単純なページ遷移を取っても、「ページ用のコンポーネントはどこに置くか」「そのページはどのURLと関連付けるか」がありますが  
これまでvuejsでアプリを作る時には、独自で規約を作っていました。  
Nuxt.jsではそこに一定の規約が作られます  
この後行いますが、  
・デフォルトで存在するページ用ディレクトリにファイルを置くと、ビルド時に自動でURLと関連付けてくれる  
・各ページに対してbladeにあるような継承する機能  
などなど  

規約以外にも  
・webpackのHMR(Hot Module Replacement)  
・静的ファイルの出力  
といった便利な機能がデフォルトで備わっています。  

ただし便利機能については、nuxt.jsではなくてもvue-cliコマンドを使うとある程度やってくれるかもしれません  

## セットアップ
では説明はこれぐらいにして実際にアプリを作っていきたいと思います。  
初めに環境ですが、nodeがv10.13.0、npmが6.4.1にしてください  
プロジェクトの作成ですが、下記のコマンド一つで完了します  

```npm create nuxt-app プロジェクト名```

いくつか質問をされるので、下記画像のように答えてください
※Project Name、description、authorについては変更可能です(念の為、動かなかった気が・・・)

![create-app](./create-app.jpg?raw=true)

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

今までのVuejs+VueRouterだと、コンポーネントとURLのマッピングを作ることが必要でした  
VueRouter公式より  
https://router.vuejs.org/ja/guide/#javascript  
```
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar }
  ]
})

const app = new Vue({
  router
}).$mount('#app')
```

さらに、ページファイルが多すぎると初回のロードが重くなるので、必要な時にサーバーからページを取得する必要が出てきます(遅延ローディング)  
その為にwebpackのコード分割機能を使って下記のように書きます  
Vuejs公式より  
https://jp.vuejs.org/v2/guide/components-dynamic-async.html  
```
Vue.component('async-webpack-example', function (resolve) {
  require(['./my-async-component'], resolve)
})
```

nuxtjsではこんな面倒なことをしなくて良いです  
規約に従ってpagesディレクトリへファイルを作ると、ビルド時にルーティングファイルを生成してくれ、遅延ローディングも自動で行われます  
それではタスクページを作成してみます  
pages/tasksディレクトリに下記のようなindex.vueファイルを作成してください  

```
<template>
    <div>
        タスクページです
    </div>
</template>
```
/loginへアクセスするとページが表示されるはずです  
.nuxt/router.js配下に下記のようなコードが生成されています。  
ルーティングファイルの自動生成を感じてください  
```
const _3444c9d6 = () => import('..\\pages\\tasks\\index.vue' /* webpackChunkName: "pages_tasks_index" */).then(m => m.default || m)
const _6c548c70 = () => import('..\\pages\\index.vue' /* webpackChunkName: "pages_index" */).then(m => m.default || m)

return new Router({
    mode: 'history',
    base: '/',
    linkActiveClass: 'nuxt-link-active',
    linkExactActiveClass: 'nuxt-link-exact-active',
    scrollBehavior,
    routes: [
		{
			path: "/tasks",
			component: _3444c9d6,
			name: "tasks"
		},
		{
			path: "/",
			component: _6c548c70,
			name: "index"
		}
    ],
    
    
    fallback: false
  })
```

次にTOPページに二つのリンクを作成します(一つは検証用です)  

```
<a href="/tasks">タスク一覧(aタグ)</a>  
<nuxt-link :to="'/tasks'">
    タスク一覧
</nuxt-link>
```

また、タスク一覧ページにもTOPへ戻るリンクを作成します

```
<nuxt-link :to="'/'">
    トップへ
</nuxt-link>
```

通常のaタグではdocumentの読み込みを行うためSPAの挙動になりません  
nuxt-link、もしくはrouter-linkを使用してください  
現在はどちらも同じ機能ですが、将来的にはパフォーマンスを考慮した機能がnuxt-linkに提供されるそうです  
それではTOPページタスク一覧ページを遷移してみます  
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

タスク一覧ページからTOPへのリンクを削除します  
TOPページ、タスク一覧ページどちらでもリンクが表示されているはずです  

### 設定
次にnuxtjsに関する設定を見ていきます  
nuxt.config.jsを開いてください  
丁寧にコメントが書かれているので大体分かるかもしれません  
いくつかの項目は後述しますが、まずはタイトルを変えたいと思います  
head内にあるtitleがページのタイトルになります  
変更をしてみて確認してください  
ただ実際の開発では、サイト全体の名前と各ページのタイトルを表示するのが一般的かと思います  
今から「ページタイトル - アプリケーション名」の形式で表示出来るように設定します  
nuxt.config.js内のtitleキーを、下記のようにtitleTemplateへ変更してください  

```
- title: pkg.name,
+ titleTemplate: "%s - サンプルアプリ",
```

%sに各ページのタイトルを埋め込みます  
pages/index.vueを開いてください  
scriptタグ内でexportしているオブジェクトを下記のように変更します  

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

pages/tasks/index.vueにも下記を加えてください

```
<script>
export default {
  head () {
    return {
      title: "タスクページ"
    }
  }
}
</script>
```

アクセスしてみるとページによってタイトルが変わっているのを確認できます  
その他headに設定できる項目については下記を参照してください  
https://github.com/declandewet/vue-meta#recognized-metainfo-properties

## axiosの設定
次にnuxtjsのモジュールという機能について、うすーくお話しします
公式では下記のように説明されています

```
モジュールは、Nuxt.js のコア機能を拡張し、無限のインテグレーションを加える Nuxt.js の拡張機能です。
```

Nuxt.jsが起動する前に、何かしらの変更を加えることが出来ます
今回はHttpクライアントであるaxiosを、Vueインスタンスから扱うということをやってみます
初めにプロジェクトをインストールした時、axiosを使用する設定にしました
実はその設定でaxiosがVueインスタンスから扱えるようになっています
pages/tasks/index.vueに下記を追加してください

```
created: function () {
    this.$axios.$get('tasks').then(res => {
      console.log(res)
    })
}
```

実行してみるとimportをしなくてもVueインスタンスからaxiosを扱えることが分かります
これを行うメリットは、あらかじめセットアップしたaxiosインスタンスが、全ページで使うことが出来ることにあります
セットアップを行う方法はnuxt.config.jsに書くか、プラグインを使用するか、です
今回はプラグインを使用して、axiosにモックの設定を行います

## プラグインによるモックの設定
axiosでモックを行うために、下記コードを実行してください

```
npm i axios-mock-adapter --save-dev
```

plugins/axios.jsへ下記を追加します

```
import MockAdapter from 'axios-mock-adapter'

export default function({ $axios }) {
    const mock = new MockAdapter($axios, { delayResponse: 500 })

    mock.onGet('/tasks').reply(200, {
        tasks: [
            { title: 'タイトル１' },
	    { title: 'タイトル2' }
        ]
    })
}
```

nuxt.config.jsに下記を設定します

```
plugins: [
    '~/plugins/axios.js'
],
```

また、pages/tasks/index.vueを下記のように変更します

```
<template>
    <div>
        タスク一覧
        <ul>
            <li v-for="task in tasks">
                {{ task.title }}
            </li>
        </ul>
    </div>
</template>

<script>
    export default {
        data: function () {
            return {
                tasks: []
            }
        },
        created: function () {
            this.$axios.get('/tasks')
            .then(response => {
                console.log(response.data)
                this.tasks = response.data.tasks
            });
        }
    }
</script>
```

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
