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
css が読み込めないって言われる。以下動かない。

#### Loading Fonts
webpack.config.js の module/rules に以下を追加する。
file-loader はどんな種類のファイルも扱えるので、これを再利用する。
```js
{
  test: /\.(woff|woff2|eot|ttf|otf)$/,
  use: [
    'file-loader'
  ]
}
```
src にフォントファイル my-font.woff, my-font.woff2 を追加する。
style.css を以下のようにする。

```CSS
+ @font-face {
+   font-family: 'MyFont';
+   src:  url('./my-font.woff2') format('woff2'),
+         url('./my-font.woff') format('woff');
+   font-weight: 600;
+   font-style: normal;
+ }

  .hello {
    color: red;
+   font-family: 'MyFont';
    background: url('./icon.png');
  }
```
これで実行可能。

#### Loading Data
JSONやCSV,TSVやXMLを扱う。
JSONだけはデフォルトで　import Data from './data.json' ができるが、他はそうはいかないので、以下でローダーを追加する。
```bash
npm install --save-dev csv-loader xml-loader
```
で、webpack.config.js の module/rules に以下を追加する。

```js
{
  test: /\.(csv|tsv)$/,
  use: [
    'csv-loader'
  ]
},
{
  test: /\.xml$/,
  use: [
    'csv-loader'
  ]
}
```

試しに、src に以下の xml を追加してみる。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Mary</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Call Cindy on Tuesday</body>
</note>
```

で、 index.js で　import Data from './data.xml'; して、 component() 内で console.log(Data); すれば、データの中身が見れるはず。

#### Global Assets
コードがちゃんと動かないとわからない。

## Output Management
まず、前章で追加したコードとファイルを全部消す。

次に src/print.js を作成し、中身を以下のようにする。

```js
export default function printMe() {
  console.log('I get called from print.js!');
}
```

で、この printMe() を index.js 内で使う。
```js
import _ from 'lodash';
+ import printMe from './print.js';

function component() {
  const element = document.createElement('div');
+   const btn = document.createElement('button');

  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

+   btn.innerHTML = 'Click me and check the console!';
+   btn.onclick = printMe;
+
+   element.appendChild(btn);

  return element;
}

document.body.appendChild(component());
```

併せてhtmlもいじっておく。

```HTML
<!doctype html>
<html>
  <head>
-     <title>Asset Management</title>
+     <title>Output Management</title>
+     <script src="./print.bundle.js"></script>
  </head>
  <body>
-     <script src="./bundle.js"></script>
+     <script src="./app.bundle.js"></script>
  </body>
</html>
```

最後に webpack.config.js をいじる。エントリーポイントに print.js を追加し、二つのエントリーポイントそれぞれについてバンドルファイルを生成するよう書き換える。

```js
const path = require('path');

