# つぶやきを保存するElectron版10

　

<!-- more -->

# ブツ

* [リポジトリ][]

[リポジトリ]:https://github.com/ytyaru/Electron.MyLog.20220910101121

## インストール＆実行

```sh
NAME='Electron.MyLog.20220910101121'
git clone https://github.com/ytyaru/$NAME
cd $NAME
npm install
npm start
```

### 準備

1. [GitHubアカウントを作成する](https://github.com/join)
1. `repo`スコープ権限をもった[アクセストークンを作成する](https://github.com/settings/tokens)
1. [インストール＆実行](#install_run)してアプリ終了する
	1. `db/setting.json`ファイルが自動作成される
1. `db/setting.json`に以下をセットしファイル保存する
	1. `username`:任意のGitHubユーザ名
	1. `email`:任意のGitHubメールアドレス
	1. `token`:`repo`スコープ権限を持ったトークン
	1. `repo.name`:任意リポジトリ名
	1. `address`:任意モナコイン用アドレス
1. `dst/mytestrepo/.git`が存在しないことを確認する（あれば`dst`ごと削除する）
1. GitHub上に同名リモートリポジトリが存在しないことを確認する（あれば削除する）

### 実行

1. `npm start`で起動またはアプリで<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>R</kbd>キーを押す（リロードする）
1. `git init`コマンドが実行される
	* `repo/リポジトリ名`ディレクトリが作成され、その配下に`.git`ディレクトリが作成される
1. [createRepo][]実行後、リモートリポジトリが作成される

### GitHub Pages デプロイ

　アップロードされたファイルからサイトを作成する。

1. アップロードしたユーザのリポジトリページにアクセスする（`https://github.com/ユーザ名/リポジトリ名`）
1. 設定ページにアクセスする（`https://github.com/ユーザ名/リポジトリ名/settings`）
1. `Pages`ページにアクセスする（`https://github.com/ユーザ名/リポジトリ名/settings/pages`）
    1. `Source`のコンボボックスが`Deploy from a branch`になっていることを確認する
    1. `Branch`を`master`にし、ディレクトリを`/(root)`にし、<kbd>Save</kbd>ボタンを押す
    1. <kbd>F5</kbd>キーでリロードし、そのページに`https://ytyaru.github.io/リポジトリ名/`のリンクが表示されるまでくりかえす（***数分かかる***）
    1. `https://ytyaru.github.io/リポジトリ名/`のリンクを参照する（デプロイ完了してないと404エラー）

　すべて完了したリポジトリとそのサイトが以下。

* [作成DEMO][]
* [作成リポジトリ][]

[作成DEMO]:https://ytyaru.github.io/Electron.MyLog.20220908121018.Site/
[作成リポジトリ]:https://github.com/ytyaru/Electron.MyLog.20220908121018.Site

# やったこと

* リファクタリング
    * テキストエリアの字数・行数制限
        * `renderer.js`
            * `valid()`関数作成
        * CSS
            * `.warning`, `.error`を`style.css`から`ui.css`に移した
    * `innerHTML`を[`insertAdjacentHTML`][]に修正した
        * 指定箇所にHTML文字列を挿入できる（既存要素破壊せず高速なためかちらつきが発生しない）
* バグ修正
    * URLなど連続した半角英数字がボーダーを超えてしまう件
        * [情報源](https://qiita.com/akane_kato/items/2b1385574e1a1babdde1#comment-6ab0c5eb9189b3087baa)
        * `style.css`
            * `line-break:anywhere;`
            * `white-space: break-spaces;`
    * 閉じタグのスラッシュ忘れ
        * `<span>あと<span id="content-length"></span>字<span id="content-line"></span>行</span>`
* つぶやきプレビュー機能追加

[`insertAdjacentHTML`]:https://developer.mozilla.org/ja/docs/Web/API/Element/insertAdjacentHTML

# つぶやきプレビュー機能追加

　テキストエリアに入力したテキストから最終表示をプレビューする機能を追加した。

　表示される位置はテキストエリアの直下で、`opacity:0.5`により半透明で表示される。

![preview][]

[preview]:memo/preview.png

## ちらつきを抑えるために

　なるだけちらつきを抑えるのに苦労した。最初は以下のように、つぶやき一件まるごとHTMLをそのままプレビューDOMにぶちこんでいた。

```javascript
innerHTML = TextToHtml.toHtml()
```

　だが、モナコインアドレスを設定しているとき、投げモナボタンが表示され、文字を入力するたびに画像がちらついてしまった。

　これを解決するため、画像だけは初回だけDOM生成するよう修正した。`renderer.js`, `text-to-html.js`

```javascript
document.querySelector('#content').addEventListener('input', async(e)=>{
    ...
    if (0 === e.target.value.length) {
        document.querySelector('#preview').innerHTML = ''
    } else {
        if (0 === document.getElementById('preview').children.length) {
            document.getElementById('preview').insertAdjacentHTML('afterbegin', TextToHtml.toBody(0, Math.floor(new Date().getTime()/1000), document.querySelector('#address').value))
        }
        document.querySelector('#preview div.mylog p').innerHTML = TextToHtml.toText(e.target.value)
        document.querySelector('#preview div.mylog time').remove()
        document.querySelector('#preview div.mylog-meta').insertAdjacentHTML('afterbegin',TextToHtml.toTime(Math.floor(new Date().getTime()/1000)))
    }
})
```

# 折り返し

　つぶやきの内容がちゃんと折り返しされずボックスを超過して表示されてしまう問題があった。これを解決した。

対応|状態
----|----
前|![line-break-before.png][]
後|![line-break-after.png][]

[line-break-before.png]:memo/line-break-before.png
[line-break-after.png]:memo/line-break-after.png

　結論からいうとstyle.cssの`div.mylog`に以下を追記した。

```css
div.mylog {
  overflow-wrap: break-word;
  line-break: anywhere;
  white-space:break-spaces;
}
```

CSS|意味
---|----
[`overflow-wrap: break-word;`][]|ボックスからはみ出ないようにする
[`word-wrap: break-word;`][]|ボックスからはみ出ないようにする（IE後方互換用）
[`line-break: anywhere;`][]|句読点や記号を用いたときの改行規則 (禁則処理) を設定する
[`white-space: break-spaces;`][]|空白文字の折りたたみや折り返しを設定する

[`overflow-wrap: break-word;`]:https://developer.mozilla.org/ja/docs/Web/CSS/overflow-wrap
[`word-wrap: break-word;`]:http://www.htmq.com/style/word-wrap.shtml
[`line-break: anywhere;`]:https://developer.mozilla.org/ja/docs/Web/CSS/line-break
[`white-space: break-spaces;`]:https://developer.mozilla.org/ja/docs/Web/CSS/white-space

　じつはこの折り返しがとても複雑。一見、[`overflow-wrap: break-word;`][]すれば解決するようにみえるが、今回の場合は折り返しされなかった。[`line-break: anywhere;`][]を追加することで解決した。ついでに連続した空白文字も折り返し対象にすべく[`white-space: break-spaces;`][]した。

　[`word-wrap: break-word;`][]はIEでのみ有効。もうサポート期間が終了したので無視してもいいが、念の為いれておいた。

　詳しくは以下参照。

* [overflow-wrap: break-word; や word-break: break-all; が万能の改行処理だったなら、こんなに苦労していない][]

[overflow-wrap: break-word; や word-break: break-all; が万能の改行処理だったなら、こんなに苦労していない]:https://qiita.com/akane_kato/items/2b1385574e1a1babdde1

　`word-break: break-all;`なるコードもある。が、これは禁則処理を壊してしまうので非推奨。

* [word-breakとword-wrapはややこしい][]

CSS|意味
---|----
`word-break: break-all;`|禁則処理を解除し、どの文字の間でも改行する
`word-wrap: break-word;`|ボックスからはみ出ないようにする（IE後方互換用）

[word-breakとword-wrapはややこしい]:https://w3g.jp/blog/confusing_word-break_word-wrap

