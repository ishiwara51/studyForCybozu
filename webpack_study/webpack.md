# Webpack 勉強メモ
## 環境構築
https://ics.media/entry/12140/ の手順で環境構築。

####
次のコマンドを実行して、プロジェクトの設定情報が記述されたpackage.jsonを生成する。
```bash
npm init -y
```
また、webpackを実行する為に、webpack本体をインストールする。
```bash
npm i -D webpack webpack-cli
```
$npm i -D webpack webpack-cli  をプロジェクトディレクトリごとにやらなきゃいけないのか？ 7.1 MB のファイルが毎回生成されるんですが・・・

#### モジュール方式のJavaScriptを書いてみよう
可読性の観点から、1つのJavaScriptファイルに長い処理を書くのではなく、複数ファイルへ分割することが推奨される。ウェブのフロントエンド界隈では、機能ごとに分割されたJavaScriptファイルのことを一般的に「モジュール」と呼ぶ。

JavaScriptをモジュールで書くにはお作法があり、2019年現在は標準仕様のECMAScript Modules（略してES Modules、もしくはESM）で書くのが一般的。モジュールを取り扱うための仕様で代表的なものとしては、CommonJS、AMD、ES2015のModules等がある。

ES ModulesのJavaScript処理を例に、index.jsでsub.jsに定義されたhello()メソッドを呼び出す仕組みを考えてみる。

```js
// index.js
// import 文を使って sub.js ファイルを読み込む。
import { hello } from "./sub";

// sub.jsに定義されたJavaScriptを実行する。
hello();
```
```js
// sub.js
// export文を使ってhello関数を定義する。
export function hello() {
  alert('helloメソッドが実行された。');
}
```

JavaScriptモジュールはこのままだと古いブラウザ（例：Internet Explorer 11）で使用できないため、古いブラウザが解釈できる形に変換する必要がある。

#### webpackでJavaScriptモジュールを扱う
webpackを使うと、JavaScriptモジュールをブラウザで扱える形に変換できる。index.jsのようにメインとなる処理を行うJavaScriptファイルを「エントリーポイント」と呼び、 このエントリーポイントをビルドすることで、関連するファイル群を統合する。

以下のビルドコマンドを、index.js と sub.js を含む src ディレクトリのある階層で実行する。$npm i -D webpack webpack-cli してない場合、以下のビルド中にやってくれる。html ファイルも入れたい場合は src と同じ階層。
```sh
npx webpack
```

#### package.jsonをカスタマイズする
npx webpackコマンドでビルドするのもいいが、実際の開発ではnpm scriptsを使う方が便利らしい。npm scriptsとはコマンドのショートカット（エイリアス）を貼るための機能。

package.jsonファイルのscriptsに、webpackのビルドコマンドを追加する ($npm i -D webpack webpack-cli すれば自動で追加されるっぽい)。package.jsonファイルには最低限scriptsとdevDependencies指定が記述されてあれば使えるので、nameやlicenseなどは消してしまって大丈夫。

```json
{
  "scripts": {
    "build": "webpack" // 追加するのはここと private ?
  },
  "devDependencies": {
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.1"
  },
  "private": true
}
```

こうしておけば、npm run buildとコマンドラインで入力することで、内部的にwebpackが呼び出され、npx webpack と同じ結果が得られる。何が便利なのかよくわからないけど。

#### webpack.config.jsをカスタマイズする
webpack.config.jsファイルを用意することで、webpackの挙動を調整できる。よく使う設定として、エントリーポイントを指定するentryと、出力フォルダーをカスタマイズするoutputがある。必須ではないが押さえておくべき。以下のように記述しておく。
```js
module.exports = {
  // メインとなるJavaScriptファイル（エントリーポイント）
  entry: `./src/index.js`,

  // ファイルの出力設定
  output: {
    //  出力ファイルのディレクトリ名
    path: `${__dirname}/dist`,
    // 出力ファイル名
    filename: "main.js"
  }
};
```
webpack 4では、エントリーポイントを指定しなければ自動的に「src/index.js」がエントリーポイントに、出力先を指定しなければ自動的に「dist/main.js」が出力先になる。

#### webpackでコードの圧縮とソースマップを有効にする
JavaScriptの開発では、元のソースファイルとの関連性を示すソースマップが、プロジェクト管理的に欠かせない。また、ウェブサイトへの公開時にはウェブページの読み込みを早くするために、ファイル容量を圧縮することも重要。webpackでは設定ファイルの記述によって、それらをカスタマイズできる。