module.exports = {
-   entry: './src/index.js',
+   entry: {
+     app: './src/index.js',
+     print: './src/print.js'
+   },
  output: {
-     filename: 'bundle.js',
+     filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

これで npm run build すれば、app.bundle.js と print.bundle.js の二つが出力されるはず。だけど動きませんでした。以降、勉強だけ。

エントリーポイントの名前を変えたり、エントリーポイントを追加したりした場合、自動で output の名前も変更してくれる。しかし、html 内の参照は変更されないのでバグる。これを HtmlWebpackPlugin で解決する。

#### Setting up HtmlWebpackPlugin
npm install --save-dev html-webpack-plugin して、webpack.config.js を以下のようにする。

```js
const path = require('path');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
+   plugins: [
+     new HtmlWebpackPlugin({
+       title: 'Output Management'
+     })
+   ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

これで npm run built すると、新しい、同時に生成されたバンドルファイルの情報を反映した index.html が生成される。すでに dist/index.html を自前で作っている場合、新しく生成されるファイルに上書きされる。

#### Cleaning up the /dist folder
npm run built はバンドルファイルを出力するまでしか管理せず、その後のバンドルファイルのうち、実際にどれが使われるかまでは管理してくれない。ので run ごとの前に、dist の中を消しておくと良い。 clean-webpack-plugin でお手軽になる。npm install --save-dev clean-webpack-plugin してから、webpack.config.js を以下のようにする。
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
+ const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
  plugins: [
+     new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Output Management'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

これで、 run のたびに dist 内の古いファイルを消してくれるようになる。

#### The Manifest
output 先を管理する別の方法として、manifest がある。これは、webpack が個々のモジュールがバンドルのなかでどう配置されるのかを管理するためのもので、WebpackManifestPlugin を使用することで、json ファイルとして出力できる。

## development
ここでは、mode を development にして進める。各モジュールは development モード下でのみ動く。

#### Using source map
最終的にバンドルファイルを動かしてエラーを吐いた場合、エラーメッセージはただそのバンドルファイルについてのみ説明し、バンドル元のファイルのうちどれが原因なのかはわからない。これを追跡するには source map を見る必要があり、いろいろな方法があるが、ここでは inline-source-map オプションを、webpack.config.js を以下のようにして使う。

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
+   devtool: 'inline-source-map',
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Development'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

実験するために、print.js を以下のように変えてエラーを持たせる。

```js
export default function printMe() {
-   console.log('I get called from print.js!');
+   cosnole.log('I get called from print.js!');
}
```

これで npm run build すると、出力ログは以下のようになる。

```bash
...
          Asset       Size  Chunks                    Chunk Names
  app.bundle.js    1.44 MB    0, 1  [emitted]  [big]  app
print.bundle.js    6.43 kB       1  [emitted]         print
     index.html  248 bytes          [emitted]
...
```

print.js は index.js で生成されたボタンにより呼び出されるものであった。生成された index.html を開いて、ボタンを押してみると、以下のようなエラーログが見られるはず。

```bash
Uncaught ReferenceError: cosnole is not defined
   at HTMLButtonElement.printMe (print.js:2)
```

よかったよかった。

#### Choosing a Development Tool
コンパイルしたい時、毎回 npm run build を走らせるのは面倒。以下のような自動コンパイルの手段がある。
1. webpack's Watch Mode
2. webpack-dev-server
3. webpack-dev-middleware

だいたいの場合 webpack-dev-server を使う。

#### Watch mode
package.json/scripts で "watch": "webpack --watch" を追加して、npm run watch すれば、コンソール上で回り続け、src下に変更があった場合にコンパイルを再度走らせてくれる。

ただ、watch の場合ブラウザは再読み込みする必要があり、その点では webpack-dev-server の方が良い。

#### using webpack-dev-server
webpack と違い、こちらは新たに npm install --save-dev webpack-dev-server する必要がある。
さらに、webpack.config.js に以下を追加する。
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
  devtool: 'inline-source-map',
+   devServer: {
+     contentBase: './dist'
+   },
  plugins: [
    // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Development'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

で、サーバーを走らせるコマンドを以下のように追加する。

```json
{
  "name": "development",
  "version": "1.0.0",
  "description": "",
  "main": "webpack.config.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch": "webpack --watch",
+     "start": "webpack-dev-server --open",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "clean-webpack-plugin": "^2.0.0",
    "css-loader": "^0.28.4",
    "csv-loader": "^2.1.1",
    "file-loader": "^0.11.2",
    "html-webpack-plugin": "^2.29.0",
    "style-loader": "^0.18.2",
    "webpack": "^4.30.0",
    "xml-loader": "^1.2.1"
  }
}
```

#### webpack-dev-middleware
内部的には webpack-dev-server 内で、webpack により処理されたファイル群をサーバーに送るためのラッパー。独立したパッケージとして使用することもでき、カスタム性の高い使用が可能。以下の dev インストールが必要。

```bash
npm install --save-dev express webpack-dev-middleware
```

で、webpack.config.js を以下のようにいじる。
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
  devtool: 'inline-source-map',
  devServer: {
    contentBase: './dist'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Output Management'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
+     publicPath: '/'
  }
};
```

追加した publicPath は、サーバー上のスクリプトが、http://localhost:3000 上にファイルが正しく送られているかを確認するためのもの。次に、express サーバーを設定する。

dist や src と同じ階層に server.js を以下のように作成する。

```js
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');

const app = express();
const config = require('./webpack.config.js');
const compiler = webpack(config);

// Tell express to use the webpack-dev-middleware and use the webpack.config.js
// configuration file as a base.
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath
}));

// Serve the files on port 3000.
app.listen(3000, function () {
  console.log('Example app listening on port 3000!\n');
});
```

で、package.json/scripts に "server": "node server.js", を追加して、npm run server してみると、以下のようなログが見られ、http://localhost:3000 をひらけば走っていることがわかる。

```bash
Example app listening on port 3000!
...
          Asset       Size  Chunks                    Chunk Names
  app.bundle.js    1.44 MB    0, 1  [emitted]  [big]  app
print.bundle.js    6.57 kB       1  [emitted]         print
     index.html  306 bytes          [emitted]
...
webpack: Compiled successfully.
```

#### Adjusting Your Text Editor
エディタによっては、上記の自動再コンパイルと競合する機能があるらしい。

## Code splitting
ソースコード群を複数のバンドルファイルに分けて出力できる機能。3つの手段がある。
1. Entri Points: webpack.config.js の entry をいじる。
2. Prevent Duplication: SplitChunksPlugin を使って、重複排除とチャンクわけを行う。
3. Dynamic Imports: モジュール内で使用されるインライン関数によってコードを分割する。

#### Entry Points
最も単純な方法だが、以下のような落とし穴もある。
* 複数のエントリーポイントで使用されるモジュールがあった場合、そのモジュールは両方のバンドルに内包されてしまう。
* core application logic により動的にコードを分割することができない。

先の index.js と pring.js の場合、どっちも lodash を使っているので、複製が起こってしまう。次の SplitChunksPlugin を使えばこれを解消できる。

#### Prevent Duplication
SplitChunksPlugin は、全てのチャンクに共通する依存関係をまとめたり、新しいチャンクを作ってくれたりする。webpack v4 legato までは CommonsChunkPlugin だった。

webpack.config.js を以下のようにする。

```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: {
    index: './src/index.js',
    another: './src/another-module.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
+   optimization: {
+     splitChunks: {
+       chunks: 'all'
+     }
+   }
};
With the optim
```

optimization.splitChunks により、以下のようにログが出される。print が another になっているが変わりはない。

```bash
...
                          Asset      Size                 Chunks             Chunk Names
              another.bundle.js  5.95 KiB                another  [emitted]  another
                index.bundle.js  5.89 KiB                  index  [emitted]  index
vendors~another~index.bundle.js   547 KiB  vendors~another~index  [emitted]  vendors~another~index
Entrypoint index = vendors~another~index.bundle.js index.bundle.js
Entrypoint another = vendors~another~index.bundle.js another.bundle.js
...
```

vendors~another~index.bundle.js には分離された lodash が入っているらしい。
コミュニティから、コード分割のための便利なプラグインやローダーが出ている。
* mini-css-extract-plugin: Useful for splitting CSS out from the main application.
* bundle-loader: Used to split code and lazy load the resulting bundles.
* promise-loader: Similar to the bundle-loader but uses promises.

#### Dynamic Imports
動的なコード分割については、二つの類似した手段がある。一つ目は、import() を使って ECMAScript Proposal を適用する方法で、こちらが推奨されている。もう一つはより古い手段で、 require.ensure を使う方法。一つ目を見てみる。

import() は内部的には promises を呼んでおり、古いブラウザで使用する際には、es6-promise や promise-polyfill により補填する必要がある。

まず、webpack.config.js を以下のようにする。
```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: {
+     index: './src/index.js'
-     index: './src/index.js',
-     another: './src/another-module.js'
  },
  output: {
    filename: '[name].bundle.js',
+     chunkFilename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
-   optimization: {
-     splitChunks: {
-       chunks: 'all'
-     }
-   }
};
```

chunkFilename は、エントリーポイント以外のチャンクの名前を指定するものらしい。

さらに、src 内を index.js だけにして、lodash を動的にインポートしてみる。
index.js を以下のようにいじる。

```
- import _ from 'lodash';
-
- function component() {
+ function getComponent() {
-   const element = document.createElement('div');
-
-   // Lodash, now imported by this script
-   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ }) => {
+     const element = document.createElement('div');
+
+     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+     return element;
+
+   }).catch(error => 'An error occurred while loading the component');
  }

- document.body.appendChild(component());
+ getComponent().then(component => {
+   document.body.appendChild(component);
+ })
```

これで　npm run build すると、以下のようなログが出る。

```bash
 ...
                   Asset      Size          Chunks             Chunk Names
         index.bundle.js  7.88 KiB           index  [emitted]  index
vendors~lodash.bundle.js   547 KiB  vendors~lodash  [emitted]  vendors~lodash
Entrypoint index = index.bundle.js
...
```

import() は promise を返すので、async 関数と一緒に使える。ただし、Babel や Syntax Dynamic Import Babel Plugin などの前処理が必要。よくわからん。とにかく、async 関数を使えば index.js を以下のように簡略化できる。

```
- function getComponent() {
+ async function getComponent() {
-   return import(/* webpackChunkName: "lodash" */ 'lodash').then({ default: _ } => {
-     const element = document.createElement('div');
-
-     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
-
-     return element;
-
-   }).catch(error => 'An error occurred while loading the component');
+   const element = document.createElement('div');
+   const { default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');
+
+   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+   return element;
  }

  getComponent().then(component => {
    document.body.appendChild(component);
  });
```

#### Prefetching/Preloading modules
webpack 4.6.0 以降なら使える。インポートを宣言するときにこれらの inline directives を使うと、ブラウザに以下を伝える Resource Hint を出力できるようになる。
* prefetch: resource is probably needed for some navigation in the future
* preload: resource might be needed during the current navigation

よくわからんので要調べ。

## Caching
dist がサーバーにデプロイされると、ブラウザなどのクライアントはサイトにアクセスするためにサーバーに働きかける。このサーバーとのやり取りは往往にしてボトルネックになるため、ブラウザはキャッシュを使用する。webpack がコンパイルするファイルにキャッシュを適用するには、設定が必要になる。

#### Output Filenames
webpack は、substitutions と呼ばれる、文字列を [] で括ることでファイル名テンプレートを作る方法を提供している。[contenthash] substitution は、asset のコンテンツの状態に固有のハッシュ値であり、状態が変化すれば変わる。webpack.config.js をいじってこれを使えるようにする。

```js
 const path = require('path');
 const { CleanWebpackPlugin } = require('clean-webpack-plugin');
 const HtmlWebpackPlugin = require('html-webpack-plugin');

 module.exports = {
   entry: './src/index.js',
   plugins: [
     // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
     new CleanWebpackPlugin(),
     new HtmlWebpackPlugin({
-       title: 'Output Management'
+       title: 'Caching'
     })
   ],
   output: {
-     filename: 'bundle.js',
+     filename: '[name].[contenthash].js',
     path: path.resolve(__dirname, 'dist')
   }
 };
 ```

実行すると以下のようなログが得られる。

```bash
...
                       Asset       Size  Chunks                    Chunk Names
main.7e2c49a622975ebd9b7e.js     544 kB       0  [emitted]  [big]  main
                  index.html  197 bytes          [emitted]
...
```

ハッシュ値がファイル名に反映されていることがわかる。

#### Extracting Boilerplate
optimization.runtimeChunk オプションを使うことで、ランタイムコードを個別のチャンクに分けることができる。以下のように、これに single を指定すれば、それぞれのチャンクに一つずつランタイムコードを割り当てられる。

 ```js
 const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  plugins: [
    // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Caching'
    })
  ],
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist')
  },
+   optimization: {
+     runtimeChunk: 'single'
+   }
};
```

実行した時のログは以下。

```bash
Hash: 82c9c385607b2150fab2
Version: webpack 4.12.0
Time: 3027ms
                          Asset       Size  Chunks             Chunk Names
runtime.cc17ae2a94ec771e9221.js   1.42 KiB       0  [emitted]  runtime
   main.e81de2cf758ada72f306.js   69.5 KiB       1  [emitted]  main
                     index.html  275 bytes          [emitted]
[1] (webpack)/buildin/module.js 497 bytes {1} [built]
[2] (webpack)/buildin/global.js 489 bytes {1} [built]
[3] ./src/index.js 309 bytes {1} [built]
    + 1 hidden module
```

また、lodash などの変更されにくいものは別のチャンクに分けておいた方が良い。SplitChunksPlugin の cacheGroups オプションで実現できる。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  plugins: [
    // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Caching'
    }),
  ],
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
-     runtimeChunk: 'single'
+     runtimeChunk: 'single',
+     splitChunks: {
+       cacheGroups: {
+         vendor: {
+           test: /[\\/]node_modules[\\/]/,
+           name: 'vendors',
+           chunks: 'all'
+         }
+       }
+     }
  }
};
```

ログは以下。

```bash
...
                          Asset       Size  Chunks             Chunk Names
