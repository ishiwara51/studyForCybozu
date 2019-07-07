# Vue.js 学習メモ
## はじめに
#### 宣言的レンダリング
Vue.jsでは、例えば以下のようにして、htmlページにデータを描画できる。
```html
<div id="app">
  {{ message }}
</div>
```
```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

このページを開いた状態で、command + alt + i で開くコンソールに app.message を変更するよう打ち込むと、リアルタイムにページが更新されることがわかる。

また、以下のようにして、描画されたDOMに特定の振る舞いを与えることができる。

```html
<div id="app-2">
  <span v-bind:title="message">
    Hover your mouse over me for a few seconds
    to see my dynamically bound title!
  </span>
</div>
```
```javascript
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: 'You loaded this page on ' + new Date().toLocaleString()
  }
})
```
ここでは、「この要素の title 属性を Vue インスタンスの message プロパティによって更新して保存する」ということが宣言されている。v-bind はディレクティブの一つである。

#### if と ループ処理
if は以下のようにして

```html
<div id="app-3">
  <span v-if="seen">Now you see me</span>
</div>
```
```javascript
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```

for は以下のようにして実現できる。

```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```
```javascript
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: 'Learn JavaScript' },
      { text: 'Learn Vue' },
      { text: 'Build something awesome' }
    ]
  }
})
```

なお、配列へのいわゆる append は、
```
"arrayName".push("key": new item)
```　
で実現される。

#### ユーザー入力の制御
以下のようにして、ボタンとその押下結果を指定できる。

```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
```
```javascript
vvar app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```

ここでは、v-on ディレクティブによって reverseMessage を読んでいる。reverseMessage内では、DOM操作ではなくアプリケーション状態更新のみを行なっているらしい。

ディレクティブには v-model などもあり、これはテキストボックスを描画、その input により変数を変更する、双方向バインディングを可能にする。

#### Vueコンポーネント
以下のコードにより、grocerList 中の各 item について、 todo-item コンポーネントのインスタンスを作成することができる。
```html
<div id="app-7">
  <ol>
    <!--
      各 todo-item の内容を表す todo オブジェクトを与えます。
      これにより内容は動的に変化します。
      また後述する "key" を各コンポーネントに提供する必要があります。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>
```
```javascript
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: 'Vegetables' },
      { id: 1, text: 'Cheese' },
      { id: 2, text: 'Whatever else humans are supposed to eat' }
    ]
  }
})
```

このようにして、親コンポーネントに影響を与えることなく、一定の処理を <todo-item> コンポーネントとして記述できる。要はオブジェクト指向？

## Vueインスタンス
#### Vue インスタンスの作成
全ての Vue アプリケーション は、Vue 関数で新しい Vue インスタンスを作成することによって起動される。オプションの一覧は API リファレンスで参照できる。
```js
var vm = new Vue({
  // オプション
})
```
ここで作成された vm は、Vue アプリケーションにおいてルート Vue インスタンス(root Vue instance) となり、その下層に Todo-item などのオプションコンポーネントが連なる。

#### データとメソッド
Vue インスタンスが作成されると、自身の data オブジェクトの全てのプロパティをリアクティブシステムに追加する。つまり、以下のように、これらのプロパティの値を変更すると、ビューが”反応”し、新しい値に一致するように更新される。

```js
// データオブジェクト
var data = { a: 1 }

// Vue インスタンスにオブジェクトを追加する
var vm = new Vue({
  data: data
})

// インスタンスのプロパティを取得すると、
// 元のデータからそのプロパティが返されます
vm.a == data.a // => true

// プロパティへの代入は、元のデータにも反映されます
vm.a = 2
data.a // => 2

// ... そして、その逆もまたしかりです
data.a = 3
vm.a // => 3
```

vm に格納された data は、元のデータオブジェクトと同一であり、複製ではないことに注意したい。

vm.b = ~ などとして存在しないプロパティを参照してもエラーにはならないらしい。

また、オプションコンポーネント data の宣言と vue インスタンスの宣言の間に Object.freeze(data) というように記述した場合、 data 中のプロパティの全てが定数となって変更を受け付けなくなる。これも変更を試みた際にエラーとはならない。

#### インスタンスライフサイクルフック
Vue インスタンスの宣言時、オプションコンポーネントとしてライフサイクルフック(lifecycle hooks) と呼ばれる関数を実行するよう記述できる。例えば、以下のcreated フックは、インスタンスが生成された後にコードを実行したいときに使われる。

```js
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` は vm インスタンスを指します
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```
この他にもインスタンスのライフサイクルの様々な段階で呼ばれるフックがあり、例えば、mounted、 updated、そして destroyed など。全てのライフサイクルフックは、this が Vue インスタンスを指す形で実行される。ライフサイクルは以下の図のとおり。

![http_responce](images/lifecycle.png)

また、こうしたインスタンスプロパティやコールバックでは、this を持たないアロー関数の使用はエラーを招くらしい。

## テンプレート構文
Vue は、HTMLのテンプレートを内部的に Virtual DOM の描画関数にコンパイルすることで、リアクティブシステムと組み合わせて、アプリケーションの状態が変わった時に最低限の DOM 操作を適用する。

#### 展開
* テキスト：
まずテキストは、{{}} (mustache tag) に内包されることで、対応するオブジェクト中の同名プロパティの値として展開され、プロパティの変更に同期して更新される。この更新を防ぐには、以下のように v-once ディレクティブを使用することで実現できる。
```HTML
<span v-once>This will never change: {{ msg }}</span>
```

* HTML：
mustache はデータを HTML ではなく、プレーンなテキストとして扱う。実際の HTML として出力するためには、mustache の代わりに v-html ディレクティブを使用すれば良い。
```html
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
"XSS脆弱性" なるもののリスクがあるので、このHTML 展開はセキュリティ上推奨されないらしい。

* 属性：
mustache は HTML 属性(おそらくタグのこと)の内部で使用することができず、代わりにv-bind ディレクティブを使用することが求められる。v-bindは、以下のようにした場合、isButtonDisabled が True 的でないなら、描画された<button>要素のなかに disabled は含まれなくなる。
```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

* JavaScript 式の利用：
ここまでの展開は、全てテンプレート中の単純なキーに関するものであったが、実際には JavaScript 式が、全てのデータバインディング内で可能である。
```HTML
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
このサポートは Vue.js によるものであるため、当然ながら Vue インスタンスのデータスコープないでしか動作しない。また、それぞれのバインディングは単一の式だけを含むことができ、変数宣言や、if(){}などのフロー制御は動作しない。

#### ディレクティブ
ディレクティブは v- から始まる特別な属性であり、v-for 以外は単一の JavaScript 式を期待する。ディレクティブの仕事は、属性値の式が変化したときに、リアクティブに結果を DOM に反映することである。

#### 引数
v-bind のように、直後にコロンと引数を要求するものがある。以下の href は、 v-bind ディレクティブに、要素の href 属性に式 url の値を束縛することを教えるための引数である。
```HTML
<a v-bind:href="url"> ... </a>
```
また v-on ディレクティブでは、受け取りたいイベント名として DOM イベントを受け取る。
```HTML
<a v-on:click="doSomething"> ... </a>
```

#### 動的引数
引数に JavaScript 式を使うためには、式を[]で囲む。
```HTML
<a v-bind:[attributeName]="url"> ... </a>
```
例えば、Vue インスタンスが "href" という値の attributeName という data プロパティをもつ場合、このバインディングは v-bind:href と等しくなる。

* 動的引数の制約：動的引数は、null を除くと string に評価されることが想定されており、 null は明示的にバインディングを削除するのに使われる。また、html の属性名としての制約、すなわちスペースや引用符や大文字に関する制約も受ける。例えば、以下は不正となる。
```HTML
<!-- これはコンパイラ警告を引き起こします -->
<a v-bind:['foo' + bar]="value"> ... </a>
```
この場合回避策としては、スペースや引用符を含まない式を使うか、複雑な式を算出プロパティで置き換えるかというところ。加えて、in-DOM テンプレート (HTML ファイルに直接書かれるテンプレート) を使う場合、ブラウザが強制的に属性名を小文字にすることに気をつける必要がある。
```html
<!-- in-DOM テンプレートの中では、v-bind:[someattr] に変換されます -->
<a v-bind:[someAttr]="value"> ... </a>
```

#### 修飾子
修飾子 (Modifier) は、ドットで表記された特別な接尾語で、ディレクティブが特別な方法で束縛されるべきということを示す。例えば、.prevent 修飾子は v-on ディレクティブに、イベントがトリガされた際 event.preventDefault() を呼ぶように伝える。
```HTML
<form v-on:submit.prevent="onSubmit"> ... </form>
```

#### 省略記法
v- 接頭辞は、テンプレート内の Vue 独自の属性を識別するための目印となっており、既存のマークアップに対して、 Vue.js を利用して動的な振る舞いを適用する場合に便利であるが、頻繁に利用されるディレクティブに対しては冗長に感じることがある。したがって、 Vue は 2 つの最もよく使われるディレクティブ v-bind と v-on に対して特別な省略記法を提供している。

* v-bind 省略記法：
```HTML
<!-- 完全な構文 -->
<a v-bind:href="url"> ... </a>

<!-- 省略記法 -->
<a :href="url"> ... </a>

<!-- 動的引数の省略記法 (2.6.0 以降) -->
<a :[key]="url"> ... </a>
```

* v-on 省略記法：
```HTML
<!-- 完全な構文 -->
<a v-on:click="doSomething"> ... </a>

<!-- 省略記法 -->
<a @click="doSomething"> ... </a>

<!-- 動的引数の省略記法 (2.6.0 以降) -->
<a @[event]="doSomething"> ... </a>
```

## 算出プロパティとウォッチャ
#### 算出プロパティ
* テンプレート内に多くのロジックを詰め込むと、コードが肥大化しメンテナンスが難しくなルため、テンプレート内に書く式は簡単なものにしなければならない。 message.split('').reverse().join('') とかは長すぎ。このようなものには算出プロパティを使用すべきである。

* 基本的な例：
```HTML
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```
```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 算出 getter 関数
    reversedMessage: function () {
      // `this` は vm インスタンスを指します
      return this.message.split('').reverse().join('')
    }
  }
})
```

* 算出プロパティ vs メソッド：
上記の js のなかで、computed を method にしても、最終的には、2つのアプローチは完全に同じ結果になる。しかしながら、算出プロパティはリアクティブな依存関係にもとづきキャッシュされるという違いがあり、算出プロパティは、リアクティブな依存関係が更新されたときにだけ再評価される。これはつまり、message が変わらない限りは、reversedMessage に何度アクセスしても、関数を再び実行することなく以前計算された結果を即時に返すため、メモリが節約されることを意味する。
逆に、Date.now() のような、リアクティブな依存を持たない要素のみで構成された computed 内関数の場合、二度と更新されない。

* 算出プロパティ vs 監視プロパティ：
Vue は Vue インスタンス上のデータの変更を監視し反応させることができる、より汎用的な 監視プロパティ(watched property) を提供している。AngularJS などでは watch を使うことが多いが、以下のような命令的な watch コールバックよりも、多くの場合では算出プロパティを利用するほうが良い。
```HTML
<div id="demo">{{ fullName }}</div>
```
```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```
こんな冗長なコードにしたくないので、算出プロパティを使う。
```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

* 算出 Setter 関数：
算出プロパティはデフォルトでは getter 関数のみであるが、必要があれば setter 関数も使える。デフォルトってどういうことですか？？？

#### ウォッチャ
カスタムウォッチャが必要な時もある。データが変わるのに応じて非同期やコストの高い処理を実行したいときに最も便利。
```HTML
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```
```js
<!-- ajax ライブラリの豊富なエコシステムや、汎用的なユーティリティ	-->
<!-- メソッドがたくさんあるので、Vue のコアはそれらを再発明せずに	-->
<!-- 小さく保たれています。この結果として、慣れ親しんでいるものだけを	-->
<!-- 使えるような自由さを Vue は持ち合わせています。			-->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // この関数は question が変わるごとに実行されます。
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // _.debounce は特にコストの高い処理の実行を制御するための
    // lodash の関数です。この場合は、どのくらい頻繁に yesno.wtf/api
    // へのアクセスすべきかを制限するために、ユーザーの入力が完全に
    // 終わるのを待ってから ajax リクエストを実行しています。
    // _.debounce (とその親戚である _.throttle )  についての詳細は
    // https://lodash.com/docs#debounce を見てください。
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```
この場合では、watch オプションを利用することで、非同期処理( API のアクセス)の実行や、処理をどのくらいの頻度で実行するかを制御したり、最終的な answer が取得できるまでは中間の状態にしておく、といったことが可能になっている、らしい。

watch オプションに加えて、命令的な vm.$watch API を利用することもできる。

## クラスとスタイルのバインディング
#### HTMLクラスのバインディング
* オブジェクト構文：v-bind:class にオブジェクトを渡すことでクラスを動的に切り替えることができる。
```HTML
<div v-bind:class="{ active: isActive }"></div>
```
上記の構文は、active クラスの有無がデータプロパティ isActive の真偽性によって決まることを意味している。
```HTML
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```
```js
data: {
  isActive: true,
  hasError: false
}
```
上記の例は、次のように描画される。
```html
<div class="static active"></div>
```
isActive もしくは hasError が変化するとき、クラスリストはそれに応じて更新され、例えば、hasError が true になった場合、クラスリストは "static active text-danger" になる。
束縛されるオブジェクトはインラインでなくてもよく、以下の例も同じ結果を描画する。
```html
<div v-bind:class="classObject"></div>
```
```js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```
また、以下のように、オブジェクトを返す算出プロパティに束縛することもできる。
```html
<div v-bind:class="classObject"></div>
```
```js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

* 配列構文：
v-bind:class に配列を渡してクラスのリストを適用することができる。
```HTML
<div v-bind:class="[activeClass, errorClass]"></div>
```
```js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```
以上は以下のように描画される。
```HTML
<div class="active text-danger"></div>
```
リスト内のクラスを条件に応じて切り替えたい場合は、三項演算子式を使って実現することができる。
```HTML
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```
この場合 errorClass は常に適用されるが、activeClass クラスは isActive が真と評価されるときだけ適用される。冗長になる場合、配列構文の内部ではオブジェクト構文を使うこともできる。
```HTML
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

* コンポーネントにおいて

#### インラインスタイルのバインディング
* オブジェクト構文
v-bind:style で CSSのようなプロパティを指定できる。プロパティ名はキャメルケースでもケバブケースでもOK。
```HTML
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```js
data: {
  activeColor: 'red',
  fontSize: 30
}
```
テンプレートが冗長になるので、data の style オブジェクトに束縛すると良い。算出プロパティとも相性が良い。
```HTML
<div v-bind:style="styleObject"></div>
```
```js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