webpackの設定ファイルには以下のように記述する。modeにdevelopmentを記述することでソースマップを有効にする。逆に、modeの部分でproductionを指定することで、JavaSciptのコードを圧縮できる。開発時にはdevelopmentを指定し、ウェブサイト公開時にはproductionに設定するのが良いだろう。

```js
module.exports = {
  // モード値を production に設定すると最適化された状態で、
  // development に設定するとソースマップ有効でJSファイルが出力される
  mode: "development"
};
```

webpack.config.jsファイルにdevelopmentを指定を指定した場合は、npm run buildコマンドを入力すると、srcフォルダーに配置したJSファイルがコンパイルされ、distフォルダーにmain.jsファイルが出力される。

#### webpackでローカルサーバーを起動し、変更時にブラウザをリロードする
毎回ビルドコマンドをコマンドラインで打ち込むのは効率的でない。ファイルの変更を検知し（watchともいう）、自動的にビルドコマンドを実行し、ブラウザをリロードする・・・といった手順を、 webpack-dev-server により自動化できる。類似の技術として「lite-server」や「BrowserSync」といったものがあり、それと類似のものと考えて良い。

またこれにより、二度目以降のビルドは、差分ビルドとして時間が大幅に短縮される。ビルド短縮のため、webpack-dev-serverもしくは後述のwatch機能は必ず利用すべき。

#### npmモジュールのインストール
webpack関連モジュールとwebpack-dev-serverモジュールをインストールする。
```bash
npm i -D webpack webpack-cli webpack-dev-server
```
これをインストールすると、package.jsonファイルは以下の内容になる。scripts に自前のビルドコマンドとして "start": "webpack-dev-server" を記述しておくのがポイント。らしいが、package.json はこのコマンドでは更新されなかった。何が悪いのか。

```json
{
  "scripts": {
    "build": "webpack",
    "start": "webpack-dev-server"
  },
  "devDependencies": {
    "webpack": "^4.5.0",
    "webpack-cli": "^2.0.14",
    "webpack-dev-server": "^3.1.1"
  }
}
```

#### webpackの設定ファイル
webpack.config.js は次のように記述する。devServerにルートフォルダーを設定する。open: trueを指定しておくと、自動的にブラウザが立ち上がる。

```js
module.exports = {
  // モード値を production に設定すると最適化された状態で、
  // development に設定するとソースマップ有効でJSファイルが出力される
  mode: "development",

  // ローカル開発用環境を立ち上げる
  // 実行時にブラウザが自動的に localhost を開く
  devServer: {
    contentBase: "dist",
    open: true
  }
};
```

npm run start コマンド、もしくは　npx webpack-dev-server コマンドで起動する。自動的にブラウザが起動しローカルホストで表示され、ファイル保存時にブラウザが自動的にリロードするので、コーディング作業が楽になる。

なお、webpack-dev-serverに似た、webpack-serveというツールがあるが非推奨。

#### ファイル変更時に差分ビルドを。ウォッチを利用する
webpack-dev-serverはとても便利ではあるが、ブラウザで確認する必要がないときは機能が多すぎて余分になるときもある。JavaScriptをビルドしたいだけであれば、watch機能を利用するのが良い。watch機能を利用するにはコマンドラインの引数に「–watch」を追加するだけ。もしくは、package.jsonファイルのscriptsに自前のビルドコマンドとして "start": "webpack --watch" を記述しておけば、npm run watch でいける。

```bash
npx webpack --watch
or
npm run watch
```

何してくれてるのかいまいちわからないので要調べ。

#### タスクランナーとの使い分け
webpackはこうした性質上、タスクランナーであるGulpやGruntの代わりとして紹介されることがしばしばあり、実際にタスクランナーでできることの多くはwebpackでも可能。しかし、プロジェクトによってはタスクランナーGulp、Gruntの資産があり、webpackを部分的に採用したいケースもある。また、webpackは「CSSや画像を含むあらゆるアセットファイルをJavaScriptとして出力する」ことが基本的な使い方となっているため、CSSや画像をそのまま扱いたい時はタスクランナーが必要になる。

つまりは、webpackとタスクランナーは併用して使うことも選択肢の1つである。

#### 他のモジュールバンドラーとの性能比較
モジュールバンドラーとして知られているのはwebpackだけではなく、以下のようなものが代表的。

!(image)[images/170704_webpack_comparison_tools.png]