runtime.cc17ae2a94ec771e9221.js   1.42 KiB       0  [emitted]  runtime
vendors.a42c3ca0d742766d7a28.js   69.4 KiB       1  [emitted]  vendors
   main.abf44fedb7d11d4312d7.js  240 bytes       2  [emitted]  main
                     index.html  353 bytes          [emitted]
...
```

node_modules から vendor コードを引っ張って来る必要がなくなったため、main.~.js のサイズが 240 bytes まで削減できている。

#### Module Identifiers
再び print.js を作る。

```js
+ export default function print(text) {
+   console.log(text);
+ };
```

index.jsをいじる。

```js
import _ from 'lodash';
+ import Print from './print';

function component() {
  const element = document.createElement('div');

  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.onclick = Print.bind(null, 'Hello webpack!');

  return element;
}

document.body.appendChild(component());
```

変更は main.~.js バンドルだけに起こっているので、このハッシュ値だけが変わるべきだが、実際に実行すると runtime.~.js と vendor.~.js のハッシュ値も変わる。これはそれぞれの module.id が resolving をもとに増加していくため。どういうこと？つまり以下。
* The main bundle changed because of its new content.
* The vendor bundle changed because its module.id was changed.
* And, the runtime bundle changed because it now contains a reference to a new module.

main と runtimeは理解できるので、vendor だけ直したい。optimization.moduleIds の hashed オプションで直そう。

```js
const path = require('path');
 const { CleanWebpackPlugin } = require('clean-webpack-plugin');
 const HtmlWebpackPlugin = require('html-webpack-plugin');

 module.exports = {
   entry: './src/index.js',
   plugins: [
     // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
     new CleanWebpackPlugin(),
     new HtmlWebpackPlugin({
       title: 'Caching'
     })
   ],
   output: {
     filename: '[name].[contenthash].js',
     path: path.resolve(__dirname, 'dist')
   },
   optimization: {
+     moduleIds: 'hashed',
     runtimeChunk: 'single',
     splitChunks: {
       cacheGroups: {
         vendor: {
           test: /[\\/]node_modules[\\/]/,
           name: 'vendors',
           chunks: 'all'
         }
       }
     }
   }
 };
