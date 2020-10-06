# Node-RED オリジナルノードの開発

## はじめに
本ハンズオンは、Node-RED公式サイトを元に、手順をまとめ直したものになります。ここでは、初めてノード開発を行う方向けに手順をシンプルにしています。
オリジナルの情報は [Node-RED公式サイト ノードの開発](https://nodered.jp/docs/creating-nodes/) を御覧ください。


## 全体の流れ
ノード開発の主な流れは以下の通りです。
1. ノードプログラム開発(JavaScript)
2. ノード外観作成(HTML)
3. パッケージ化
4. ノードのインストール
5. ノード名の変更
6. ノード処理の変更
7. ノードアイコン変更
8. ノード色変更
9. フローライブラリへの公開
10. 公開したノードの削除
11. 公開したノードのインポート

## ノード開発のポリシー
新しいノードを作成する時には、いくつかの一般的なルールに従う必要があります。これらはコアノードで採用されているアプローチに準拠しており、一貫したユーザーエクスペリエンズを提供します。

ノード作成においては以下のルールに従います:

* 目的が明確に定義されていること
    * APIに含まれるすべての項目を設定できるようにしたノードは、 単一の目的のために分割した複数のノードよりも使いにくいことが多いです。

* 元の機能に関係なく簡単に使えること
    * 複雑さを隠蔽して専門用語やドメイン固有の知識の使用を避けます。

* 多様なメッセージ型が渡されても正常に処理できること
    * メッセージは文字列、数値、Boolean、Buffer、オブジェクト、配列、nullなどの様々な型で渡される可能性があります。 いずれの型であっても正しく処理される必要があります。

* 送信される内容に一貫性を持つこと
    * ノードはメッセージにどのようなプロパティを追加するのかを文書化する必要があり、動作において一貫性があり、 予測可能でなければなりません。

* フローの先頭、中間、または末尾に配置されるようにすること - すべてを一度にしようとしないこと

* エラーをキャッチすること
    * ノードがキャッチできないエラーをスローすると、 Node-REDはシステム全体の状態がわからなくなるため停止します。
    * ノードはできるかぎり、 エラーをキャッチするか非同期呼び出しのためのエラーハンドラを登録しなければなりません。


## 1. ノードプログラム開発(JavaScript)
Node-REDのノードは、処理を定義するJavaScriptファイルと、設定画面などのUIを提供するhtmlファイルの2つから構成されています。
JavaScriptでは、前のノードから渡されたメッセージに対して行う処理を実装します。htmlファイルは、Node-REDのフローエディタで表示されるプロパティ設定画面が記述されています。このhtmlファイルで表示したプロパティ設定画面で入力した設定値をjsファイルから呼び出して処理を行います。

### 1-1. 開発用リポジトリの準備
https://github.com/ へアクセスし、GitHubアカウントでログインします。
GitHubページの右上にある「＋」プルダウンから「New repository」を選択します。

![](./images/img01.png)

ここで作成するリポジトリは、ノードを開発するためのプロジェクトとして存在し、その後パッケージ化されてnpmへ公開されるものになります。（公開するかどうかはもちろん任意です）

ですので、リポジトリ名はノード開発の命名規則に沿ったものにしましょう。

ルールでは「node-red-contrib-<ノードのグループを表す名称>ですので、これに従います。以下の画像ので例では「node-red-contrib-taiponrock」にしています。
リポジトリ名を指定したら、リポジトリ公開範囲をPublicにし、READMEファイルのチェックをONにして、ライセンスを指定します。例ではApache License 2.0ライセンスで作成します。
すべて設定したら「Create repository」ボタンをクリックしてリポジトリを作成してください。

![](./images/img02.png)

無事リポジトリが作成されました。

![](./images/img03.png)

### 1-2. リポジトリのClone
では、次にローカルの開発環境に、先程作成したリポジトリをクローンしましょう。

リポジトリのURLをクリップボードへコピーします。緑色の「Code」のプルダウンをクリックして、クリップボードボタンをクリックしてURLをコピーします。

![](./images/img04.png)

リポジトリをローカルに取得(git clone)
Bashが実行できるコマンドラインインターフェース（ターミナルなど）からリポジトリをクローンする（コピーする）作業ディレクトリへ移動します。ここでは、ユーザーディレクトリの下へworkディレクトリを作成し、そこへ移動しました。

```
$ mkdir work
$ cd work
```

![](./images/img05.png)

先程作成したリポジトリのURLで```git clone```コマンドを実行してください。クローンが終了したら、```ls```コマンドで正常にクローンされていることを確認しましょう。

```
git clone https://github.com/<GitHubアカウント>/node-red-contrib-<指定した任意の文字列>.git
```

![](./images/img06.png)


### 1-3. Javascriptの作成
ここからは、実際のノードの処理を作成します。と、言ってもすでにコードは用意してありますのでご安心ください。
まずは、クローンしたリポジトリのディレクトリの中へ移動します。

```
cd node-red-contrib-<指定した任意の文字列>
```

![](./images/img07.png)

こちらのディレクトリ配下に、以下のコードの通り **node.js** というファイル名で、ファイルを作成します。
```
module.exports = function(RED) {
    function LowerCaseNode(config) {
        RED.nodes.createNode(this,config);
        var node = this;
        node.on('input', function(msg) {
            msg.payload = msg.payload.toLowerCase();
            node.send(msg);
        });
    }
    RED.nodes.registerType("lower-case",LowerCaseNode);
}
```

![](./images/img08.png)

node.jsが作成されました。

![](./images/img09.png)

ここではviを使って作成していますが、もちろん任意のエディターを使って構いません。

### 1-4. HTMLの作成
続いて、同じディレクトリ配下に、以下のコードの通り **node.html** というファイル名で、ファイルを作成します。

```
<script type="text/javascript">
    RED.nodes.registerType('lower-case',{
        category: 'function',
        color: '#a6bbcf',
        defaults: {
            name: {value:""}
        },
        inputs:1,
        outputs:1,
        icon: "file.png",
        label: function() {
            return this.name||"lower-case";
        }
    });
</script>

<script type="text/x-red" data-template-name="lower-case">
    <div class="form-row">
        <label for="node-input-name"><i class="icon-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
</script>

<script type="text/x-red" data-help-name="lower-case">
    <p>A simple node that converts the message payloads into all lower-case characters</p>
</script>
```

![](./images/img10.png)

node.htmlが作成されました。

![](./images/img11.png)

## 2. パッケージ化
## 3. ノードのインストール
## 4. ノード名の変更
## 5. ノード処理の変更
## 6. ノードアイコン変更
## 7. ノード色変更
## 8. フローライブラリへの公開
## 9. 公開したノードの削除
## 10. 公開したノードのインポート