webpack4 は、これらのバンドラーの中でビルド時間及び容量の面で優れる。トレンドもwebpack。

# 公式チュートリアル
## はじめに
#### セットアップ
npm init -y して、 npm i -D webpack webpack-cli する。
index.html と　src/index.js 作る。内容は以下。

```html
<!doctype html>
<html>
  <head>
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```
```js
function component() {
  const element = document.createElement('div');

  // Lodash, currently included via a script, is required for this line to work
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

また、事故的に公開してしまうことを防ぐために、package.json をいじって、private を追加して index.js のエントリーポイントを消す。

```json
 {
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
+   "private": true,
-   "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.20.2",
      "webpack-cli": "^3.1.2"
    },
    "dependencies": {}
  }
```

#### Bundle を形成する
形成する前に、我々がいじるソースコード (src) と、minimized and optimized されたディストリビューションコード (dist) を分ける。今回は、index.html を dist に移動させる。

今回作成する index.js は lodash ライブラリに依存する予定のため、以下のコードでローカルにライブラリを落とす。このように production bundle に統合すべきパッケージを落とす時には npm install --save を使い、linter, testing libraries などの開発用パッケージの時は npm install --save-dev を使う。

```bash
npm install --save lodash
```

落としたら、index.js 内で import する。こうすることで、依存関係を明示でき、webpack がバンドルできるようにできる。_ としているのは、no global scope pollution　にするためらしい。よくわからん。

```js
+ import _ from 'lodash';
+
  function component() {
    const element = document.createElement('div');

-   // Lodash, currently included via a script, is required for this line to work
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');

    return element;
  }

  document.body.appendChild(component());
```

これで、元のコードでは <script> によって有効化していたパッケージを、<script> なしで使えるようになった。のでindex.html の <script~lodash~ の部分を消して、出力される bundle ファイルを使用するために、body の index.js を main.js にする。

```html
 <!doctype html>
  <html>
   <head>
     <title>Getting Started</title>
-    <script src="https://unpkg.com/lodash@4.16.6"></script>
   </head>
   <body>
-    <script src="./src/index.js"></script>
+    <script src="main.js"></script>
   </body>
  </html>
```

これで npx webpack できる。

#### Modules
import と export は ES2015により実現されるが、古いブラウザでは動かない。そこで、Webpack は import と export を含む様々なモジュールを transpile して、古いブラウザでも動くように意してくれる。

import と export 以外は、webpack はこの先 alter しないので、 webpack loader system から babel などを使用することが推奨される。

#### webpack.config.js
これ以外の名前で bundle 時の config に使いたい時は、npx webpack --config ~.js とする必要がある。

## Asset Management
Webpack は、エントリーポイントの コードを起点として dependency graph を想定し、src のファイルを dist へ動的にバンドルする。ので、例えば使われていないモジュールを一緒にバンドルするなどのリスクをなくせる。以下、js 以外のファイル形式をバンドルする方法を試す。

#### Setup
index.html と webpack.config.jsを少しいじっておく。リファクタ的。
```html
  <!doctype html>
  <html>
    <head>
-    <title>Getting Started</title>
+    <title>Asset Management</title>
    </head>
    <body>
-     <script src="main.js"></script>
+     <script src="bundle.js"></script>
    </body>
  </html>
```
```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
-     filename: 'main.js',
+     filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

#### Loading CSS
CSS を js 内で import するには、まず module configlation に style-loader と css-loader をインストールする。これは開発用のインストールであるから、 --save ではなく -D を使う。

```bash
npm i --D style-loader css-loader
```
そして、webpack.config.js を以下のようにする。
```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
+   module: {
+     rules: [
+       {
+         test: /\.css$/,
+         use: [
+           'style-loader',
+           'css-loader'
+         ]
+       }
+     ]
+   }
  };
```
これで、css を js 上で import できるようになった。この二つの loader は、html の <head> に <style> を挿入してくれる。

style.css を src に作成して、ビルドしてみよう。

```css
.hello {
  color: red;
}
```
```js
  import _ from 'lodash';
+ import './style.css';

  function component() {
    const element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.classList.add('hello');

    return element;
  }

  document.body.appendChild(component());
```
動かなかった。loader が適切じゃないらしい。

#### Loading Image
ローダーをインストールする。
```bash
npm install --save-dev file-loader
```
他のファイルもチュートリアルに従って変更する。
css が読み込めないって言われる。