```
これで、新しいローカル依存関係が追加されても、vendor ハッシュ値は常に固定になった。
試しに、index.js をいじって print.js との依存を切ってみても、vendor のハッシュ値は変わらない。

## Authoring libraries
#### Authorizing a Library
webpack-numbers という、numeric と string を相互変換するライブラリを作るとする。
この場合、src内に index.js と ref.json を配置し、npm init -y と npm i -D webpack lodash をする。

```js
import _ from 'lodash';
import numRef from './ref.json';

export function numToWord(num) {
  return _.reduce(numRef, (accum, ref) => {
    return ref.num === num ? ref.word : accum;
  }, '');
}

export function wordToNum(word) {
  return _.reduce(numRef, (accum, ref) => {
    return ref.word === word && word.toLowerCase() ? ref.num : accum;
  }, -1);
}
```
```JSON
[
  {
    "num": 1,
    "word": "One"
  },
  {
    "num": 2,
    "word": "Two"
  },
  {
    "num": 3,
    "word": "Three"
  },
  {
    "num": 4,
    "word": "Four"
  },
  {
    "num": 5,
    "word": "Five"
  },
  {
    "num": 0,
    "word": "Zero"
  }
]
```

モジュールの規約ごとに以下のようにimportする。

* ES2015 module import
```js
import * as webpackNumbers from 'webpack-numbers';
// ...
webpackNumbers.wordToNum('Two');
```
* CommonJS module require
```js
const webpackNumbers = require('webpack-numbers');
// ...
webpackNumbers.wordToNum('Two');
```
* AMD module require
```js
require(['webpackNumbers'], function (webpackNumbers) {
  // ...
  webpackNumbers.wordToNum('Two');
});
```

また、consumer でも script tag によりライブラリを使うことができる。
```HTML
<!doctype html>
<html>
  ...
  <script src="https://unpkg.com/webpack-numbers"></script>
  <script>
    // ...
    // Global variable
    webpackNumbers.wordToNum('Five')
    // Property in the window object
    window.webpackNumbers.wordToNum('Five')
    // ...
  </script>
</html>
```

他にも以下のようなライブラリの使用法がある。
* node.js のみ。グローバルオブジェクトのプロパティとして。
* this のプロパティとして。

#### Base configuration
それでは、このライブラリーを以下の条件でバンドルする。
* externals を使い、 lodash を省いてバンドルし、consumer に lodash のロードを求める。
* ライブラリ名を webpack-numbers とする。
* webpackNumbers という変数名でライブラリを登録する。
* Node.js の内部でライブラリにアクセスできるようにする。

また、consumer はライブラリーに以下の規約でアクセスできる必要がある。
* ES2015 module. i.e. import webpackNumbers from 'webpack-numbers'.
* CommonJS module. i.e. require('webpack-numbers').
* Global variable when included through script tag.

webpack.config.js を以下のようにする。
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'webpack-numbers.js'
  }
};
```