* 配列構文：
スタイルオブジェクトも配列でバインディングすることで複数適用できる。
```HTML
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

* 自動プリフィックス：
transform など、ベンダー接頭辞を要求される CSS プロパティを使用するとき、Vue.js は自動的に適切な接頭辞を検出し、適用されるスタイルに追加する。
2.3.0 以降では、 style プロパティに複数の (接頭辞付き) 値の配列を設定でき、例えば次のようになる。

## 条件付きレンダリング
v-if ディレクティブを使用すると、ブロックは、ディレクティブの式が真を返す場合のみ描画される。v-else や v-else-if もある。
```HTML
<h1 v-if="awesome">Vue is awesome!</h1>
```

* テンプレートでの v-if による条件グループ：
v-if はディレクティブなので、単一の要素に付加する必要がある。複数の要素と切り替えたい場合、非表示ラッパー (wrapper) として提供される <template> 要素で v-if を使用できる。
```HTML
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

* key による再利用可能な要素の制御：
要素をスクラッチから描画する代わりに再利用することがよくある。たとえば、ユーザーが複数のログインタイプを切り替えることを許可する場合は、次のようにする。
```HTML
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```
上記のコードで loginType を切り替えても、両方のテンプレートが同じ要素を使用するので、ユーザーが既に input フィールドに入力したテキストは消去されない。再利用されている。「再利用しないで」と Vue に師事するには、key 属性を追加する。
```HTML
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```
この時でも、label は key を持たないので、依然再利用されテイル。

* v-show：
v-if とほとんど同じ形で、真偽判定的に要素を表示できる。v-if と違う点として、v-if が偽に当たって描画を行わないのに対し、v-show は常に描画を行い、 CSS の display プロパティを切り替えることで、偽に当たって隠すことが挙げられる。また、<template> 要素をサポートせず、v-else とも連動しない。

* v-if vs v-show：
v-show は 切り替えのコスト面で優れ、v-if は初期描画コストにおいて優れる。

* v-if と v-for：
v-if といっしょに使用されるとき、v-for は v-if より優先度が高くなる。このことで、v-if と v-for を同時に利用することは推奨されないらしい。詳細はスタイルおよびリストレンダリングのガイドに書いてあるらしい。

## リストレンダリング
#### v-for で配列に要素をマッピングする
```HTML
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```
```js
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```
enumerateは、v-for = "item... の item を(item, index) に置き換えて実現できる。

#### オブジェクトの v-for
```HTML
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
```
```js
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```
プロパティ名、すなわちキーを取得するには、以下のように第二引数を入れる。感覚と逆で紛らわしい。index は第３引数。
```html
<div v-for="(value, name, index) in object">
  {{ name }}. {{ value }}:{{index}}
</div>
```
反復処理の順序は Object.key() の列挙順のキーに基づいていて、これが JavaScript エンジンの実装により一貫性が保証されていない。

#### 状態の維持
データのアイテムの順序が変更された場合、アイテムの順序に合わせて DOM 要素を移動する代わりに、 Vue は各要素にその場でパッチを適用して、その特定のインデックスに何を描画するべきかを確実に反映する。

これは効率という意味では優れているが、描画されたリストが子コンポーネントの状態や、一時的な DOM の状態に依存していないときにだけ適している。

Vue が各ノードの識別情報を追跡できるヒントを与えるために、また、先ほど説明したような既存の要素の再利用と並び替えができるように、一意な key 属性を全てのアイテムに与える必要がある。
```HTML
<div v-for="item in items" v-bind:key="item.id">
  <!-- content -->
</div>
```

#### 配列の変化を検出
Vue は画面の更新もトリガするために、監視された配列の以下の変更メソッドを抱合 (wrap) する。
* push()
* pop()
* shift()
* unshift()
* splice()
* sort()
* reverse()

変更メソッドは、名前が示唆するように、それらが呼ばれると元の配列を変更するが、変更しないメソッドもある。例えば、filter()、concat()、そしてslice() のようなメソッドは、元の配列を変更せず、常に新しい配列を返す。これにより既存のリスト DOM を更新しても、リスト全体が更新されて効率が落ちるといったことはないらしい。

注意事項として、JavaScript の制限のため、Vue は配列で以下の変更をリアクティブに検出することができない。
```js
vm.items[indexOfItem] = newValue
vm.items.length = newLength
```
これを回避するには、それぞれ以下のように置き換えれば良い。
```js
Vue.set(vm.items, indexOfItem, newValue)
vm.items.splice(indexOfItem, 1, newValue)
vm.$set(vm.items, indexOfItem, newValue)
```
```js
vm.items.splice(newLength)
```

#### オブジェクトの変更検出の注意
現代の JavaScript の制約のため、Vue はプロパティの追加や削除を検出することはできない。
```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` はリアクティブです

vm.b = 2
// `vm.b` はリアクティブではありません
```

Vue はすでに作成されたインスタンスに新しいルートレベルのリアクティブプロパティを動的に追加することもできない。しかし、以下のように、Vue.set（object、propertyName、value） メソッドを使って、ネストされたオブジェクトにリアクティブなプロパティを追加することは可能。
```js
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
```
ここでネストされている userProfile オブジェクトに新しい age プロパティを追加することができる。
```js
Vue.set(vm.userProfile, 'age', 27)
//もしくは
vm.$set(vm.userProfile, 'age', 27)
```

## イベントハンドリング
v-on で DOM イベントのサブスクリプション、発火時の制御ができる。
```html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```
```js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```
click= に続くのはjs式だけでなく、メソッド名も受け付ける。
```HTML
<div id="example-2">
  <!-- `greet` は、あらかじめ定義したメソッドの名前 -->
  <button v-on:click="greet">Greet</button>