#### Externalize Lodash
この状態で run すると、やや大きい bundle がつくれられ、中に lodash が入っていることがわかる。この場合、lodash をpeerDependencyとして、つまり consumer が lodash をインストールしていることを要求するものとして扱う。これは external コンフィグによって実現される。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'webpack-numbers.js'
-   }
+   },
+   externals: {
+     lodash: {
+       commonjs: 'lodash',
+       commonjs2: 'lodash',
+       amd: 'lodash',
+       root: '_'
+     }
+   }
};
```

external とは、このライブラリの外部、つまりは consumer の環境に lodash が利用可能であることを要求するということ。

#### External Limitations
dependency の中からいくつかのファイルを利用するライブラリの場合、以下のようにする。。
```js
import A from 'library/one';
import B from 'library/two';

// ...
```

どちらも library に入っているからと言って、external に library を指定するだけで下層を省略することはできず、一つずつ指定するか、もしくは正規表現を使って行を省略する。

```js
module.exports = {
  //...
  externals: [
    'library/one',
    'library/two',
    // Everything that starts with "library/"
    /^library\/.+$/
  ]
};
```

#### Expose the Library
ライブラリは幅広い環境に対応していた方が良い。consumpution でライブラリを利用可能にするには、output のなかに library プロパティを作成する。
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
-     filename: 'webpack-numbers.js'
+     filename: 'webpack-numbers.js',
+     library: 'webpackNumbers'
  },
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_'
    }
  }
};
```

ここで、多くのライブラリは一つのエントリポイントをもつ。配列をエントリポイントとしてもつライブラリは非推奨。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'webpack-numbers.js',
-     library: 'webpackNumbers'
+     library: 'webpackNumbers',
+     libraryTarget: 'umd'
  },
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_'
    }
  }
};
```

この webpack.config.js の場合、インポートされたライブラリはグローバル変数として扱われる。使用できる環境を追加したい場合、libraryTarget プロパティを追加する。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'webpack-numbers.js',
-     library: 'webpackNumbers'
+     library: 'webpackNumbers',
+     libraryTarget: 'umd'
  },
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_'
    }
  }
};
```

他にも、以下のような方法でライブラリを expose できる。
* Variable: as a global variable made available by a script tag (libraryTarget:'var').
* This: available through the this object (libraryTarget:'this').
* Window: available trough the window object, in the browser (libraryTarget:'window').
* UMD: available after AMD or CommonJS require (libraryTarget:'umd').

libraryTarget を設定せず、 library だけを設定した場合、libraryTarget は var になる。

#### Final Steps
生成されたバンドルのパスを package.json に追加しておく。

```json
{
  ...
  "main": "dist/webpack-numbers.js",
  ...
}
```

standard module として登録する場合は以下。
```json
{
  ...
  "module": "src/index.js",
  ...
}
```
The key main refers to the standard from package.json, and module to a proposal to allow the JavaScript ecosystem upgrade to use ES2015 modules without breaking backwards compatibility.

これで、ライブラリを unpkg.com で npm package として配布し、検索できるようになった。

## Environment Variables
environment 変数は、webpack.config.js に対して、コマンドライン上から --env を用いて追加することができる。例えば以下。

```bash
webpack --env.NODE_ENV=local --env.production --progress
```

webpack.config.js にはさらに、module.exports ポイントを追加して、関数化する必要がある。

```js
const path = require('path');

module.exports = env => {
  // Use env.<YOUR VARIABLE> here:
  console.log('NODE_ENV: ', env.NODE_ENV); // 'local'
  console.log('Production: ', env.production); // true

  return {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
};
```

## Build Performance
#### Loaders
```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader'
      }
    ]
  }
};
```
これの代わりに、以下のように include を使って必要最低限のローダーモジュールを適用する。
```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'babel-loader'
      }
    ]
  }
};
```

#### Bootstrap
loader や plugin は、それぞれがブートアップタイムを必要とする。最低限にしよう。

#### resolving
展開の時間を節約するため、以下に気をつける。
* ファイルシステムのコールを削減するため、resolve.modules, resolve.extensions, resolve.mainFiles, resolve.descriptionFiles 内のアイテムを最小限にする。
* npm link や yarn link などの　symlink を使わない場合、resolve.symlinks: false を設定する。
* contest specific 出ない plugin を使う場合、resolve.cacheWithContext: false　を設定する。

#### Dlls
あまり変更しないコードは、異なるコンパイルチャンクに分離する。ただし、ビルドの複雑性は増加するので注意。

#### Smaller = Faster
以下を守り、チャンクをできるだけ小さくする。
* Use fewer/smaller libraries.
* Use the SplitChunksPlugin in Multi-Page Applications.
* Use the SplitChunksPlugin in async mode in Multi-Page Applications.
* Remove unused code.
* Only compile the part of the code you are currently developing on.

#### Worker Pool
高負荷のローダーを worker pool に移動させるために、thread-loader を使うといい。ただし、worker とメインの処理の間の移動も最小限にした方が良い。

#### Persistent cacheGroups
cache-loader によって persistent cache を使用可能にし、キャッシュディレクトリを postinstall か package.json でクリアできる。

## development
以下、Environment = development 下で便利。

#### Incremental Builds
プロジェクトのリアクティブには watch だけを使おう。watch は タイムスタンプを記録し、この情報をキャッシュの無効化のために提供してくれる。

watch の処理は、polling mode に返る時がある。これにより発生する多くの watch により、CPU負荷が増大する。このような場合には、watchOptions.poll で polling interval を延長する。

#### Compile in Memory
以下のユーティリティを使うことで、ディスクではなくメモリ上でコンパイルやアセットの配布ができるので、パフォーマンスが改善する。
* webpack-dev-server
* webpack-hot-middleware
* webpack-dev-middleware

#### stats.toJson speed
webpack 4 では、大容量データを出力するときには stats.toJson() がデフォルトで使用される。
よくわかんないので要調べ。

#### devtool
devtool の設定によりパフォーマンスが変化する。以下のようなバリエーションがある。
* "eval" has the best performance, but doesn't assist you for transpiled code.
* The cheap-source-map variants are more performant, if you can live with the slightly worse mapping quality.
* Use a eval-source-map variant for incremental builds.

多くの場合、cheap-module-eval-source-map が最適。

#### Avoid Production Specific Tooling
一部の　utilities, plugins, and loaders は、env = production の時だけ有用である。例えば、作成したコードを TerserPlugin を使って圧縮したり分割したりすることは、development 環境下では無意味である。以下のツールはdev環境下では排除されるべき。
* TerserPlugin
* ExtractTextPlugin
* [hash]/[chunkhash]
* AggressiveSplittingPlugin
* AggressiveMergingPlugin
* ModuleConcatenationPlugin

#### Minimal Entry Chunk
webpack は、鮮度の高いチャンクのみをファイルシステムに送る。HMR, [name]/[chunkhash] in output.chunkFilename, [hash] などの configuration option では、エントリーチャンクは

エントリーチャンクは小さくして、送りやすくするのが基本。以下のコードブロックは、他の全てのチャンクを子として指定しているだけなので、

#### Avoid Extra Optimization Steps
optimization を指定することで、サイズやロードのパフォーマンスを改善できるが、大きなコード内では逆効果になりがち。なので、不要な optimization は無効化しておこう。
```js
module.exports = {
  // ...
  optimization: {
    removeAvailableModules: false,
    removeEmptyChunks: false,
    splitChunks: false,
  }
};
```

#### Output Without Path Info
多くのモジュールを含むプロジェクトでは、output bandle にパスを含める機能は、ガベージコレクションに負荷をかけやすい。以下のように無効化しておこう。
```js
module.exports = {
  // ...
  output: {
    pathinfo: false
  }
};
```

#### TypeScript Loader
ここまで来て、Guide ではなくConcept を読むべきだったことに気づく。

# 公式ドキュメント(Concept)
## Concept
以下のコアコンセプトで構成される。
* Entry
* Output
* Loaders
* Plugins
* Mode
* Browser Compatibility

#### Entry
エントリーポイントは、webpack が、どのモジュールから dependency graph の作成を始めたらいいのかを示すもの。デフォルトでは ./src/index.js であるが、他もしくは複数の鳥ポイントを、entry オプションで指定できる。
```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

#### output
output プロパティは、webpack に出力先と出力ファイルの名前を指定する。./dist/main.js がメインの出力ファイルであり、他の出力ファイルも dist 以下に吐き出される。これも設定でいじれる。
```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

一行目は node.js のコアモジュールであり、ファイルパスを操作するために使われる。

#### loaders
webpack は js と json のみを扱うが、loaders によって他のファイルも処理でき、またそれらをアプリ内で使用可能な状態の有効なモジュールに変換し、また dependency graph に追加できる。どんなタイプのファイルでもインポートできるのは、webpack の大きな特徴である。

ローダーは二つのプロパティを持つ。
1. The test property identifies which file or files should be transformed.
2. The use property indicates which loader should be used to do the transforming.

```js
const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```

このコードは以下のようなことを言っている。
```
"Hey webpack compiler, when you come across a path that resolves to a '.txt' file inside of a require()/import statement, use the raw-loader to transform it before you add it to the bundle."
```

このような定義が、rules ではなく module.rules の下で行われることは覚えておくべき。
また、このように正規表現を使う場合、クオートではくくらないことに注意。

#### plugins
loaders が、一定の形式のモジュールを変換するものなのに対し、plugins は bundle optimization や asset management、injection of environment variables のような、もっと幅広いタスクを担当する。

plugins を使用するには、これをrequire() して、plugins 配列に追加する必要がある。plugins の多くはオプションにより拡張可能で、コンフィグの中で異なる用途で複数回利用できる。また、new によりインスタンス化する必要がある。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins

module.exports = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

この例では、html-webpack-plugin は全てのバンドルファイルに対し、必要なHTMLファイルを埋め込んでくれるようになっている。

#### mode
デフォルトは prodution。

#### Browser compatibility
webpack は ES5-comliant なブラウザをサポートしていて、Promise, import(), require() で指定する必要がある。また、古いブラウザもサポートしたい場合は、これらの記述の前に load a polyfill をする必要がある。

## Entry Points
エントリポイントの宣言方法はいくつかある。