</div>
```
```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // `methods` オブジェクトの下にメソッドを定義する
  methods: {
    greet: function (event) {
      // メソッド内の `this` は、 Vue インスタンスを参照します
      alert('Hello ' + this.name + '!')
      // `event` は、ネイティブ DOM イベントです
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
```

#### インラインメソッドハンドラ
上の例ではイベントを引数として受け取るメソッドが記述されているが、Any なメソッドを記述して、method(arg) として呼び出すことも可能。

また、Any とイベントを同時に引数として受け取りたい場合は、$event 変数を他の引数と一緒に渡してあげれば良い。

#### イベント修飾子
v-on にはイベント修飾子(event modifiers)が提供されている。
* .stop
* .prevent
* .capture
* .self
* .once
* .passive
```HTML
<!-- クリックイベントの伝搬が止まります -->
<a v-on:click.stop="doThis"></a>

<!-- submit イベントによってページがリロードされません -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修飾子は繋げることができます -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 値を指定せず、修飾子だけ利用することもできます -->
<form v-on:submit.prevent></form>

<!-- イベントリスナーを追加するときにキャプチャモードで使います -->
<!-- 言い換えれば、内部要素を対象とするイベントは、その要素によって処理される前にここで処理されます -->
<div v-on:click.capture="doThis">...</div>

<!-- event.target が要素自身のときだけ、ハンドラが呼び出されます -->
<!-- 言い換えると子要素のときは呼び出されません -->
<div v-on:click.self="doThat">...</div>

<!-- 2.1.4から -->
<!-- 最大1回、クリックイベントはトリガされます -->
<a v-on:click.once="doThis"></a>

<!-- 2.3.0から -->
<!-- `onScroll` が `event.preventDefault()` を含んでいたとしても -->
<!-- スクロールイベントのデフォルトの挙動(つまりスクロール)は -->
<!-- イベントの完了を待つことなくただちに発生するようになります -->
<div v-on:scroll.passive="onScroll">...</div>
```

#### キー修飾子
KeyboardEvent.key で公開されている任意のキー名を、ケバブケースに変換することで修飾子として直接使用でくる。
```HTML
<!-- `key` が `Enter` のときだけ、`vm.submit()` が呼ばれます  -->
<input v-on:keyup.enter="submit">
<input v-on:keyup.page-down="onPageDown">
```

#### キーコード
推奨されないらしい。以下のキーコードがエイリアスとして提供されている。
* .enter
* .tab
* .delete
* .esc
* .space
* .up
* .down
* .left
* .right

#### システム修飾キー
* .ctrl
* .alt
* .shift
* .meta (command)
```HTML
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```
通常のキーと異なり、イベント発生時に押されていなければならない。つまり、keyup.ctrl は ctrl だけをはなしてもトリガされない。そうしたいなら keyup.17 のように keyCode を使用すべきらしい。推奨されていないのでは？

* .exact 修飾子：
```HTML
<!-- これは Ctrl に加えて Alt や Shift キーが押されていても発行されます -->
<button @click.ctrl="onClick">A</button>

<!-- これは Ctrl キーが押され、他のキーが押されてないときだけ発行されます -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- これは システム修飾子が押されてないときだけ発行されます -->
<button @click.exact="onClick">A</button>
```

* マウスボタンの修飾子：
.left, .right, .middle。

## フォーム入力バインディング
#### 基本
v-model は、内部的には input 要素に応じて異なるプロパティを使用し、異なるイベントを送出する。
* テキスト
```HTML
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```
value プロパティと input イベントを使用している。

* 複数行テキスト：
```HTML
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```
value プロパティと input イベントを使用している。<textarea>{{text}}</textarea> は動かないのでこっちを使おう。

* チェックボックス：
```HTML

<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
```
```js
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

* ラジオ
```HTML
<input type="radio" id="one" value="One" v-model="picked">
<label for="one">One</label>
<br>
<input type="radio" id="two" value="Two" v-model="picked">
<label for="two">Two</label>
<br>
<span>Picked: {{ picked }}</span>
```

 * 選択：
 ```HTML
 <select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Selected: {{ selected }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```
disabled な空の値のオプションを追加しておくことが推奨される。複数選択を許す場合は下。

```HTML
<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>Selected: {{ selected }}</span>
```

動的オプションは v-for 。
``` HTML
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```

#### 値のバインディング
radio 、 checkbox 、 select オプションの、 v-model でバインディングされる値は通常は静的な文字列 (チェックボックスなら boolean)。
```HTML
<!-- チェックされたとき、`picked` は文字列"a"になります -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` は true かまたは false のどちらかです -->
<input type="checkbox" v-model="toggle">

<!-- 最初のオプションが選択されたとき、`selected` は文字列"abc"です -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```
これ以外の値を Vue インスタンスの動的なプロパティに束縛したい場合は、二つ上の例のように v-bind を使用する。

* チェックボックス：
```HTML
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```
```js
// チェックされたとき:
vm.toggle === 'yes'
// チェックが外されたとき:
vm.toggle === 'no'
```

* ラジオ：
```HTML
<input type="radio" v-model="pick" v-bind:value="a">
```
```js
// チェックしたとき:
vm.pick === vm.a
```

* 選択オプション
```HTML
<select v-model="selected">
<!-- インラインオブジェクトリテラル -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```
```js
// 選択したとき:
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

#### 修飾子
* .lazy：
デフォルトでは、IME 必須言語を除いて、 v-model は各 input イベント後に、データと入力を同期する。change イベント後に同期するように変更するために lazy 修飾子を追加することができる。
```HTML
<!-- "input" の代わりに "change" 後に同期します -->
<input v-model.lazy="msg" >
```

* .number：
ユーザの入力を数値として自動的に型変換したいとき、 v-model に管理された入力に number 修飾子を追加することができる。
```HTML
<input v-model.number="age" type="number">
```
type=number と書いたときでさえ、 HTML の input 要素の value は常に文字列を返すため、これを使うと良い。もし値が parseFloat() で解釈できない場合は、もとの値が返却される。

* .trim :
ユーザの入力から空白を自動的に取り除きたいときは、 v-model に管理された入力に trim 修飾子を追加することができる。
```HTML
<input v-model.trim="msg">
```

## コンポーネントの基本
#### 基本例
```js
// button-counter と呼ばれる新しいコンポーネントを定義します
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```
コンポーネントは名前付きの再利用可能な Vue インスタンスであり、この例の場合、<button-counter> である。このコンポーネントを new Vue で作成されたルート Vue インスタンス内でカスタム要素として使用することができる。
```HTML
<div id="components-demo">
  <button-counter></button-counter>
</div>
```
```js
new Vue({ el: '#components-demo' })
```

コンポーネントは再利用可能な Vue インスタンスなので、data、computed、watch、methods、ライフサイクルフックなどの new Vue と同じオプションを受け入れる。el はルート固有のオプションなどで例外。

#### コンポーネントの再利用
```HTML
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
 ```
何度でもOK。オブジェクト的。

コンポーネントの data オプションは関数でなければならない。これにより、上記の例などにおいてそれぞれのインスタンスのデータを独立させられる。

#### コンポーネントの構成
例えば、ヘッダー、サイドバー、およびコンテンツ領域のコンポーネントがあり、それぞれには一般的にナビゲーションリンク、ブログ投稿などの他のコンポーネントが含まれている、といったアプリケーションは一般的。これらのコンポーネントをテンプレートで使用するには、Vue がそれらを認識できるように登録する必要がある。ンポーネント登録には、グローバルとローカルの2種類があり、ここまでで何回か使った Vue.component はグローバル登録を行なっていた。
```js
Vue.component('my-component-name', {
  // ... オプション ...
})
```
グローバルに登録されたコンポーネントは、その後に作成されたルート Vue インスタンス(new Vue)、およびその下層のサブコンポーネント内のテンプレートで使用できる。

#### プロパティを使用した子コンポーネントへのデータの受け渡し
プロパティはコンポーネントに登録できるカスタム属性であり、値がプロパティ属性に渡されると、それを内包するコンポーネントインスタンスのプロパティになる。
```js
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
```
上記のテンプレートでは、data と同様に、コンポーネントインスタンスでこの値にアクセスできる。

プロパティが登録されると、次のようにそれをカスタム属性としてデータを渡すことができる。
```HTML
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>
```
しかしながら、普通のアプリケーションでは、おそらく data に投稿の配列があり、v-bind を使って動的にプロパティを渡すことができる。
```js
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```
```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```

#### 単一のルート要素
<blog-post>コンポーネントを構築するとき、テンプレートには最終的にタイトル以上のものが含まれる。例えば投稿の内容。
```HTML
<h3>{{ title }}</h3>
<div v-html="content"></div>
```
これだけでは動かず、このテンプレートは親要素でラップしなければならない。
```HTML
<div class="blog-post">
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</div>
```
含めたい情報が増えてきたとき、それぞれの情報ごとにプロパティを定義してしまうと、とてもうるさいものになってしまう。その場合、<blog-post> コンポーネントをリファクタして、単一の post プロパティを受け入れる形にする。
つまり、
```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
></blog-post>
```
これを、
```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>
```
```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```
こうすれば、宣言文を節約できる。さらに、post 新しいプロパティが追加された時にも、<blog-post> 内で自動的に利用可能になる。

#### 子コンポーネントのイベントのサブスクリプション
<blog-post> コンポーネントを開発する際、親コンポーネントとやり取りする機能が必要になる場合がある。えば、ブログの投稿のテキストを拡大するためのアクセシビリティ機能を追加し、他のページのデフォルトのサイズにすることができる。親コンポーネントでは、postFontSize データプロパティを追加することでこの機能をサポートすることができる。

```js
new Vue({
  el: '#blog-posts-events-demo',
  data: {
    posts: [/* ... */],
    postFontSize: 1
  }
})
```
これは、単純に変数的にテンプレート内で使用できる。
```HTML
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
    ></blog-post>
  </div>
</div>
```
すべての投稿の内容の前にテキストを拡大するボタンを追加すると以下になる。
```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button v-on:click="$emit('enlarge-text')">
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```
```HTML
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
      v-on:enlarge-text="postFontSize += 0.1"
    ></blog-post>
  </div>
</div>
```

親コンポーネントは、v-on:enlarge-text="postFontSize += 0.1" リスナのおかげで、このイベントを受け取って postFontSize の値を更新することができる。

* イベントと値を送出する：
イベントを特定の値付きで送出すると便利なことがある。例えば、<blog-post> コンポーネントにテキストをどれだけ拡大するかを指示したい場合、$emit の第二引数を使ってこの値を提供することができる。
```HTML
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
親コンポーネントでイベントをリッスンすると、送出されたイベントの値に $event でアクセスできる。
```HTML
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```
または、イベントハンドラがメソッドの場合、値はそのメソッドの最初のパラメータとして渡される。
```js
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

* コンポーネントで v-model を使う：
カスタムイベントは v-model で動作するカスタム入力を作成することもできる。
```HTML
<input v-model="searchText">
```
これは以下と同じ事。
```HTML
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
コンポーネントで使用する場合、v-model は代わりに以下を行う。
```HTML
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```
こうなると、コンポーネント内の <input> はこうなる。
```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```
v-model はこのコンポーネントで完璧に動作するらしい。よくわからんので要復習。
```HTML
<custom-input v-model="searchText"></custom-input>
```

#### スロットによるコンテンツ配信
HTML 要素と同様に、コンポーネントにコンテンツを渡すことができると便利なことがよくある。
```HTML
<alert-box>
  Something bad happened.
</alert-box>
```
これでエラーアラートを出したい場合、Vue のカスタム <slot> 要素によって非常に簡単になる。
```js
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```
は？

#### 動的なコンポーネント
タブ付きのインターフェイスのように、コンポーネント間を動的に切り替える場合、Vue の <component> 要素と 特別な属性の is で可能になる。
```HTML
<!-- currentTabComponent が変更されたとき、コンポーネントを変更します -->
<component v-bind:is="currentTabComponent"></component>
```

## コンポーネントの登録
#### コンポーネント名
コンポーネントを登録するときには、常に名前を与える。グローバル登録の場合、以下になる。
```js
Vue.component('my-component-name', { /* ... */ })
```
DOM 上で直接コンポーネントを使用する場合は、W3C rules に従ったカスタムタグ名(全て小文字で、ハイフンが含まれていること)が推奨される。

コンポーネント名を定義する時、二つの命名規則の選択肢がある。

* ケバブケース：
```js
Vue.component('my-component-name', { /* ... */ })
```
ケバブケースで命名したコンポーネントでは、そのカスタム要素を参照するときもケバブケースを用いなければならない。

* パスカルケース：
```js
Vue.component('MyComponentName', { /* ... */ })
```
ケバブケースと違い、パスカルケースで命名したコンポーネント中のカスタム要素は、ケバブパスカルどちらによっても参照できる。ただし、DOM内に直接使用する場合にはケバブケースのみ。

#### グローバル登録
ここまでは、 Vue.component のみを使ってコンポーネントを作成してきた。
これらのコンポーネントはグローバル登録されていて、登録後に作成された、全てのルート Vue インスタンス(new Vue)のテンプレート内で使用できる。

#### ローカル登録
グローバルは良くない。コンポーネントを素の Javascript オブジェクトとして定義できる。
```js
Vue.component('component-a', { /* ... */ })
Vue.component('component-b', { /* ... */ })
Vue.component('component-c', { /* ... */ })

new Vue({ el: '#app' })
```
これが、
```js
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }

new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```
こうなる。この方法で登録されたコンポーネントは、グローバル登録と違い他のサブコンポーネント内で利用できない。使用したい場合、例えばComponentB 内で ComponentA を使いたい場合、 ComponentB の宣言を以下のようにする。
```js
var ComponentA = { /* ... */ }

var ComponentB = {
  components: {
    'component-a': ComponentA
  },
  // ...
}
```

#### モジュールシステム
Webpack のモジュールシステムを勉強してから。

## プロパティ
#### プロパティの形式(キャメル vs ケバブ)
DOM(HTML) のテンプレート内においては、HTMLの属性名が大文字小文字を区別しないことから、キャメルのプロパティはケバブを使用する必要がある。

```js
Vue.component('blog-post', {
  // JavaScript 内ではキャメルケース
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```
```HTML
<!-- HTML 内ではケバブケース -->
<blog-post post-title="hello!"></blog-post>
```

#### プロパティの型
これまで、props の中はプロパティ名の文字列リストであった。それぞれのプロパティの属性に特定の型を付与したい場合、以下のようにする。
```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```
この型指定により、異なる型を渡した場合に警告を表示するようにできる。

#### 静的あるいは動的なプロパティの受け渡し
オブジェクト中の全てのプロパティをコンポーネントのプロパティ (props) として渡したい場合は、 v-bind を引数なしで使うことができる。つまり、 v-bind:prop-name が v-bind になる。例えば、 post オブジェクトの場合、
```js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```
こうした時のテンプレートを、
```HTML
<blog-post v-bind="post"></blog-post>
```
以下にする。
```HTML
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

#### 単方向のデータフロー
全てのプロパティは、子プロパティと親プロパティの間に、 単方向のバインディング を形成する。親のプロパティが更新されると、子へと流れるが、それ以外の方法でデータが流れることはない。親コンポーネントが更新されるたびに、子コンポーネント内の全てのプロパティが最新の値へと更新され、子コンポーネント内だけプロパティが変化するということはない。これが行われた場合、警告が出る。

子コンポーネント内のプロパティを変化させたい場合、二つのケースがある。
1. プロパティを初期値として受け渡し、子コンポーネントにてローカルのデータとして後で利用したいと考える場合：この場合、プロパティの値をローカルの data の初期値として定義することが推奨される。
```js
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```
2. プロパティを未加工の値として渡す場合：この場合、プロパティの値を使用した算出プロパティを別途定義することが推奨される。
```js
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

#### プロパティのバリデーション
コンポーネントは、プロパティに対して既存の型などの要件を指定することができる。指定した要件が満たされない場合、 Vue はブラウザの JavaScript コンソールにて警告を表示する。これは、他の人が利用することを意図したコンポーネントを開発する場合に、特に便利。

プロパティへのバリデーションは、以下のように、文字列の配列の代わりに、 props の値へとバリデーションの条件をもったオブジェクトを渡すことで指定できる。

```js
Vue.component('my-component', {
  props: {
    // 基本的な型の検査 (`null` と `undefined` は全てのバリデーションにパスします)
    propA: Number,
    // 複数の型の許容
    propB: [String, Number],
    // 文字列型を必須で要求する
    propC: {
      type: String,
      required: true
    },
    // デフォルト値つきの数値型
    propD: {
      type: Number,
      default: 100
    },
    // デフォルト値つきのオブジェクト型
    propE: {
      type: Object,
      // オブジェクトもしくは配列のデフォルト値は
      // 必ずそれを生み出すための関数を返す必要があります。
      default: function () {
        return { message: 'hello' }
      }
    },
    // カスタマイズしたバリデーション関数
    propF: {
      validator: function (value) {
        // プロパティの値は、必ずいずれかの文字列でなければならない
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

* 型の検査
type はカスタムコンストラクタ関数であり、instanceof によるアサーションを行う。
```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```
例えば、このコンストラクタ関数は、以下のように利用できる。
```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

#### プロパティでない属性
プロパティでない属性は、属性としてコンポーネントに受け渡されるが、対応するプロパティは定義されていない。

例えば、サードパーティの bootstrap-date-input コンポーネントを用いて data-date-picker 属性を要求する Bootstrap プラグインを使っている場合、その属性をコンポーネントのインスタンスに追加できる。
```HTML
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```
また、この data-date-picker="activated" 属性は、自動的に bootstrap-date-input のルート属性へと追加される。
```HTML
<input type="date" class="form-control">
```
このようなテンプレートが bootstrap-date-input 側にあった場合、この date プラグインのテーマを指定するには、以下のような特定のクラスを追加する必要がある。
```HTML
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```
この場合、異なる2つのクラスが定義されている。
* form-control：コンポーネント自身のテンプレート内で定義されている。
* date-picker-theme-dark：親コンポーネントによって受け渡される。

ほとんどの属性においては、コンポーネント側の値が、コンポーネントに受け渡された値へと置換される。

コンポーネントのルート要素に対して、属性を継承させたくない場合は、コンポーネントのオプション内にて inheritAttrs: false を設定できる。
```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```
inheritAttrs: false と $attrs を併用すると、要素に送る属性を手動で決定することができる。これは、基底コンポーネントの利用においてはしばしば望ましいことがある。
```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```
このパターンを使用すると、ルート上にある要素を気にすることなく、生の HTML 要素のように基底コンポーネントを利用できる。
```HTML
<base-input
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```

## カスタムイベント
#### イベント名
コンポーネントやプロパティとは違い、イベント名の大文字と小文字は自動的に変換されない。その代わり発火されるイベント名とイベントリスナ名は全く同じにする必要がある。例えばキャメルケースのイベント名でイベントを発火した場合、以下のようにケバブケースでリスナ名を作っても参照されない。
```js
this.$emit('myEvent')
```
```HTML
<!-- 動作しません -->
<my-component v-on:my-event="doSomething"></my-component>
```
さらに、html内では myEvent は myevent になってしまうので、結局参照できない。コンポーネントやプロパティとは違い、イベント名は JavaScript 内で変数やプロパティ名として扱われることはないので、ケバブケースを使うべき。

#### v-model を使ったコンポーネントのカスタマイズ
デフォルトではコンポーネントにある v-model は value をプロパティとして、input をイベントして使う一方、チェックボックスやラジオボタンなどのインプットタイプは value 属性を別の目的で使う事がある。model オプションを使うことでこういった衝突を回避する事ができる。
```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
```HTML
<base-checkbox v-model="lovingVue"></base-checkbox>
```
lovingVue の値が checked プロパティに渡り、<base-checkbox> が change イベントを新しい値で発火した時に lovingVue プロパティが更新される。

#### コンポーネントにネイティブイベントをバインディング
コンポーネントのルート要素にあるネイティブイベントをサブスクリプトしたい場合、.native 修飾子を v-on に付けてる。
```HTML
<base-input v-on:focus.native="onFocus"></base-input>
```
<input> など特定の要素をサブスクリプとしたい場合、例えば上にある <base-input> コンポーネントがリファクタリングされた場合、元要素は <label> 要素になってしまう可能性があり、危険。
```HTML
<label>
  {{ label }}
  <input
    v-bind="$attrs"
    v-bind:value="value"
    v-on:input="$emit('input', $event.target.value)"
  >
</label>
```
このような場合は親にある .native リスナは動作しなくなり、onFocus ハンドラが呼ばれるはずの時に呼ばれなくなる。

この問題を解決するために Vue は $listeners という、コンポーネントで使えるリスナオブジェクトの入ったプロパティを提供している。
```js
{
  focus: function (event) { /* ... */ }
  input: function (value) { /* ... */ },
}
```
$listeners プロパティを使うことで、コンポーネントの全てのイベントリスナを v-on="$listeners" を使って特定の子要素に送ることができる。以下の inputListeners の様に新しい算出プロパティを作った方が便利なことも多い。
```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` が複数のオブジェクトを一つの新しいオブジェクトにマージします
      return Object.assign({},
        // 親からの全てのリスナを追加します
        this.$listeners,
        // そしてカスタムリスナを追加したり
        // すでに存在するリスナの振る舞いを変えることができます
        {
          // こうすることでコンポーネントが v-model と動作します
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```
.native 修飾子なしで全ての同じ要素とリスナが動作する。

#### .sync 修飾子
子コンポーネントは親でも子でもその変更元が明らかでなくても親を変更させることができるようになってしまうため、双方向バインディングはメンテナンスの問題を引き起こす可能性がある。このため代わりに update:myPropName というパターンでイベントを発火させる事が推奨される。例えば title というプロパティを持つ仮のコンポーネントがあった場合、意図的に新しい値を割り当てる事ができる。
```js
this.$emit('update:title', newTitle)
```
こうする事で、必要な場合、親がこのイベントをサブスクリプとし、ローカルデータプロパティを更新することができる。例えば以下。
```HTML
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```
これを、.sync 修飾子で短く書くことができる。
```HTML
<text-document v-bind:title.sync="doc.title"></text-document>
```
このように v-bind に .sync をつける場合、= の後に式を指定しても動作しない。v-model と同様にバインドしたいプロパティ名のみを指定する。オブジェクトを使えば、複数のプロパティを一度にセットできる。
```HTML
<text-document v-bind.sync="doc"></text-document>
```
こうする事で doc オブジェクト内の各プロパティ (例えば title) がひとつのプロパティとして渡され、v-on アップデートリスナがそれぞれに付けられる。

## スロット
Vue には Web Components spec draft にヒントを得たコンテンツ配信 API が実装されており、 <slot> 要素をコンテンツ配信の受け渡し口として利用できる。例えば以下。
```HTML
<navigation-link url="/profile">
  Your Profile
</navigation-link>
```
この navigation-link のテンプレートは以下になる。
```HTML
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot></slot>
</a>
```
コンポーネントを描画する時、 <slot></slot> は「Your Profile」に置換される。つまり、<navigation-link></navigation-link> の中身に置換されるので、以下のように任意のテンプレートを入れることが出来る。
```html
<navigation-link url="/profile">
  <!-- Font Awesome のアイコンを追加 -->
  <span class="fa fa-user"></span>
  Your Profile
</navigation-link>
```
また、以下のようにコンポーネントを入れることもできる。
```HTML
<navigation-link url="/profile">
  <!-- コンポーネントを使ってアイコンを追加 -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Your Profile
</navigation-link>
```
navigation-link のテンプレートに <slot> が含まれない場合、開始タグと終了タグの間にある任意のコンテンツは破棄される。

#### コンパイルスコープ
スロットの中でデータを扱いたい場合、以下のようにする。
```HTML
<navigation-link url="/profile">
  Logged in as {{ user.name }}
</navigation-link>
```
このスロットのスコープは、テンプレートの残りの部分であるため、<navigation-link> のスコープにアクセスすることはできない。例えば、url にはアクセスできない。ルールとしては、以下のようになる。
```
親テンプレート内の全てのものは親のスコープでコンパイルされ、子テンプレート内の全てものは子のスコープでコンパイルされる。
```

#### フォールバックコンテンツ
スロットに対して、コンテンツがない場合にだけ描画されるフォールバック (つまり、デフォルトの) コンテンツを指定すると便利な場合がある。例えば、以下の <submit-button> コンポーネントにおいて、<button> の中に「Submit」という文字を描画したい場合は多い。「Submit」をフォールバックコンテンツにするには、<slot> タグの中に記述する。
```HTML
<button type="submit">
  <slot></slot>
</button>
```
これを、
```HTML
<button type="submit">
  <slot>Submit</slot>
</button>
```
そして、以下のように、親コンポーネントからスロットのコンテンツを指定せずに <submit-button> を使うと、
```HTML
<submit-button></submit-button>
```
以下のテンプレートが返る。
```HTML
<button type="submit">
  Submit
</button>
```
コンテンツを指定すれば、Submit はそのコンテンツに上書きされる。

#### 名前付きスロット
例えば、<base-layout> コンポーネントが下記のようなテンプレートだった場合、複数のスロットが欲しい。
```HTML
<div class="container">
  <header>
    <!-- ここにヘッダコンテンツ -->
  </header>
  <main>
    <!-- ここにメインコンテンツ -->
  </main>
  <footer>
    <!-- ここにフッターコンテンツ -->
  </footer>
</div>
```
こういった場合のために、 <slot> 要素は name という特別な属性を持っていて、追加のスロットを定義できる。
```HTML
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
name のないスロットがあるが、これは暗黙的に default という名前になる。名前付きスロットにコンテンツを指定するには、<template> に対して v-slot ディレクティブを使って、スロット名を引数として与える。
```HTML
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```
default の部分は、以下のように明示的にしても良い。
```HTML
<template v-slot:default>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
</template>
```
何れにせよ、描画されるHTMlは次のようになる。
```HTML
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```
v-slot は <template> だけに追加できる点に注意。

#### スコープ付きスロット
スロットコンテンツから、子コンポーネントの中だけで利用可能なデータにアクセスしたい場合、例えば、以下のようなテンプレートの <current-user> コンポーネントがあるとすると、
```HTML
<span>
  <slot>{{ user.lastName }}</slot>
</span>
```
ここで、user.lastName を user.firstName に変えたい場合があるかもしれない。
```HTML
<current-user>
  {{ user.firstName }}
</current-user>
```
しかし、user にアクセスできるのが <current-user> コンポーネントだけなので、親コンポーネントで描画することができず、動作しない。これを解決するには、親コンポーネントのスロット属性に user をバインドする。
```HTML
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```
<slot> 要素にバインドされた属性は、スロットプロパティ と呼ばれる。親スコープ内で v-slot の値として名前を指定することで、スロットプロパティを受け取ることができる。
```HTML
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```
* デフォルトスロットしかない場合の省略記法
上の例のようにデフォルトスロット だけの 場合は、コンポーネントのタグをスロットのテンプレートとして使うことができる。つまり、コンポーネントに対して v-slot を直接使える。
```HTML
<current-user v-slot:default="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```
さらに短くすることもでき、未指定のコンテンツがデフォルトスロットのものとみなされるのと同様に、引数のない v-slot もデフォルトコンテンツを参照しているとみなされる。
```html
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```
デフォルトスロットに対する省略記法は、名前付きスロットと混在させることができない。
```HTML
<!-- 不正。警告が出る -->
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
  <template v-slot:other="otherSlotProps">
    slotProps is NOT available here
  </template>
</current-user>
```
* スロットプロパティの分割代入
内部的には、スコープ付きスロットはスロットコンテンツを単一引数の関数で囲むことで動作している。
```js
function (slotProps) {
  // ... slot content ...
}
```
これは、v-slot の値が関数定義の引数部分で有効な任意の JavaScript 式を受け付けることを意味する。そのため、サポートされている環境 (単一ファイルコンポーネント または モダンブラウザ) では、特定のスロットプロパティを取得するために ES2015 の分割代入 を使うこともできる。
```HTML
<current-user v-slot="{ user }">
  {{ user.firstName }}
</current-user>
```
特に、スロットが多くのプロパティを提供している場合に有用。また、プロパティをリネームする (例えば、user から person) など別の可能性も開ける。
```HTML
<current-user v-slot="{ user: person }">
  {{ person.firstName }}
</current-user>
```
スロットプロパティが未定義だった場合のフォールバックを定義することも。
```HTML
<current-user v-slot="{ user = { firstName: 'Guest' } }">
  {{ user.firstName }}
</current-user>
```

#### 動的なスロット名
ディレクティブの動的引数 は v-slot でも動作し、動的なスロット名の定義が可能。
```HTML
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

#### 名前付きスロットの省略記法
v-on や v-bind と同様に v-slot にも省略記法があり、引数の前のすべての部分 (v-slot:) を特別な記号 # で置き換えられる。例えば、v-slot:header は #header に書き換えることができる。
```HTML
<base-layout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```
しかし、ほかのディレクティブと同様に、省略記法は引数がある場合にのみ利用でき、以下のようなものは不正。
```HTML
<!-- これは警告を引き起こす -->
<current-user #="{ user }">
  {{ user.firstName }}
</current-user>
```

 #### その他の例
 スロットプロパティを使えば、入力プロパティに応じて異なるコンテンツを描画する再利用可能なテンプレートにスロットを変えることができる。これは、データロジックをカプセル化する一方で親コンポーネントによるレイアウトのカスタマイズを許すような、再利用可能なコンポーネントを設計しているときに便利。

 例えば以下のように、リストのレイアウトと絞り込みロジックを含む <todo-list> コンポーネントを実装している場合、
 ```HTML
 <ul>
  <li
    v-for="todo in filteredTodos"
    v-bind:key="todo.id"
  >
    {{ todo.text }}
  </li>
</ul>
```
それぞれの todo に対するコンテンツをハードコーディングする代わりに、todo ごとにスロットを作成し、スロットプロパティとして todo をバインドすることで、親コンポーネントから制御できるようにする。
```HTML
<ul>
  <li
    v-for="todo in filteredTodos"
    v-bind:key="todo.id"
  >
    <!--
    それぞれの todo のためのスロットがあり、 `todo` オブジェクトを
    スロットのプロパティとして渡している
    -->
    <slot name="todo" v-bind:todo="todo">
      <!-- フォールバックコンテンツ -->
      {{ todo.text }}
    </slot>
  </li>
</ul>
```
この <todo-list> コンポーネントを利用する時、子からのデータにはアクセスしながらも、todo アイテムに対して代わりの <template> を定義することができる。
```HTML
<todo-list v-bind:todos="todos">
  <template v-slot:todo="{ todo }">
    <span v-if="todo.isComplete">✓</span>
    {{ todo.text }}
  </template>
</todo-list>
```
実世界の強力な利用例については、Vue Virtual Scroller や Vue Promised, Portal Vue といったライブラリを見てみるとよい。

#### 非推奨の構文
* slot 属性による名前付きスロット
* slot-scope 属性によるスコープ付きスロット

## 動的 & 非同期コンポーネント
#### 動的コンポーネントにおける keep-alive の利用
まず、is 属性を利用してタブインタフェースのコンポーネントを切り替えることができる。
```HTML
<component v-bind:is="currentTabComponent"></component>
```
新しいタブに切り替えるたびに、Vue は currentTabComponent の新しいインスタンスを作成する。
そのため、別のタブに切り替えてから戻ってきた場合、最初のタブの操作内容は保持されていない。
最初に作成されたタブコンポーネントのインスタンスがキャッシュされるのが好ましい場合、動的コンポーネントを <keep-alive> 要素でラップすることができる。
```HTML
<!-- インアクティブなコンポーネントはキャッシュされます! -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```
<keep-alive> にラップされるコンポーネントは、コンポーネントの name オプションを使うか、ローカル/グローバル登録を使用して、全て name を持つ必要がある。

#### 非同期コンポーネント
大規模なアプリケーションでは、アプリケーションを小さなまとまりに分割し、必要なコンポーネントだけサーバからロードしたい場合がある。Vue では、コンポーネント定義を非同期で解決するファクトリ関数としてコンポーネントを定義することができ、コンポーネントをレンダリングする必要がある場合にのみファクトリ関数をトリガし、将来の再レンダリングのために結果をキャッシュする。例えば以下。
```js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // resolve コールバックにコンポーネント定義を渡します
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```
ファクトリ関数は、コンポーネント定義をサーバから取得したときに呼び出す必要がある resolve コールバックを受け取る。コンポーネントの取得方法としては、以下のように、非同期コンポーネントと Webpack の code-splitting の機能 を使用するとよい。
```js
Vue.component('async-webpack-example', function (resolve) {
  // この特別な require 構文は、ビルドされたコードを
  // 自動的に Ajax リクエストを介してロードされるバンドルに分割するよう Webpack に指示します
  require(['./my-async-component'], resolve)
})
```
以下、多分Webpackがわかってないとダメなので後回し。

## 特別な問題に対処する
後回し。

## Enter/Leave とトランジション一覧
Vue は、DOM からアイテムが追加、更新、削除されたときにトランジション効果を適用するための方法を複数提供している。
* 自動的に CSS トランジションやアニメーションのためのクラスを適用する。
* Animate.css のようなサードパーティの CSS アニメーションライブラリと連携
* トランジションフックが実行されている間、JavaScript を使って直接 DOM 操作を行う。
* Velocity.js のようなサードパーティの JavaScript アニメーションライブラリと連携する。

#### 単一要素/コンポーネントのトランジション
Vue は、transition ラッパーコンポーネントを提供しており、次のコンテキストにある要素やコンポーネントに entering/leaving トランジションを追加することが可能。
* 条件付きの描画 (v-if を使用)
* 条件付きの表示 (v-show を利用)
* 動的コンポーネント
* コンポーネントルートノード (Component root nodes)
以下のような例が考えられる。
```HTML
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```
```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```
```CSS
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```
transition コンポーネントにラップされた要素が挿入あるいは削除されるとき、次のことが行われる。
1. 対象の要素に CSS トランジションあるいはアニメーションが適用されるか自動的に察知する。適用されない場合、適切なタイミングで、CSS トランジションのクラスを追加/削除する。
2. もし、トランジションコンポーネントが JavaScript フック を提供している場合は、適切なタイミングでそれらのフックが呼ばれる。
3. もし、CSS トランジション/アニメーションが検出されず、JavaScript フックも提供されない場合、挿入、削除のいずれか、あるいは両方の DOM 操作を次のフレームでただちに実行する。ここでのフレームはブラウザのアニメーションフレームを指し、 Vue の nextTick のコンセプトのそれとは異なる。

#### トランジションクラス
enter/leave トランジションのために適用されるクラスは以下の6つ。
1. v-enter: enter の開始状態。要素が挿入される前に適用され、要素が挿入された 1 フレーム後に削除される。
2. v-enter-active: enter の活性状態。トランジションに入るフェーズ中に適用される。要素が挿入される前に追加され、トランジション/アニメーションが終了すると削除される。
3. v-enter-to: バージョン 2.1.8 以降でのみ利用可能な、enter の終了状態。要素が挿入後 (同時に v-enter が削除される。) 1 フレーム後に追加される。トランジション/アニメーションが終了すると削除される。
4. v-leave: leave の開始状態。トランジションの終了がトリガされるとき、直ちに追加され、1フレーム後に削除される。
5. v-leave-active: leave の活性状態。トランジションが終わるフェーズ中に適用される。leave トランジションがトリガされるとき、直ちに追加され、トランジション/アニメーションが終了すると削除される。このクラスは、トランジションの終了に対して、期間、遅延、およびイージングカーブを定義するために使用できる。
6. v-leave-to: バージョン 2.1.8 以降でのみ利用可能な、leave の終了状態。トランジションの終了がトリガされた後 (同時に v-leave が削除される。) ) 1 フレーム後に追加される。トランジション/アニメーションが終了すると削除される。

各クラスは、トランジションの名前が先頭に付く。<transition> 要素に名前がない場合は、デフォルトで v- が先頭に付く。例えば、<transition name="my-transition"> の場合は、v-enter クラスではなく、my-transition-enter となる。

v-enter-active と v-leave-active は、次のセクションの例で見ることができるような、enter/leave トランジションで異なるイージングカーブの指定を可能にする。

#### CSSトランジション
トランジションを実現する最も一般な方法として、以下のようにCSS トランジションが使える。
```HTML
<div id="example-1">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition name="slide-fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```
```js
new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
```
```CSS
/* enter、 leave アニメーションで異なる間隔やタイミング関数を利用することができます */
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
```

#### CSSアニメーション
CSS アニメーションは、CSS トランジションと同じように適用されるが、異なるのは v-enter が要素が挿入された直後に削除されず、animationend イベント時には削除される点。以下は、簡潔にするために CSS ルールの接頭辞を除いた例。
```HTML
<div id="example-2">
  <button @click="show = !show">Toggle show</button>
  <transition name="bounce">
    <p v-if="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>
```
```js
new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
```
```CSS
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```

#### カスタムトランジションクラス
次の属性で、カスタムトランジションクラスを指定できる。
* enter-class
* enter-active-class
* enter-to-class (2.1.8 以降のみ)
* leave-class
* leave-active-class
* leave-to-class (2.1.8 以降のみ)

これらは、クラス名の規約を上書きし,Vue のトランジションシステムと Animate.css のような既存の CSS アニメーションライブラリを組み合わせたいときに特に便利。以下は一例。
```HTML
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">hello</p>
  </transition>
</div>
```
```js
new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
```

####トランジションとアニメーションを両方使う
Vue はトランジションが終了したことを把握するためのイベントリスナのアタッチを必要とする。イベントは、適用される CSS ルールに応じて transitionend か animationend のいずれかのタイプになり、トランジションとアニメーション、どちらか一方だけ使用する場合は、Vue は自動的に正しいタイプを判断することができる。

しかし、例えば、ホバーの CSS トランジション効果と Vue による CSS アニメーションのトリガの両方を持つ場合など、時には、同じ要素に両方を使うこともあるかもしれない。これらのケースでは、Vue に扱って欲しいタイプを type 属性で明示的に宣言するべき。この属性の値は、animation あるいは transition を取る。

#### 明示的なトランジション期間の設定
ほとんどの場合、 Vue は、自動的にトランジションが終了したことを見つけ出すことができる。デフォルトでは、 Vue はルート要素の初めの transitionend もしくは animationend イベントを待つが、例えば、幾つかの入れ子となっている内部要素にてトランジションの遅延がある場合や、ルートのトランジション要素よりも非常に長いトランジション期間を設けている場合の、一連のトランジションのまとまりなどでは、望ましくない。

このような場合 <transition> コンポーネントがもつ duration プロパティを利用することで、明示的に遷移にかかる時間（ミリ秒単位）を指定することが可能。

```HTML
<transition :duration="1000">...</transition>
```
また、活性化時と終了時の期間を、個別に指定することも可能。
```HTML
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

#### JavaScript フック
属性で JavaScript フックを定義することができる。
```HTML
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```
```js
// ...
methods: {
  // --------
  // ENTERING
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // CSS と組み合わせて使う時、done コールバックはオプションです
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // LEAVING
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // CSS と組み合わせて使う時、done コールバックはオプションです
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // v-show と共に使うときだけ leaveCancelled は有効です
  leaveCancelled: function (el) {
    // ...
  }
}
```
これらのフックは、CSS トランジション/アニメーション、または別の何かと組み合わせて使うことができる。
* JavaScript のみを利用したトランジションの場合は、done コールバックを enter と leave フックで呼ぶ必要があり、呼ばない場合は、フックは同期的に呼ばれ、トランジションはただちに終了する。
* JavaScript のみのトランジションのために明示的に v-bind:css="false" を追加すると、Vue に CSS 判定をスキップさせてメモリを節約し、また、誤って CSS ルールがトランジションに干渉するのを防ぐので良い。

```HTML
<!--
Velocity は jQuery.animate と非常によく似ています。
JavaScript アニメーションのための良い選択です。
-->
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="example-4">
  <button @click="show = !show">
    Toggle
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
    v-bind:css="false"
  >
    <p v-if="show">
      Demo
    </p>
  </transition>
</div>
```
```js
new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
```

#### 初期描画時のトランジション
ノードの初期描画時にトランジションを適用したい場合は、appear 属性を追加することができる。
```HTML
<transition appear>
  <!-- ... -->
</transition>
```
デフォルトで、これは entering と leaving のために指定されたトランジションが使用される。あるいは、カスタム CSS クラスを指定することもできる。
```HTML
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8 以降から)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```
そして、カスタム JavaScript フックも指定できる。
```HTML
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```
この例では、appear 属性と v-on:appear フックのどちらも appear トランジションを引き起こす。

#### 要素間のトランジション
v-if/v-else を使った通常の要素同士でもトランジションできる。最も共通の2つの要素のトランジションの例として、リストコンテナとリストが空と説明するメッセージの間で行うものがある。
```HTML
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Sorry, no items found.</p>
</transition>
```
同じタグ名を持つ要素同士でトグルするとき、それらに key 属性を指定することで、個別の要素であることを Vue に伝えなければならない。そうしないと、 Vue のコンパイラは効率化のために要素の内容だけを置き換えようとする。以下は例。
```HTML
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```
このケースでは、他にも key 属性を同一要素の異なる状態のトランジションのために使うこともできる。v-if と v-else を使う代わりに、以下のように書きかえることができる。
```HTML
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```
v-if を複数使ったり、ひとつの要素に対して動的プロパティでバインディングを行ういずれの場合でも、複数個の要素を対象にトランジションすることが可能。
```HTML
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Edit
  </button>
  <button v-if="docState === 'edited'" key="edited">
    Save
  </button>
  <button v-if="docState === 'editing'" key="editing">
    Cancel
  </button>
</transition>
```
これはいかに書き換えられる。
```HTML
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```
```js
// ...
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Edit'
      case 'edited': return 'Save'
      case 'editing': return 'Cancel'
    }
  }
}
```
#### トランジションモード
<transition> のデフォルトの振る舞いとして、entering と leaving は同時に起きる。このため、例えば on ボタンと off ボタンをフェードアウト/インさせたいとき、off は最初 on の隣に現れ、on が消えると on のあった位置にテレポートする。

これは、例えば、位置が絶対位置で指定されているアイテムのトランジションを行うような場合や、スライドのようなトランジションを行う場合には問題にならない。ただ、同時に entering と leaving が行われることは必ずしも望ましくないこともあり、このために Vue は代替となる トランジションモード を提供している。

* in-out: 最初に新しい要素がトランジションして、それが完了したら、現在の要素がトランジションアウトする。
* out-in: 最初に現在の要素がトランジションアウトして、それが完了したら、新しい要素がトランジションインする。

on/offボタン問題を out-in で解決する場合は、以下のようにする。特別なスタイルの追加無しで、ひとつのシンプルな属性を追加するだけでオリジナルのトランジションを修正できる。
```HTML
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```
in-out モードは使用されることは多くないが、見栄え的に使える場面もある。

#### コンポーネント間のトランジション
コンポーネント間のトランジションは、 key 属性が必要ではないのでさらに単純。代わりに、ただ 動的コンポーネント でラップするだけ。
```HTML
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```
```js
new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Component A</div>'
    },
    'v-b': {
      template: '<div>Component B</div>'
    }
  }
})
```
```CSS
.component-fade-enter-active, .component-fade-leave-active {
  transition: opacity .3s ease;
}
.component-fade-enter, .component-fade-leave-to
/* .component-fade-leave-active for below version 2.1.8 */ {
  opacity: 0;
}
```

#### リストトランジション
ここまでで、次のトランジションを扱えるようになった。
* 独立したノード
* 一度に 1 つのみ描画される複数ノード
それでは、例えば、v-for のように同時に描画したいリストのアイテムがある場合はどうすれば良いか。の場合は、<transition-group> コンポーネントを使うことができる。例を詳しく見ていく前に、このコンポーネントについて知っておくべき幾つかの重要なことをあげておく。
* <transition> と異なり、実際に要素を描画する: <span> がデフォルト。この要素は、tag 属性で変えることができる。
* トランジションモード は利用できない。もはや相互に排他的な要素を交互に利用する事が無いため。
* 中の要素は、key 属性を持つことが必須。

#### リスト Entering/Leaving トランジション
シンプルな例をみてみる。これまでに使っていたものと同じ CSS クラスを entering と leaving のトランジションで使う。
```HTML
<div id="list-demo">
  <button v-on:click="add">Add</button>
  <button v-on:click="remove">Remove</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>
```
```js
new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
```
```CSS
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to /* .list-leave-active for below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}

```

#### リスト移動トランジション
<transition-group> コンポーネントは、entering と leaving のアニメーションだけでなく、位置の変化も同様にアニメーションできる。この新しい機能を使うために知らないといけない新しいコンセプトは、アイテムの位置が変わる時に追加される v-move クラスだけ。他のクラスと同様に、接頭辞は name 属性値と一致し、move-class 属性でクラスを指定することもできる。
以下で分かるように、このクラスは主にトランジションのタイミングやイージングカーブを指定するのに便利。
```HTML
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="flip-list-demo" class="demo">
  <button v-on:click="shuffle">Shuffle</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" v-bind:key="item">
      {{ item }}
    </li>
  </transition-group>
</div>
```
```js
new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```
```CSS
.flip-list-move {
  transition: transform 1s;
}
```
内部で Vue は transforms を使って、前の位置から新しい位置へ要素を滑らかにトランジションさせるために FLIP と呼ばれる単純なアニメーションテクニックを使っているらしい。

これまでの実装とこのテクニックを組み合わせることで、リストに対する全ての変更をアニメーションさせることができる。
```HTML
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="list-complete-demo" class="demo">
  <button v-on:click="shuffle">Shuffle</button>
  <button v-on:click="add">Add</button>
  <button v-on:click="remove">Remove</button>
  <transition-group name="list-complete" tag="p">
    <span
      v-for="item in items"
      v-bind:key="item"
      class="list-complete-item"
    >
      {{ item }}
    </span>
  </transition-group>
</div>
```
```js
new Vue({
  el: '#list-complete-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```
```css
.list-complete-item {
  transition: all 1s;
  display: inline-block;
  margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active for below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}
```

一点、FLIP トランジションは display: inline が指定されていると動かない。代わりに display: inline-block を使うか、flex コンテキストに要素を置き換えることで動かすことができる。

FLIP アニメーションは、単一の軸だけに限定されるものではなく、多次元のグリッドにあるアイテムも同じように簡単にトランジションできる。

#### スタッガリングリストトランジション
data 属性を介して、JavaScript トランジションとやりとりを行うことで、リスト内の遷移をずらすことが可能。
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="staggered-list-demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>
```
```js
new Vue({
  el: '#staggered-list-demo',
  data: {
    query: '',
    list: [
      { msg: 'Bruce Lee' },
      { msg: 'Jackie Chan' },
      { msg: 'Chuck Norris' },
      { msg: 'Jet Li' },
      { msg: 'Kung Fury' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})
```

#### トランジションの再利用
Vue のコンポーネントシステムを通して、トランジションを再利用することができる。再利用できるトランジションを生成するには、ルートに <transition> や <transition-group> を配置して、トランジションコンポーネントに子を渡す。以下は例。
```js
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```
関数型コンポーネントも使える。
```js
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```
#### 動的トランジション
動的トランジションの最も基本的な例は、name 属性を動的プロパティとして束縛すること。
```HTML
<transition v-bind:name="transitionName">
  <!-- ... -->
</transition>
```
これは、Vue のトランジションクラス規約を使って CSS トランジション/アニメーションを定義したとき、それらをシンプルに切り替える場合に便利。

任意のトランジション属性を動的に束縛できるが、それは属性だけに限らない。イベントフックはメソッドなので、コンテキストのいかなるデータにもアクセスできる。これは、コンポーネントの状態に応じて、JavaScript トランジションが異なる振る舞いをすることを意味する。

```HTML
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="dynamic-fade-demo" class="demo">
  Fade In: <input type="range" v-model="fadeInDuration" min="0" v-bind:max="maxFadeDuration">
  Fade Out: <input type="range" v-model="fadeOutDuration" min="0" v-bind:max="maxFadeDuration">
  <transition
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">hello</p>
  </transition>
  <button
    v-if="stop"
    v-on:click="stop = false; show = false"
  >Start animating</button>
  <button
    v-else
    v-on:click="stop = true"
  >Stop it!</button>
</div>
```
```js
new Vue({
  el: '#dynamic-fade-demo',
  data: {
    show: true,
    fadeInDuration: 1000,
    fadeOutDuration: 1000,
    maxFadeDuration: 1500,
    stop: true
  },
  mounted: function () {
    this.show = false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function () {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function () {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})
```

## 状態のトランジション
Vue のトランジションシステムは entering、leaving、およびリストをアニメーションさせるための多くの単純な方法を提供するが、データ自体をアニメーションさせる場合はどうか。例えば以下の場合。
* 数値と計算
* 表示される色
* SVG ノードの位置
* 要素のサイズやその他のプロパティ

これらはすべて、生の数値として保持されているか、あるいは数値に変換することが可能一度それをすれば、Vue のリアクティブな性質とコンポーネントシステムの組み合わせにより、状態から中間フレームを生成するためのサードパーティのライブラリを使ってアニメーションさせることができる。

## ウォッチャによる状態のアニメーション
ウォッチャは、任意の数値プロパティの変化によって他のプロパティをアニメーションさせることができるようにする。以下は GreenSock を使った例。

```HTML
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/1.20.3/TweenMax.min.js"></script>

<div id="animated-number-demo">
  <input v-model.number="number" type="number" step="20">
  <p>{{ animatedNumber }}</p>
</div>
```
```js
new Vue({
  el: '#animated-number-demo',
  data: {
    number: 0,
    tweenedNumber: 0
  },
  computed: {
    animatedNumber: function() {
      return this.tweenedNumber.toFixed(0);
    }
  },
  watch: {
    number: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedNumber: newValue });
    }
  }
})
```
数値を更新すると、その変更が入力の下でアニメーションする。例えば有効な CSS の色のように、直接的に数値として保持されていないものの場合は、Tween.js と Color.js を加えることによってアニメーションができる。
```HTML
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>
<script src="https://cdn.jsdelivr.net/npm/color-js@1.0.3"></script>

<div id="example-7">
  <input
    v-model="colorQuery"
    v-on:keyup.enter="updateColor"
    placeholder="Enter a color"
  >
  <button v-on:click="updateColor">Update</button>
  <p>Preview:</p>
  <span
    v-bind:style="{ backgroundColor: tweenedCSSColor }"
    class="example-7-color-preview"
  ></span>
  <p>{{ tweenedCSSColor }}</p>
</div>
```
```js
var Color = net.brehaut.Color

new Vue({
  el: '#example-7',
  data: {
    colorQuery: '',
    color: {
      red: 0,
      green: 0,
      blue: 0,
      alpha: 1
    },
    tweenedColor: {}
  },
  created: function () {
    this.tweenedColor = Object.assign({}, this.color)
  },
  watch: {
    color: function () {
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween(this.tweenedColor)
        .to(this.color, 750)
        .start()

      animate()
    }
  },
  computed: {
    tweenedCSSColor: function () {
      return new Color({
        red: this.tweenedColor.red,
        green: this.tweenedColor.green,
        blue: this.tweenedColor.blue,
        alpha: this.tweenedColor.alpha
      }).toCSS()
    }
  },
  methods: {
    updateColor: function () {
      this.color = new Color(this.colorQuery).toRGB()
      this.colorQuery = ''
    }
  }
})
```
```css
.example-7-color-preview {
  display: inline-block;
  width: 50px;
  height: 50px;
}
```

#### 動的な状態のトランジション
Vue のトランジションコンポーネントを使う場合と同様に、状態のトランジションの背後にあるデータはリタルタイムに更新でき、特にプロトタイピングにおいて便利。

#### コンポーネント内のトランジションの整理
多くの状態のトランジションを管理することで、Vue インスタンスやコンポーネントの複雑さをが増加する。多くのアニメーションは専用の子コンポーネントに抽出することができ、親インスタンスの複雑さを解消できる。以下は例。
```HTML
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>

<div id="example-8">
  <input v-model.number="firstNumber" type="number" step="20"> +
  <input v-model.number="secondNumber" type="number" step="20"> =
  {{ result }}
  <p>
    <animated-integer v-bind:value="firstNumber"></animated-integer> +
    <animated-integer v-bind:value="secondNumber"></animated-integer> =
    <animated-integer v-bind:value="result"></animated-integer>
  </p>
</div>
```
```js
// この複雑な中間フレーム生成ロジックは現在、アプリケーション内の
// アニメーションさせたいと望むあらゆる数値で再利用できます。
// コンポーネントは、よりダイナミックなトランジションと複雑な
// トランジション戦略を構成するクリーンなインターフェイスを提供します。
Vue.component('animated-integer', {
  template: '<span>{{ tweeningValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data: function () {
    return {
      tweeningValue: 0
    }
  },
  watch: {
    value: function (newValue, oldValue) {
      this.tween(oldValue, newValue)
    }
  },
  mounted: function () {
    this.tween(0, this.value)
  },
  methods: {
    tween: function (startValue, endValue) {
      var vm = this
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween({ tweeningValue: startValue })
        .to({ tweeningValue: endValue }, 500)
        .onUpdate(function () {
          vm.tweeningValue = this.tweeningValue.toFixed(0)
        })
        .start()

      animate()
    }
  }
})

// すべての複雑さがメインの Vue インスタンスから取り除かれました！
new Vue({
  el: '#example-8',
  data: {
    firstNumber: 20,
    secondNumber: 40
  },
  computed: {
    result: function () {
      return this.firstNumber + this.secondNumber
    }
  }
})
```
## ミックスイン
Vue コンポーネントに再利用可能で柔軟性のある機能を持たせるための方法。ミックスインオブジェクトは任意のコンポーネントオプションを含み、コンポーネントがミックスインを使用するとき、ミックスインの全てのオプションはコンポーネント自身のオプションに”混ぜられ”る。
```js
// ミックスインオブジェクトを定義
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// このミックスインを使用するコンポーネントを定義
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```

#### オプションのまーじ
ミックスインとコンポーネントそれ自身がオプションと重複するとき、それらは適切なストラテジを使用して”マージ”される。例えば、データオブジェクトは再帰的マージされ、コンフリクトした場合にはコンポーネントのデータが優先される。
```js
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```

同じ名前のフック関数はそれら全てが呼び出されるよう配列にマージされる。ミックスインのフックはコンポーネント自身のフック前に呼ばれる。
```js
var mixin = {
  created: function () {
    console.log('mixin hook called')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('component hook called')
  }
})

// => "mixin hook called"
// => "component hook called"
```
オブジェクトの値を期待するオプション、例えば、methods、components、そして directives らは同じオブジェクトにマージされる。コンポーネントオプションはこれらのオブジェクトでキーのコンフリクトがあるとき、優先される。Vue.extend() でも同じ。
```js
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})

vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
```

#### グローバルミックスイン
グローバルにミックスインを適用することもできるが、一度、グローバルにミックスインを適用すると、それはその後に作成する全ての Vue インスタンスに影響することに注意。これはカスタムオプションに対して処理ロジックを注入するために使用することができる。
```js
// `myOption` カスタムオプションにハンドラを注入する
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```
多くのケースでは、上記の例のような、カスタムオプションを処理するようなものに使用すべきであり、重複適用を避けるため プラグイン として作成することが推奨される。

#### カスタムオプションのマージストラテジ
カスタムオプションがマージされるとき、それらは単純に既存の値を上書きするデフォルトのストラテジを使用する。
カスタムロジックを使用してカスタムオプションをマージする場合、Vue.config.optionMergeStrategies をアタッチする必要がある。
```js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // マージされた値を返す
}
```
ほとんどのオブジェクトベースのオプションでは、単純に methods で使用されるのと同じストラテジを使用することができる。
```js
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```
より高度な例として Vuex 1.x のマージストラテジがある。
```js
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```

## カスタムディレクティブ
#### 基本
Vue.js 本体で出荷されたディレクティブの標準セットに加えて (v-model と v-show)、カスタムディレクティブ (custom directive) を登録することができる。
Vue 2.0 では、コードの再利用と抽象化における基本の形はコンポーネントであるが、通常の要素で低レベル DOM にアクセスしなければならないケースもあり得る。
こういった場面では、カスタムディレクティブが役立つ。例として、input 要素へのフォーカスが挙げられる。

ページを読み込むと、input 要素はフォーカスを手に入れる。実際、このページに訪れてから他のところをクリックしていなければ、この input にフォーカス(注意:モバイル Safari では自動でフォーカスしない。)が当たっているはず。これを実現するディレクティブは以下。
```js
// `v-focus` というグローバルカスタムディレクティブを登録します
Vue.directive('focus', {
  // ひも付いている要素が DOM に挿入される時...
  inserted: function (el) {
    // 要素にフォーカスを当てる
    el.focus()
  }
})
```

代わりにローカルディレクティブに登録したいならば、コンポーネントの directives オプションで登録できる。
```js
directives: {
  focus: {
    // ディレクティブ定義
    inserted: function (el) {
      el.focus()
    }
  }
}
```
これでテンプレートで、以下のような新しい v-focus 属性が使えるようになる。
```HTML
<input v-focus>
```

#### フック関数
directive definition object はいくつかのフック関数(全て任意)を提供する。
* bind: ディレクティブが初めて対象の要素にひも付いた時に 1 度だけ呼ばれる。ここで 1 回だけ実行するセットアップ処理を行える。
* inserted: ひも付いている要素が親 Node に挿入された時に呼ばれる。(これは、親 Node が存在している時にだけ保証され、必ずしもドキュメントにあるとは限らない)。
* update: ひも付いた要素を抱合しているコンポーネントの VNode が更新される度に呼ばれる。多くの場合子コンポーネントが更新される前。 ディレクティブの値が変化してもしなくても、バインディングされている値と以前の値との比較によって不要な更新を回避することができる。
* componentUpdated: 抱合しているコンポーネントの VNode と子コンポーネントの VNode が更新された後に呼ばれる。
* unbind: ディレクティブがひも付いている要素から取り除かれた時に 1 度だけ呼ばれる。

#### ディレクティブフック引数
ディレクティブフックには以下の引数が渡せる。
1. el: ディレクティブがひも付く要素。DOM を直接操作するために使用できる。
2. binding: 以下のプロパティを含んでいるオブジェクト。
* name: v- 接頭辞 (prefix) 無しのディレクティブ名。
* value: ディレクティブに渡される値。例えば v-my-directive="1 + 1" では、value は 2 。
* oldValue: update と componentUpdated においてのみ利用できる以前の値。値が変化したかどうかに関わらず利用できる。
expression: 文字列としてのバインディング式。例えば v-my-directive="1 + 1" では、式は "1 + 1" 。
* arg: もしあれば、ディレクティブに渡される引数。例えば v-my-directive:foo では、arg は "foo" 。
* modifiers: もしあれば、修飾子 (modifier) を含んでいるオブジェクト。例えば v-my-directive.foo.bar では、modifiers オブジェクトは { foo: true, bar: true } 。
3. vnode: Vue のコンパイラによって生成される仮想ノード。
4. oldVnode: update と componentUpdated フックにおいてのみ利用できる以前の仮想ノード。

el を除いて、これらの全てのプロパティは読み込みのみ (read-only) 扱わなくてはならない。フックを超えてデータを共有する必要がある場合は, 要素の dataset を通じて行うことが推奨されている。

以下、いくつかのプロパティを使用したカスタムディレクティブの例。
```HTML
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```
```js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

#### 動的なディレクティブ引数
例えば、v-mydirective:[argument]="value" において、argument はコンポーネントインスタンスの data プロパティに基づいて更新される。
これにより、アプリケーション内でのカスタムディレクティブの利用が柔軟になる。

要素をページの固定位置にピン留めするカスタムディレクティブを作りたいとして、縦方向のピクセル位置を値で設定するカスタムディレクティブを次のように作ることができる。
```HTML
<div id="baseexample">
  <p>Scroll down the page</p>
  <p v-pin="200">Stick me 200px from the top of the page</p>
</div>
```
```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  }
})

new Vue({
  el: '#baseexample'
})
```

#### 動的なディレクティブ引数
これにより、ページの上端から 200px の位置に要素を固定できる。的引数はコンポーネントのインスタンス毎に更新できるので、上端を下端に変えたくなった場合などに便利。
```HTML
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
```
```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

#### 関数による省略記法
多くの場合、bind と update には同じ振舞いが欲しいが、その他のフックに関しては気にかけない。以下は例。
```js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

#### オブジェクトリテラル
ディレクティブが複数の値を必要ならば、JavaScript オブジェクトリテラルも渡すこともできる。ディレクティブは任意の妥当な JavaScript 式を取ることができる。
```HTML
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```
```js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```

##描画関数とJSX
#### 基本
Vue ではほとんどの場合 HTML をビルドするためにテンプレートを使うことが推奨される。
しかし、JavaScript による完全なプログラミングパワーを必要とする状況もあり、そこでテンプレートへの代替で、よりコンパイラに近い 描画 (render) 関数が使用できる。

render 関数が実用的になりうる簡単な例として、アンカーヘッダを生成したいとする。
```HTML
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```
上記の HTML に対して、望ましいコンポーネントのインターフェイスを決めたい。以下はダメな例。
```HTML
<anchored-heading :level="1">Hello world!</anchored-heading>
```
```HTML
<script type="text/x-template" id="anchored-heading-template">
  <h1 v-if="level === 1">
    <slot></slot>
  </h1>
  <h2 v-else-if="level === 2">
    <slot></slot>
  </h2>
  <h3 v-else-if="level === 3">
    <slot></slot>
  </h3>
  <h4 v-else-if="level === 4">
    <slot></slot>
  </h4>
  <h5 v-else-if="level === 5">
    <slot></slot>
  </h5>
  <h6 v-else-if="level === 6">
    <slot></slot>
  </h6>
</script>
```
```js
Vue.component('anchored-heading', {
  template: '#anchored-heading-template',
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```
冗長なだけでなく、全てのヘッダレベルで <slot></slot> を重複させており、アンカー要素を追加したい時に同じことをする必要がある。テンプレートはほとんどのコンポーネントでうまく動作しますが、この例はそうではないことが明確。render 関数で改善できる。
```js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // タグ名
      this.$slots.default // 子の配列
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```
このケースでは、 v-slot ディレクティブ無しでコンポーネントに子を渡した時に ( anchored-heading の内側の Hello world! のような) 、それらの子がコンポーネントインスタンス上の $slots.default にストアされていることを知っている必要がある。

#### ノード、ツリー、および仮想 DOM
render 関数を説明する前に、ブラウザの仕組みについて少し知っておくことが重要。
例えば、以下のHTMLがあったとする。
```HTML
<div>
  <h1>My title</h1>
  Some text content
  <!-- TODO: Add tagline  -->
</div>
```
ブラウザはこのコードを読み込むと、血縁関係を追跡するために家系図を構築するのと同じように、全てを追跡する “DOM ノード”ツリーを構築する。このコードの場合は以下になる。
![http_responce](images/dom-tree.png)

ノードとは全ての要素、全てのテキストであり、そして全てのコメントでさえも指す。
これらのノードの更新は手動で行う必要がなく、テンプレート内のどの HTML をページに表示するかを Vue に伝えるだけでよい。以下。
```HTML
<h1>{{ blogTitle }}</h1>
```
render 関数を使う場合は以下。
```js
render: function (createElement) {
  return createElement('h1', this.blogTitle)
}
```
どちらの場合でも、blogTitleが変更されるとき、Vue は自動的にページを更新する。

#### 仮想DOM
Vue は、実際の DOM に加える必要がある変更を追跡する仮想 DOM を構築する。上の例では以下の部分。
```js
return createElement('h1', this.blogTitle)
```
createElement が返すのは正確には実際の DOM 要素ではなく、どのノードを描画するかを記述した情報が子ノードの記述を含んで Vue に含まれているため、より正確には createNodeDescription という名前になる。このノード記述は”仮想ノード”と呼ばれ、通常は VNode と略される。”仮想 DOM” は、 VNode のツリー全体を指す。

#### createElement 引数
createElement 関数の中でテンプレートの機能をどのように使うかを見てみる。
```js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // HTML タグ名、コンポーネントオプション、もしくは
  // そのどちらかを解決する非同期関数です。必須です。
  'div',

  // {Object}
  // テンプレート内で使うであろう属性に対応する
  // データオブジェクト。任意です。
  {
    // (詳細は下の次のセクションをご参照ください)
  },

  // {String | Array}
  // 子のVNode。`createElement()` を使用して構築するか、
  // テキスト VNode の場合は単に文字列を使用します。任意です。
  [
    'Some text comes first.',
    createElement('h1', 'A headline'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```
#### データオブジェクト詳解
v-bind:class と v-bind:style がテンプレート内で特殊な扱いをされているのと同じように、それらは VNode のデータオブジェクト内で自身のトップレベルフィールドを持つ。このオブジェクトは innerHTML のような通常の HTML 属性だけでなく DOM プロパティもバインドすることができる。つまり、v-html ディレクティブに取って代わる。
```js
{
  // `v-bind:class` と同じ API、いずれかを受け付ける
  // 文字列、オブジェクトまたは文字列とオブジェクトの配列
  class: {
    foo: true,
    bar: false
  },
  // `v-bind:style` と同じ API、いずれかを受け付ける
  // 文字列、オブジェクトまたはオブジェクトの配列
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 通常の HTML 属性
  attrs: {
    id: 'foo'
  },
  // コンポーネントプロパティ
  props: {
    myProp: 'bar'
  },
  // DOM プロパティ
  domProps: {
    innerHTML: 'baz'
  },
  // イベントハンドラは `on` の配下に置かれます。
  // ただし、 `v-on:keyup.enter` などの修飾詞はサポートされません。
  // その代わり、手動で keyCode をハンドラの中で
  // 確認する必要があります。
  on: {
    click: this.clickHandler
  },
  // コンポーネントの場合のみ。
  // `vm.$emit` を使ってコンポーネントから emit されるイベントではなく
  // ネイティブのイベントを listen することができます。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // カスタムディレクティブ。`binding` の `oldValue` については
  // あなたの代わりに Vue が面倒を見るので、設定することはできません。
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // { name: props => VNode | Array<VNode> }
  // の 形式でのスコープ付きスロット
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // このコンポーネントが他のコンポーネントの子の場合は、そのスロット名
  slot: 'name-of-slot',
  // 他の特殊なトップレベルのプロパティ
  key: 'myKey',
  ref: 'myRef',
  // If you are applying the same ref name to multiple
  // elements in the render function. This will make `$refs.myRef` become an
  // array
  refInFor: true
}
```

#### 完全な例
ようやく最初のコンポーネントが完成させられる。
```js
var getChildrenTextContent = function (children) {
  return children.map(function (node) {
    return node.children
      ? getChildrenTextContent(node.children)
      : node.text
  }).join('')
}

Vue.component('anchored-heading', {
  render: function (createElement) {
    // kebab-case id の作成
    var headingId = getChildrenTextContent(this.$slots.default)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^-|-$)/g, '')

    return createElement(
      'h' + this.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, this.$slots.default)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

#### 制約
コンポーネントツリーの中で全ての VNode は一意でなければならず、以下のような render 関数は不正。
```js
render: function (createElement) {
  var myParagraphVNode = createElement('p', 'hi')
  return createElement('div', [
    // うわ - 重複の VNodes ！
    myParagraphVNode, myParagraphVNode
  ])
}
```
もし同じ要素 / コンポーネントを何度も重複させたい場合には、factory 関数を使うことで対応できる。例えば、次の render 関数は 20 個の一意に特定できるパラグラフを描画する。
```js
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```

#### 素の JavaScript によるテンプレートの書き換え
* v-if と v-for
```HTML
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```
例えば、この v-if と v-for を使ったテンプレートは、 render 関数においては JavaScript の if / else と map を使って書き換えることができる。
```js
props: ['items'],
render: function (createElement) {
  if (this.items.length) {
    return createElement('ul', this.items.map(function (item) {
      return createElement('li', item.name)
    }))
  } else {
    return createElement('p', 'No items found.')
  }
}
```
* v-model
また、v-model に関しては、描画関数には直接的な v-model の対応がないため、自身でロジックを実装する必要がある。
```js
props: ['value'],
render: function (createElement) {
  var self = this
  return createElement('input', {
    domProps: {
      value: self.value
    },
    on: {
      input: function (event) {
        self.$emit('input', event.target.value)
      }
    }
  })
}
```
v-model と比較してインタラクションをより詳細に制御することもできる。

* イベントとキー修飾子
次に、.passive、.capture と .once イベント修飾子に対して、Vue は on で使用できる接頭辞を提供している。
|修飾子|接頭辞|
|.passive|&|
|.capture|!|
|.once|~|
|.capture.once|~!|
|.once.capture|~!|

以下使用例。
```js
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  '~!mouseover': this.doThisOnceInCapturingMode
}
```
全ての他のイベントとキー修飾子に対しては、単にハンドラ内のイベントメソッドを使用できるため、独自の接頭辞は必要ない。

|修飾子|相当するイベントメソッド|
|.stop|event.stopPropagation()|
|.prevent|event.preventDefault()|
|.self|if (event.target !== event.currentTarget) return|
|キー: .enter, .13|if (event.keyCode !== 13) return (他のキー修飾子に対して、13 を別のキーコードに変更する)|
|修飾子キー: .ctrl, .alt, .shift, .meta|if (!event.ctrlKey) return (ctrlKey を altKey、shiftKey、または metaKey、それぞれ変更する)|

以下、これらの修飾子がすべて一緒に使用されている例。
```js
on: {
  keyup: function (event) {
    // イベントを発行している要素が、
    // イベントが束縛されている要素でない場合は中止します。
    if (event.target !== event.currentTarget) return
    // up したキーが enter キー (13) でなく、
    // shift キーが同時に押されていない場合は中止します。
    if (!event.shiftKey || event.keyCode !== 13) return
    // イベントの伝播を停止します。
    event.stopPropagation()
    // この要素に対する、デフォルトの keyup ハンドラを無効にします。
    event.preventDefault()
    // ...
  }
}
```
* スロット
また、this.$slots から VNode の配列として静的なスロットの内容にアクセスできる。
```js
render: function (createElement) {
  // `<div><slot></slot></div>`
  return createElement('div', this.$slots.default)
}
```
そして、this.$scopedSlots から VNode を返す関数としてスコープ付きスロットにアクセスできる。
```js
props: ['message'],
render: function (createElement) {
  // `<div><slot :text="message"></slot></div>`
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.message
    })
  ])
}
```
スコープ付きスロットを描画関数を使って子コンポーネントに渡すには、VNode データの scopedSlots フィールドを使う。
```js
render: function (createElement) {
  return createElement('div', [
    createElement('child', {
      // { name: props => VNode | Array<VNode> } の形式で
      // `scopedSlots` を データオブジェクトに渡す
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
```

#### JSX
```js
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)
```
この render 関数は、テンプレートなら以下のように簡潔になる。
```HTML
<anchored-heading :level="1">
  <span>Hello</span> world!
</anchored-heading>
```
そのような理由から Vue で JSX を使うための Babel プラグイン がある。
```js
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render: function (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```
function(h) の h　は、createElement のエイリアス。Vue の Babel プラグインの バージョン 3.4.0 以降では、ES2015 のシンタックスで宣言された JSX を含むメソッドや getter（関数やアロー関数は対象外）に対しては、自動的に const h = this.$createElement が注入されるため、(h) パラメーターは省略できる。それ以前のバージョンでは、もし h がそのスコープ内で利用可能でない場合、アプリケーションはエラーを throw する。

#### 関数型コンポーネント
先ほど作成したアンカーヘッダコンポーネントは、状態の管理や渡された状態の watch をしておらず、また、何もライフサイクルメソッドを持ないので、比較的シンプル。いくつかのプロパティを持つただの関数である。

このような場合、コンポーネントを関数型としてマークできる。状態を持たない (リアクティブデータが無い) でインスタンスを持たない ( this のコンテキストが無い) ことを意味する。関数型コンポーネントの形式は以下。
```js
Vue.component('my-component', {
  functional: true,
  // プロパティは任意です
  props: {
    // ...
  },
  // インスタンスが無いことを補うために、
  // 2 つ目の context 引数が提供されます。
  render: function (createElement, context) {
    // ...
  }
})
```
2.5.0以降では、単一ファイルコンポーネントを使用している場合、テンプレートベースの関数型コンポーネントは次のように宣言できる。
```HTML
<template functional>
</template>
```
コンポーネントが必要とする全てのものは context を通して渡され、context は以下の要素を持つ。
* props: 提供されるプロパティのオブジェクト
* children: 子 VNode の配列
* slots: slots オブジェクトを返す関数
* scopedSlots: (2.6.0 以降) スコープ付きスロットを公開するオブジェクト。通常のスロットも関数として公開します
* data: createElement の第 2 引数としてコンポーネントに渡される全体のデータオブジェクト
* parent: 親コンポーネントへの参照
* listeners: (2.3.0 以降) 親に登録されたイベントリスナーを含むオブジェクト。これは単純に data.on のエイリアスです
* injections: (2.3.0 以降) inject オプションで使用する場合、これは解決されたインジェクション(注入)が含まれます

functional: true を追加した後、アンカーヘッダコンポーネントの render 関数の更新のためには、context 引数の追加、this.$slots.default の context.children への更新、this.level の context.props.level への更新が必要になる。

関数型コンポーネントはただの関数なので、描画コストは少なく、以下が必要な時などのラッパーコンポーネントとしてもとても便利。
* いくつかの他のコンポーネントから 1 つ委譲するためのものをプログラムで選ぶ時
* 子、プロパティ、または、データを子コンポーネントへ渡る前に操作したい時

以下は、渡されるプロパティに応じてより具体的なコンポーネントに委譲する smart-list コンポーネントの例。
```js
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  },
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  }
})
```

* 子要素/コンポーネントへの属性およびイベントの受け渡し
通常のコンポーネントでは、props として定義されていない属性はコンポーネントのルート要素に自動的に追加され、同名の既存の属性と置換または賢くマージされる。

ところが、関数型コンポーネントでは、この動作を明示的に定義する必要がある。
```js
Vue.component('my-functional-button', {
  functional: true,
  render: function (createElement, context) {
    // 任意の属性、イベントリスナ、子などを透過的に渡します。
    return createElement('button', context.data, context.children)
  }
})
```
context.data を createElement の第2引数として渡すことで、my-function-button で使用された属性やイベントリスナを渡している。実際、イベントは .native 修飾子を必要としないため、とても透過的。

テンプレートベースの関数型コンポーネントを使用している場合、属性とリスナーも手動で追加する必要がある。個々のコンテキストの内容にアクセスできるので、イベントリスナを渡すための listeners (data.on のエイリアス) と HTML の属性を渡すための data.attrs を使用することができる。
```HTML
<template functional>
  <button
    class="btn btn-primary"
    v-bind="data.attrs"
    v-on="listeners"
  >
    <slot/>
  </button>
</template>
```
* slots() vs children
slots().default は children と同じようで、同じでない場合がある。以下の子を持つ関数型コンポーネントの場合とか。
```HTML
<my-functional-component>
  <p v-slot:foo>
    first
  </p>
  <p>second</p>
</my-functional-component>
```
このコンポーネントの場合、 children は両方のパラグラフが与えられ、slots().default は 2 つ目のものだけ与えられる。したがって、 children と slots() の両方を持つことで このコンポーネントが slot システムを知っているか、もしくは、単純に children を渡しておそらく他のコンポーネントへその責任を委譲するかどうかを選べるようになる。

## プラグイン
プラグインは通常 Vue にグローバルレベルで機能を追加する。プラグインに対しては厳密に定義されたスコープはなく、以下のようなタイプに分かれる。
1. 1つ、または複数のグローバル・メソッドを追加する。例: vue-custom-element

2. 1つ、または複数のグローバル・アセットを追加する。ディレクティブ/フィルタ/トランジションなど。例: vue-touch

3. グローバル・ミックスインにより、1つ、または複数のコンポーネントオプションを追加する。例: vue-router

4. Vue.prototype に記述することにより、1つまたは、複数の Vue インスタンスメソッドを追加する。

5. 同時に上記のいくつかの組み合わせを注入しながら、独自の API を提供するライブラリ。例: vue-router

#### プラグインの使用
Vue.use() グローバルメソッドを呼び出すことによってプラグインを使用する。これは new Vue() を呼び出してアプリを起動するよりも前に行われる必要がある。
```js

// `MyPlugin.install(Vue)` を呼び出します
Vue.use(MyPlugin)

new Vue({
  // ... オプション
})
```
いくつかのオプションに任意で渡すことができる。
```js
Vue.use(MyPlugin, { someOption: true })
```
Vue.use は、同じプラグインを 2 回以上使用することを自動的に防ぐ。のため、同じプラグインを同時に複数回呼び出しても、一度しかそのプラグインをインストールしない。vue-router のような Vue.js 公式プラグインによって提供されるプラグインは、Vue がグローバル変数として使用可能な場合、自動的に Vue.use() を呼ぶ。しかしながら、 CommonJS のようなモジュール環境では、常に明示的に Vue.use() を呼ぶ必要がある。
```js
// Browserify または Webpack 経由で CommonJS を使用
var Vue = require('vue')
var VueRouter = require('vue-router')

// これを呼びだすのを忘れてはいけません
Vue.use(VueRouter)
```

#### プラグインの記述
Vue.js プラグインは install メソッドを公開する必要があり、第 1 引数は Vue コンストラクタ、第 2 引数は任意で options が指定されて呼び出される。
```js
MyPlugin.install = function (Vue, options) {
  // 1. グローバルメソッドまたはプロパティを追加
  Vue.myGlobalMethod = function () {
    // 何らかのロジック ...
  }
  // 2. グローバルアセットを追加
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 何らかのロジック ...
    }
    ...
  })
  // 3. 1つ、または複数のコンポーネントオプションを注入
  Vue.mixin({
    created: function () {
      // 何らかのロジック ...
    }
    ...
  })
  // 4. インスタンスメソッドを追加
  Vue.prototype.$myMethod = function (methodOptions) {
    // 何らかのロジック ...
  }
}
```

## フィルター
Vue.js では、一般的なテキストフォーマットを適用するために使用できるフィルタを定義できます。フィルタは、mustache 展開と v-bind 式、2 つの場所で使用できる。。フィルタは JavaScript 式の末尾に追加する必要があり、これは “パイプ(‘|’)” 記号で表される。
```HTML
<!-- mustaches -->
{{ message | capitalize }}

<!-- v-bind -->
<div v-bind:id="rawId | formatId"></div>
```
コンポーネントのオプション内でローカルフィルタを定義できる。
```js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```
もしくは Vue インスタンスを作成する前にグローバルでフィルタを定義することも出来る。
```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```
グローバルフィルタがローカルフィルタと同じ名前をもつ場合、ローカルフィルタでオーバーライドされる。

フィルタ関数は常に式の値(前のチェーンの結果)を第一引数として受け取る。

フィルタは連結できる。
```HTML
{{ message | filterA | filterB }}
```

この場合、単一の引数で定義された filterA は message の値を受け取り、filterBの単一引数に filterA の結果を渡して filterB 関数が呼び出される。
フィルタは JavaScript 関数なので、引数を取ることができる。
```HTML
{{ message | filterA('arg1', arg2) }}
```
ここで filterA は3つの引数をとる関数として定義されている。message の値は最初の引数に渡され、プレーン文字列 'arg1' は第2引数として filterA に渡され、 arg2 の値が評価され、第3引数としてフィルタに渡される。

## 単一ファイルコンポーネント
#### 前書き
多くの Vue プロジェクトでは、グローバルコンポーネントは、new Vue({ el: '#container '}) の後に各ページの body においてコンテナ要素をターゲットにすることに続いて、Vue.component を使用して定義されている。

これは view を拡張するだけに利用された小さな中規模プロジェクトにおいてはとても有効である一方、フロントエンドで JavaScript 全体を操作するようなもっと複雑なプロジェクトでは、以下の点において不利益になる。
* グローバル宣言は全てのコンポーネントに一意な名前を強制すること
* シンタックスハイライトの無い文字列テンプレートと複数行 HTML の時に醜いスラッシュが強要されること
* CSS サポート無しだと、 HTML と JavaScript がコンポーネントにモジュール化されている間、これ見よがしに無視されること
* ビルド処理がないと Pug (前 Jade) や Babel のようなプリプロセッサよりむしろ、 HTML や ES5 JavaScript を制限されること

これら全ては Webpack や Browserify のビルドツールにより実現された .vue 拡張子の 単一ファイルコンポーネント で解決する。以下は、 Hello.vue と呼ばれたファイルの単純な例。

![http_responce](images/vue-component.png)

これに
* 完全版シンタックスハイライト
* CommonJS モジュール
* コンポーネントスコープ CSS
を追加したのが以下。

![http_responce](images/vue-component-with-preprocessors-component.png)

注意すべき重要な点の1つは、関心事項の分離がファイルタイプの分離と等しくないこと。現代の UI 開発では、コードベースを互いに織り交ぜる3つの巨大なレイヤーに分割するのではなく、それらを疎結合なコンポーネントに分割して構成する方がはるかに理にかなっている。コンポーネントの内部では、そのテンプレート、ロジック、スタイルが本質的に結合されているため、実際にそれらを配置するより一貫性と保守性に優れている。

単一ファイルコンポーネントのアイデアが気に入らなくても、JavaScript と CSS を別々のファイルに分けることで、ホットリロードとプリコンパイル機能を活用出来る。

```HTML
<!-- my-component.vue -->
<template>
  <div>This will be pre-compiled</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

#### 始める
チュートリアル参照。

## 単体テスト
#### 単純なテスト
テスト設計の観点から、コンポーネントのテスタビリティを向上させるためにコンポーネント内で特別な何かを行う必要はない。単純に options をエクスポートするだけ。
```HTML
<template>
  <span>{{ message }}</span>
</template>

<script>
  export default {
    data () {
      return {
        message: 'hello!'
      }
    },
    created () {
      this.message = 'bye!'
    }
  }
</script>
```
コンポーネントをテストする際には、 Vue と合わせて options のオブジェクトをインポートし、検証を実施する。以下では例として Jasmine/Jest スタイルの expect アサーションを使用している。
```js
// Vue と テスト対象のコンポーネントをインポートする
import Vue from 'vue'
import MyComponent from 'path/to/MyComponent.vue'

// テストランナーや検証には、どのようなライブラリを用いても構いませんが
// ここでは Jasmine 2.0 を用いたテスト記述を行っています。
describe('MyComponent', () => {
  // コンポーネントの options を直接検証します。
  it('has a created hook', () => {
    expect(typeof MyComponent.created).toBe('function')
  })

  // コンポーネントの options 内にある関数を実行し、
  // 結果を検証します。
  it('sets the correct default data', () => {
    expect(typeof MyComponent.data).toBe('function')
    const defaultData = MyComponent.data()
    expect(defaultData.message).toBe('hello!')
  })

  // コンポーネントインスタンスをマウントして検証します。
  it('correctly sets the message when created', () => {
    const vm = new Vue(MyComponent).$mount()
    expect(vm.message).toBe('bye!')
  })

  // マウントされたコンポーネントインスタンスを描画して検証します。
  it('renders the correct message', () => {
    const Constructor = Vue.extend(MyComponent)
    const vm = new Constructor().$mount()
    expect(vm.$el.textContent).toBe('bye!')
  })
})
```

#### テストしやすいコンポーネントの記述
コンポーネントの描画結果は、コンポーネントの受け取るプロパティによってその大半が決定される。
実際、コンポーネントの描画結果が、単にプロパティの値によってのみ決まる場合、異なる引数を用いた関数の戻り値の検証と同じ様に、シンプルに考えることができる。
```HTML
<template>
  <p>{{ msg }}</p>
</template>

<script>
  export default {
    props: ['msg']
  }
</script>
```
propsDataオプションを利用して、異なるプロパティを用いた描画結果の検証が可能。
```js
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// コンポーネントをマウントし描画結果を返すヘルパー関数
function getRenderedText (Component, propsData) {
  const Constructor = Vue.extend(Component)
  const vm = new Constructor({ propsData: propsData }).$mount()
  return vm.$el.textContent
}

describe('MyComponent', () => {
  it('renders correctly with different props', () => {
    expect(getRenderedText(MyComponent, {
      msg: 'Hello'
    })).toBe('Hello')

    expect(getRenderedText(MyComponent, {
      msg: 'Bye'
    })).toBe('Bye')
  })
})
```
#### 非同期な更新の検証
Vue は 非同期に DOM の更新を行う ため、 state の変更に対する DOM の更新に関する検証は、 Vue.nextTick コールバックを用いて行う必要がある。
```js
// state の更新後、生成された HTML の検証を行う
it('updates the rendered message when vm.message updates', done => {
  const vm = new Vue(MyComponent).$mount()
  vm.message = 'foo'

  // state 変更後、 DOM が更新されるまでの "tick" で待機する
  Vue.nextTick(() => {
    expect(vm.$el.textContent).toBe('foo')
    done()
  })
})
```

## TypeScript のサポート
#### NPM パッケージ内の公式型宣言
静的型システムは、特にアプリケーションが成長するに伴い、多くの潜在的なランタイムエラーを防止するのに役立つ。Vue は TypeScript 向けに公式型宣言を提供しており、Vue コアだけでなく Vue Router と Vuex も同様に提供している。これらは NPM に公開されており、そして最新の TypeScript は、NPM でインストールした時、TypeScript を Vue と共に使うための追加のツールを必要としない。

#### 推奨構成
```js
{
  "compilerOptions": {
    // これは Vue のブラウザサポートに合わせます
    "target": "es5",
    // これは `this` におけるデータプロパティに対して厳密な推論を可能にします
    "strict": true,
    // webpack 2 以降 または rollup を使用している場合、ツリーシェイキングを活用するために
    "module": "es2015",
    "moduleResolution": "node"
  }
}
```
コンポーネントメソッド内で this の型をチェックするには strict: true (もしくは最低でも strict フラグの一部の noImplicitThis: true) を含める必要があることに注意。

#### 開発ツール
* プロジェクト作成
Vue CLI 3 は TypeScript を利用する新規のプロジェクトを生成する事が出来る。以下が生成手順。
```shell
# 1. インストールされていない場合、 Vue CLI をインストールしてください
npm install --global @vue/cli

# 2. 新規のプロジェクトを作成し、続いて "Manually select features" を選択して下さい
vue create my-project-name
```

* 各エディタによるサポート
TypeScript による Vue アプリケーションを開発するために、すぐに利用できる TypeScript のサポートを提供する Visual Studio Code を使用することが推奨される。単一ファイルコンポーネント (SFC) を使用している場合、SFC 内部で TypeScript インターフェイスと他の多くの優れた機能を提供する　Vetur 拡張 を導入すべき。

WebStorm も TypeScript と Vue.js の両方に対してすぐに利用できるサポートを提供している。

#### 基本的な使い方
Vue コンポーネントオプション内部で TypeScript が型を適切に推測できるようにするには、Vue.component または Vue.extend でコンポーネントを定義する必要がある。
```js
import Vue from 'vue'

const Component = Vue.extend({
  // 型推論を有効にする
})

const Component = {
  // これは型推論を持っていません、
  // なぜなら、これは Vue コンポーネントのオプションであるということを伝えることができないためです。
}
```
#### クラススタイル Vue コンポーネント
コンポーネントを宣言するときにクラスベース API を使用する場合は、公式にメンテナンスされている vue-class-component のデコレータを使用できる。
```js
import Vue from 'vue'
import Component from 'vue-class-component'

// @Component デコレータはクラスが Vue コンポーネントであることを示します
@Component({
  // ここではすべてのコンポーネントオプションが許可されています
  template: '<button @click="onClick">Click!</button>'
})
export default class MyComponent extends Vue {
  // 初期データはインスタンスプロパティとして宣言できます
  message: string = 'Hello!'

  // コンポーネントメソッドはインスタンスメソッドとして宣言できます
  onClick (): void {
    window.alert(this.message)
  }
}
```

#### プラグインで使用するための型拡張
プラグインは Vue のグローバル/インスタンスプロパティやコンポーネントオプションを追加することがあり、その場合、TypeScript でそのプラグインを使用したコードをコンパイルするためには型定義が必要になる。幸い、TypeScript にはモジュール拡張 (Module Augmentation) と呼ばれる、すでに存在する型を拡張する機能がある。

例えば、string 型をもつ $myProperty インスタンスプロパティを定義するには、以下のようにする。
```js
// 1. 拡張した型を定義する前に必ず 'vue' をインポートするようにしてください
import Vue from 'vue'

// 2. 拡張したい型が含まれるファイルを指定してください
//    Vue のコンストラクタの型は types/vue.d.ts に入っています
declare module 'vue/types/vue' {
  // 3. 拡張した Vue を定義します
  interface Vue {
    $myProperty: string
  }
}
```
上記のコードを（my-property.d.ts のような）型定義ファイルとしてプロジェクトに含めると、Vue インスタンス上で $myProperty が使用できるようになる。
```js
var vm = new Vue()
console.log(vm.$myProperty) // これはうまくコンパイルされる
```
追加でグローバルプロパティやコンポーネントオプションも定義することもできる。
```js
import Vue from 'vue'

declare module 'vue/types/vue' {
  // `VueConstructor` インターフェイスにおいて
  // グローバルプロパティを定義できます
  interface VueConstructor {
    $myGlobal: string
  }
}

// ComponentOptions は types/options.d.ts に定義されています
declare module 'vue/types/options' {
  interface ComponentOptions<V extends Vue> {
    myOption?: string
  }
}
```
上記の型定義は次のコードをコンパイルできるようにする。
```js
// グローバルプロパティ
console.log(Vue.$myGlobal)

// 追加のコンポーネントオプション
var vm = new Vue({
  myOption: 'Hello'
})
```

#### 戻り値の型にアノテーションをつける
Vue の宣言ファイルは循環的な性質を持つため、TypeScript は特定のメソッドの型を推論するのが困難な場合がある。この理由のため、render や computed のメソッドに戻り値の型のアノテーションを付ける必要がありうる。

```js
import Vue, { VNode } from 'vue'

const Component = Vue.extend({
  data () {
    return {
      msg: 'Hello'
    }
  },
  methods: {
    // 戻り値の型の `this` のために、アノテーションが必要です
    greet (): string {
      return this.msg + ' world'
    }
  },
  computed: {
    // アノテーションが必要です
    greeting(): string {
      return this.greet() + '!'
    }
  },
  // `createElement` は推論されますが、`render` は戻り値の型が必要です
  render (createElement): VNode {
    return createElement('div', this.greeting)
  }
})
```

型推論やメンバの補完が機能していない場合、特定のメソッドにアノテーションを付けるとこれらの問題に対処できる。--noImplicitAny オプションを使用すると、これらのアノテーションが付けられていないメソッドの多くを見つけるのに役立つ。

## プロダクション環境への配信
Vue CLI を使ってればOK! 独自のビルド環境の場合には参照すべき。

## ルーティング
#### 公式ルータ
ほとんどのシングルページアプリケーション (SPA: single page application) を作成する場合、公式にサポートされている vue-router ライブラリ を使うことが推奨される。

#### スクラッチからの単純なルーティング
とてもシンプルなルーティングを必要としていて、フル機能のルータライブラリを使用したくない場合は、このようにページレベルのコンポーネントで動的に描画することが出来る。
```js
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```
HTML5の History API と組み合わせることで、初歩的ではあるが完全に機能するクライアント側のルータを構築することが出来る。

## 状態管理
#### 公式 Flux ライクな実装
大規模なアプリケーションは、多くの状態が色々なコンポーネントに散らばったり、コンポーネント間の相互作用のために複雑になりがち。この問題を解消するために、 Vue は Elm から触発された状態管理ライブラリの vuex を提供する。vue-devtools とも連携し、特別なセットアップなしでタイムトラベルデバッグを提供する。

#### シンプルな状態管理をゼロから作る
Vue アプリケーションの妥当性を担保しているのは生の data オブジェクトであり、Vue インスタンスは単純に data オブジェクトへのアクセスをプロキシする。それゆえに、複数のインスタンスによって共有されうる状態がある場合、シンプルに同一の状態を共有することができる。

```js
const sourceOfTruth = {}

const vmA = new Vue({
  data: sourceOfTruth
})

const vmB = new Vue({
  data: sourceOfTruth
})
```

こうすれば、sourceOfTruth が変化するたびに、vmA と vmB の両方が自動的にそれぞれの view を更新する。これらのインスタンス内のサブコンポーネントから this.$root.$data を通じてアクセスすることもできる。ただ1つの情報源を持つことにはなったが、どんなデータでも、アプリケーションのどこからでも痕跡を残すことなく変えることができてしまってよくない。単純な store パターン を適用することで、この問題を解決出来る。
```js
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    if (this.debug) console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
  clearMessageAction () {
    if (this.debug) console.log('clearMessageAction triggered')
    this.state.message = ''
  }
}
 ```
 Store の状態を変える action はすべて store 自身の中にあり、このように状態管理を中央集権的にすることで、どういった種類の状態変化が起こりうるか、あるいはそれがどうやってトリガされているか、が分かりやすくなる。これなら何か良くないことが起きても、バグが起きるに至ったまでのログを見ることもできる。
さらに、それぞれのインスタンスやコンポーネントに、プライベートな状態を持たせ管理することも可能。

```js
var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

図解すると以下。
![http_responce](images/state.png)

状態変化が監視され続けるためには、コンポーネントと store が同じオブジェクトへの参照を共有している必要があるため、action によって元の状態を決して置き換えてはいけない。

コンポーネントが store が持つ状態を直接変えることは許さず、代わりにコンポーネントは store に通知するイベントを送り出しアクションを実行する、という規約を発展させていくという流れは、最終的に Flux アーキテクチャを作り出すに至った。この規約によって、store に起こるすべての状態変化を記録することができたり、変更ログやスナップショット、履歴や時間を巻き戻す、といった高度なデバッギングヘルパーの実装などの利点がもたらされる。

## サーバサイドレンダリング
#### 完全な SSR ガイド
サーバーでレンダリングされた Vue アプリケーションを作成するためのスタンドアロンのガイドが利用可能。

他、利用できるフレームワークなどの説明。

## リアクティブの探求
Vue の最大の特徴の1つは、控えめなリアクティブシステムである。モデルは単なるプレーンな JavaScript オブジェクトであり、それらを変更するとビューが更新される。これは状態管理を非常にシンプルかつ直感的にするが、よくある問題を避けるためにその仕組みを理解することも重要。

#### 変更の追跡方法
プレーンな JavaScript オブジェクトを data オプションとして Vue インスタンスに渡すとき、Vue はその全てのプロパティを渡り歩いて、それらを Object.defineProperty を使用して getter/setter に変換する。これは ES5 だけの、シム (shim) ができない機能で、Vue が IE8 以下をサポートしないのはこのため。

getter/setter はユーザーには見えないが、内部では、Vue が依存性を追跡したり、プロパティがアクセスまたは変更されたときに変更を通知したりすることを可能にしている。しかし、変換されたデータオブジェクトをログ出力するときには、各ブラウザコンソールで getter/setter のフォーマットが異なるため、より調査に適したインターフェイスの vue-devtools をインストールしたほうが良いかもしれない。

全てのコンポーネントインスタンスは対応する ウォッチャ (watcher) インスタンスを持っていて、これはコンポーネントを描画する間に “触れた (touched)” プロパティを全て依存性として記録している。その後、依存性の setter がトリガされると、ウォッチャに通知してコンポーネントの再描画が起動される。図解すると以下。
![http_responce](images/data.png)

#### 変更検出の注意事項
モダンな JavaScript の限界(そして Object.observe の断念)のため、Vue.js はプロパティの追加または削除を検出できない。Vue はインスタンスの初期化中に getter/setter の変換を行うため、全てのプロパティは Vue が変換してリアクティブにできるように data オブジェクトに存在しなければならない。
```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` は今リアクティブです

vm.b = 2
// `vm.b` はリアクティブでは"ありません"
```
Vue では、すでに作成されたインスタンスに対して新しいルートレベルのリアクティブなプロパティを動的に追加することはできない。しかしながら、Vue.set(object, propertyName, value) メソッドを使ってネストしたオブジェクトにリアクティブなプロパティを追加することができる。
```js
Vue.set(vm.someObject, 'b', 2)
```
vm.$set インスタンスメソッドを使用することもできる。これはグローバルの Vue.set のエイリアスである。
```js
this.$set(this.someObject, 'b', 2)
```
Object.assign() や \_.extend() を使って、既存のオブジェクトに複数のプロパティを割り当てたい場合、オブジェクトに追加された新しいプロパティは変更をトリガしないため、元のオブジェクトとミックスインオブジェクトの両方のプロパティを持つ新たなオブジェクトを作成する必要がある。
```js
// `Object.assign(this.someObject, { a: 1, b: 2 })` の代わり
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

#### リアクティブプロパティの宣言
Vue では新しいルートレベルのリアクティブなプロパティを動的に追加することはできないため、インスタンスの初期化時に前もって全てのルートレベルのリアクティブな data プロパティを宣言する必要がある。空の値でも良い。
```js
var vm = new Vue({
  data: {
    // 空の値として message を宣言する
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// 後で、`message` を追加する
vm.message = 'Hello!'
```

data オプションで message を宣言していないと、Vue は render ファンクションが存在しないプロパティにアクセスしようとしていることを警告する。前もって全てのリアクティブなプロパティを宣言しておくと、後から見直したり別の開発者が読んだりしたときにコンポーネントのコードを簡単に理解することができる。

#### 非同期更新キュー
Vue は 非同期 に DOM 更新を実行する。データ変更が検出されると、Vue はキューをオープンし、同じイベントループで起こる全てのデータ変更をバッファリングする。同じウォッチャが複数回トリガされる場合、キューには一度だけ入る。この重複除外バッファリングは不要な計算や DOM 操作を回避する上で重要であり、次のイベントループ “tick” で、Vue はキューをフラッシュし、実際の(すでに重複が除外された)作業を実行する。内部的には、Vue は非同期キューイング向けにネイティブな Promise.then と MutationObserver、setImmediate が利用可能ならそれを使い、setTimeout(fn, 0) にフォールバックする。

例として、vm.someData = 'new value' をセットした時、そのコンポーネントはすぐには再描画しない。 次の “tick” でキューがフラッシュされる時に更新する。。Vue.js は一般的に”データ駆動”的な流儀で考え、DOM を直接触るのを避けることを開発者に奨励するが、必要に駆られてデータの変更後に Vue.js の DOM 更新の完了を待つには、データが変更された直後に Vue.nextTick(callback) を使用することができ、そのコールバックは DOM が更新された後に呼ばれる。以下は例。
```HTML
<div id="example">{{ message }}</div>
```
```js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message' // データを変更する
vm.$el.textContent === 'new message' // false
Vue.nextTick(function () {
  vm.$el.textContent === 'new message' // true
})
```
vm.$nextTick() というインスタンスメソッドもあり、これはグローバルな Vue を必要とせず、コールバックの this コンテキストが自動的に現在の Vue インスタンスに束縛されるため、コンポーネント内で特に役立つ。
```js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => 'not updated'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => 'updated'
      })
    }
  }
})
```

$nextTick() は Promise を返却するため、新しい ES2016 async/await 構文を用いて、同じことができる。
```js
methods: {
  updateMessage: async function () {
    this.message = 'updated'
    console.log(this.$el.textContent) // => 'not updated'
    await this.$nextTick()
    console.log(this.$el.textContent) // => 'updated'
  }
}
```