#### Single Entry (Shorthand) Syntax
entry: string|Array<string> で。
一つの場合は以下のように{}を省略できるが、複数の場合はその次の例のようにすべき。
```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```
```js
module.exports = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

一つのエントリポイントのみを持つライブラリやアプリケーションなどではこの方法が有効打が、拡張性にかける。

#### Object Syntax
Usage: entry: {[entryChunkName: string]: string|Array<string>}
```js
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js'
  }
};
```

これはより冗長ではあるが、最も拡張性が高い。

#### Multi Page Application
```js
module.exports = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```
これは、三つの dependency graph を作成するということを指定している。マルチページアプリケーションでは、サーバーは新しい html ドキュメントを持ってこようとする。ページはこの新しいドキュメントをロードし直し、アセットは再ダウンロードされる。

optimization.splitChunks は、ページ間で共有するべきコードをバンドルするのに使う。エントリポイントごとにコードやモジュールを使い回すマルチページアプリケーションに適する。

## output
#### Usage
webpack の output に辞書式に入れて設定する。

#### Adnvanced
CDN やハッシュ値を使った方法もある。
```js
module.exports = {
  //...
  output: {
    path: '/home/proj/cdn/assets/[hash]',
    publicPath: 'https://cdn.example.com/assets/[hash]/'
  }
};
```

出力されるファイルの publicPath がコンパイル時には不明である場合は、設定時には blank にしておいてよく、実行時にエントリポイントファイル内の __webpack_public_path__ 変数によって動的に定義できる。

```
__webpack_public_path__ = myRuntimePublicPath;

// rest of your application entry
```

## loaders
モジュールのソースコード上で適用される。import もしくはロードすることで、ファイルの前処理が行える。

#### example
まずインストールする。
```bash
npm install --save-dev css-loader
npm install --save-dev ts-loader
```

で、webpack に .css ファイルに対しては css-loader を、.ts ファイルには ts-loader を使用するよう指定する。

```js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```

これは Configuration と呼ばれる指定方法であり、推奨されるが、他にも以下の二つがある。
* Inline: Specify them explicitly in each import statement.
* CLI: Specify them within a shell command.

#### Configuration
一つの test/use ブロックに複数のローダーを指定することもできる。この場合、一行の中では右から左に、複数行なら下から上の順番で実行される。例えば以下の例なら、sass-css-style loader の順で評価され実行される。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // style-loader
          { loader: 'style-loader' },
          // css-loader
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          // sass-loader
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

#### Inline
import を使う方法。ローダーをリソースから ! で分離し、それぞれの断片は分離前のディレクトリ内で展開される。
```js
import Styles from 'style-loader!css-loader?modules!./styles.css';
```
よくわからないので要調べ。

#### CLI
コマンドラインからもいける。
```js
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

#### Loader Features
* Configuration の例のように、ローダーは連結して記述できる。それぞれのローダーは、処理したリソースを次のローダーに渡す。webpack は最後のローダーが js を返すことを期待する。
* 同期的にも非同期的にもなる。
* Node.js 内で動き、Node.js の仕組みの中でならあらゆることができる。
* ローダーは options オブジェクトでコンフィグすることが可能。
* package.json の loader フィールドにより、一般のモジュールは main の他に loader を出力できる。
* Plugin で loader を拡張することができる。

## plugins
ローダーができないことをやったりする。

#### Anatomy
plugin は apply メソッドを持った js オブジェクトである。この apply メソッドは webpack コンパイラから呼ばれ、全てのコンパイルライフサイクルへのアクセス権を持つ。コードは以下。

```js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, compilation => {
      console.log('The webpack build process is starting!!!');
    });
  }
}
```

5行目の tap メソッドの第一引数はプラグインの名前であり、キャメルケースである必要がある。constant にした方が良いらしい。

#### Usage
プラグインはarguments/options を取ることができるため、webpack.config.js 内では new で指定する必要がある。

#### configuration
```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

#### Node API  
Node API を使用している場合でも、plugins プロパティによりプラグインを指定することができる。

```js
const webpack = require('webpack'); //to access webpack runtime
const configuration = require('./webpack.config.js');

let compiler = webpack(configuration);

new webpack.ProgressPlugin().apply(compiler);

compiler.run(function(err, stats) {
  // ...
});
```

## Configuration
webpack.config.js は js ファイルなので、一般の js ファイルと同じく、以下のことができる。
* import other files via require(...)
* use utilities on npm via require(...)
* use JavaScript control flow expressions, e.g. the ?: operator
* use constants or variables for often used values
* write and execute functions to generate a part of the configuration

ただし、以下のようなことは避けるべき。
* Access CLI arguments, when using the webpack CLI (instead write your own CLI, or use --env)
* Export non-deterministic values (calling webpack twice should result in the same output files)
* Write long configurations (instead split the configuration into multiple files)

#### Simple configuration
```js
var path = require('path');

module.exports = {
  mode: 'development',
  entry: './foo.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'foo.bundle.js'
  }
};
```

#### Multiple Targets
上の single configuration をオブジェクトや関数、Promise 化することで、複数のコンフィグを設定できる。また、js 以外の言語でも記述できる。

## Modules
プログラムは昨日ごとのチャンクに分けられ、モジュールと呼ばれる。それぞれのモジュールは、内包するプログラム全体よりも小さな、検証用、デバッグ用、テスト用の surface area を持つ。良いモジュールはわかりやすい概要とカプセル境界をもっており、一貫したデザインとアプリ中での明確な目的を持つ。

#### What is a webpack Module
Node.js モジュールと比較して、webpack のモジュールは依存関係を様々な方法で表現できる。
* An ES2015 import statement
* A CommonJS require() statement
* An AMD define and require statement
* An @import statement inside of a css/sass/less file.
* An image url in a stylesheet (url(...)) or html (<img src=...>) file.

#### Supported Module Types
loaders は、js 以外の言語で書かれたモジュールの処理方法を示し、バンドルにそれらの依存関係を記述するものといえる。例えば、以下のような言語をサポートしている。
* CoffeeScript
* TypeScript
* ESNext (Babel)
* Sass
* Less
* Stylus

## Dependency Graph
webpack がアプリケーションを処理するとき、コマンドラインから指定されるか config ファイルで指定されたモジュールからエントリーする。このエントリポイントから、webpack は再帰的に dependency graph を作成し、ブラウザが読み込めるように、内包した全てのモジュールを少数のバンドルへとまとめる。

## Targets
js はサーバーとブラウザどちら用にもかけるので、config でターゲットを指定する。

#### Usage
```js
module.exports = {
  target: 'node'
};
```

この例の場合、Node.js-like 環境で使う想定でコンパイルされる。Node.js require で読み込むらしい。さらにこの target は、dployment/environment の使い分けも追加で記述できる。

####Multiple Targets
target プロパティに複数の strings を記述することはできないが、二つのコンフィグファイルをバンドルして isomorphic library にすることで実現できる。

```js
const path = require('path');
const serverConfig = {
  target: 'node',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.node.js'
  }
  //…
};

const clientConfig = {
  target: 'web', // <=== can be omitted as default is 'web'
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.js'
  }
  //…
};

module.exports = [ serverConfig, clientConfig ];
```

この例の場合、lib.js と lib.node.js を dist に生成する。

## Hot Module Replacement
Hot Module Replacement(HMR) とは、アプリケーションが実行中に、完全なリロードを必要とせずにモジュールを交換し、追加し、削除するためのもの。以下の要因から開発の助けとなる。
* Retain application state which is lost during a full reload.
* Save valuable development time by only updating what's changed.
* Instantly update the browser when modifications are made to CSS/JS in the source code, which is almost comparable to changing styles directly in the browser's dev tools.

#### How It Works
* In the Application
以下のようなステップで、モジュールのアプリケーションへの出入りを可能にする。
1. The application asks the HMR runtime to check for updates.
2. The runtime asynchronously downloads the updates and notifies the application.
3. The application then asks the runtime to apply the updates.
4. The runtime synchronously applies the updates.

#### In the compiler
通常のアセットに加え、コンパイラは "update" を発信することでバージョン更新を支持する。"update" は以下の二つに分けられる。
1. The updated manifest (JSON)
2. One or more updated chunks (JavaScript)

マニフェストは、新しいコンパイルのハッシュ値と全ての更新されるチャンクのリストを持つ。更新されるチャンクリストは、更新されるモジュール全てのコード、および削除されたモジュールのフラッグを含んでいる。この更新ビルド中、コンパイラはモジュールとチャンクの ID を、メモリや json ファイルに保持している。

#### In a Module
HMR は HMR コードを含んでいるモジュールのみに影響する opt-in な機能。例として、スタイルをパッチする style-loader が挙げられる。patching を有効化するために、style-loader は HMR インターフェイスを実装しており、HMR から update を受け取ると、古いスタイルを新しいものに変更する。

同様に、モジュール内に HMR インターフェイスが実装されていれば、モジュールが更新された時の処理を記述できる。しかし多くの場合、全てのモジュールに HMR コードを記述する必要はなく、依存関係にあるモジュールのうち一つに記述されていれば良い。

#### In the runtime
module system runtime では、モジュールの親子関係を追跡するコードを発信する。management 側では、check と apply メソッドをサポートする。

checkは update アニフェスとに対し HTTP リクエストを送り、これが失敗すれば更新なしとなり、成功すれば updated chunks リストが現在読み込まれているチャンクと比較される。すでに読み込まれたそれぞれのチャンクに対応する更新チャンクがダウンロードされ、全てのモジュールの更新は runtime に保存される。全ての更新チャンクがダウンロードされ、適用される準備が整うと、rutime は ready 状態へと移行する。

apply は 全ての更新モジュールを無効化する。それぞれの無効モジュールは、それ自体もしくは親モジュールに更新はんどらを必要とする。これがない場合、無効のまま。この無効化は、アプリのエントリポイントが読み込まれるか、更新ハンドラをもつモジュールが読み込まれるまで続く。エントリポイントが読み込まれた場合、更新プロセスが破棄される。

その後、無効モジュールは dispose ハンドラによって破棄され、ハッシュ値が更新され、全ての accept ハンドラが呼ばれる。そして、 runtime は idle 状態へ移行し、プロセス終了。

## Why Webpack
js をブラウザで動かすには、二つの方法がある。一つには昨日ごとのスクリプトを含むこと。この方法、読み込むスクリプトの数が膨大になりがちで拡張性に乏しい。二つ目は、全てのコードを含む大きな js ファイルを作ることであるが、これは可読性とかその辺の問題がある。

#### IIFE's - Immediately invoked function expressions
IIFE は巨大プロジェクトのスコーピングの問題を解決してくれる。IIFE でスクリプト群をラップすると、嬉しいらしい。IIFE を使用する場合、 Make, Gulp, Grunt, Broccoli, Brunch といった、task runners と呼ばれプロジェクトファイルを連結するツールも使用することになる。

連結はファイル間でのスクリプトの使い回しを容易にするが、再ビルドのコストが増加し、また使用されていないコードを見つけるのも困難である。

要はwebpack サイコーや！
