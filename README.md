## 　　Lesson13.　randomユーティリティの選択肢を動的に設定する 
#### 開発メモ
ワークフロー
<br><img width="600" src="https://user-images.githubusercontent.com/40127279/126867726-a3dc3141-44b1-476e-924a-a9eb8b971833.png">

### 1.フローデザイン
　・キーワード起動の引数によって、登録、編集、表示をそれぞれ処理する
<br>　分岐を担うConditionalは下記の通り引数{query}を評価して分岐させています
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126867884-1caeeac6-582a-4d7f-967c-7955c217c713.png">
<br>　　　
<br>　実装で利用した細かな手法
<br>　・登録するURLはSafariから取得する
<br>　・PostNotificaftionに2つのパラメータを渡す（標準出力以外の受け渡しとして変数を使う）
<br>　・Randomユーティリティのリストをファイルから読み込み動的に設定する
<br>　
<br>　実装できなかったもの
<br>　・Alfred環境変数で設定したホームディレクトリ'~'がRunScriptでうまく稼働しない。。。
### 2.PostNotificaftionに2つのパラメータを渡す
　AddLisitの流れを見てください（赤いフロー）
<br>　タイトル取得用とURL取得用として2つRunScriptを配置しています
<br>　はじめのRunScriptでHPのタイトルをechoで標準出力して、ArgandVarsユーティリティで
<br>　その内容を変数titleに格納します（後で通知に利用します）
<br>　
<br>　RunScript
<br>　<img width="600"  src="https://user-images.githubusercontent.com/40127279/126869023-cfc35c27-0bd4-44f2-b7b1-706d2a137908.png">
<br>　ArgandVars
<br>　<img width="600"  src="https://user-images.githubusercontent.com/40127279/126869028-9cddf591-a05d-411e-b174-fb8072217702.png">
<br>　
<br>　次のRunScriptでは、表示しているURLをechoで標準出力して、テキストファイルに追加します
<br>　
<br>　RunScript
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126869033-0c6782e1-a84e-4fc0-9adb-12e58b63d890.png">
<br>　AppendtoFile
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126867999-75a43077-95f4-4ccb-9fa5-dd330efa37a0.png">
<br>　
<br>　URLをAppendtoFileオブジェクトでテキストファイルに追加した後に、PostNotificaftionで
<br>　メッセージを表示させます。このとき
<br>　テキストファイルの場所は、AppendtoFileオブジェクトの標準出力の{query}
<br>　サイトのタイトルは、ArgandVarsで設定した{var:title}を使用しています　
PostNotificaftion
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126869051-9855eb0d-c5f1-48e3-aa3a-834655fea2ab.png">

### 3.ファイルを編集させる
　EditLisitの流れを見てください（白いフロー）
<br>　お気に入りURLのリストを編集させるためにデフォルトアプリで開きます
<br>　
<br>　OpenFile
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126869066-236a49b0-0199-4126-90dc-3045ec52cf22.png">

### 4.Randomユーティリティのリストをファイルから読み込み動的に設定する
　黄色い部分のフローです。
<br>　Lesson12のウィキペディアおまかせ表示では、あらかじめリストを準備しましたが、
<br>　今回はファイルから読み込んで動的にセットします
<br>　
<br>　Randomユーティリティを見てください
<br>　Random:としてword from listを設定してWords:の一覧には{query}とあるだけです
<br>　<img width="480" src="https://user-images.githubusercontent.com/40127279/126869093-5923c0a1-7552-43ba-92d3-4cd1977f3080.png">
<br>　そうです。このオブジェクトの前段のフローであるRunScriptの標準出力にリストを
<br>　書き出してセットするのです
<br>　ではそのRun Scriptを見てみましょう（黄色いRunScriptです）
<br>　
<br>　実はとてもシンプルで、ファイルの内容をechoしているだけです
<br>　こうすることで、ファイルに登録されたテキスト（URL）が全てRandomユーティリティの
<br>　リストにっなるっていう寸法さ
<br>　
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126869112-e08c8e22-4ed1-4347-8708-dea2511b8364.png">
<br>　
<br>　でもなぜか、workflowの環境変数では"~"をセットしてもうまく動きません
<br>　"Users/..."と書けば環境変数でも使えましたが、その場合配布した先の環境では
<br>　ホームディレクトリの名称が私のものとは違うのでうまく動かないでしょう
<br>　仕方なくスクリプトの中で直接"~"を書いています
<br>　
<br>　Randomユーティリティのリスト自体がURLですので、あとはOpenURLオブジェクトで
<br>　表示させればOKです
<br>　<img width="600" src="https://user-images.githubusercontent.com/40127279/126869128-4923564e-0dd9-4792-b0db-355018a6410d.png">

#### 背景
　Lesson12のおまかせウィキペディアを作成した際に、ウィキペディアの高度な検索結果を
<br>　表示させたいと思いました
<br>　例えば『ウィキペディアのタイトルに”効果”か”現象”を含む記事を更新が新しい順に検索』
<br>　みたいな。でもこの場合、フィードバックとしてLargeTypeするURLが見ただけではわからず
<br>　あまりふさわしくない感じでした
<br>　そこでこのワークフローを思いつきました
<br>　お気に入りページというか、面白いと思った記事を保存して、後で適当に読むというような
<br>　利用を想定しています
#### 取扱説明
### 機能:
　お気に入りページの登録・ランダム表示
### インストール:
　1.[Alfredworkflow](https://github.com/KitanoTamotsu/favo/releases/download/1.1/Favorite.Pages.alfredworkflow.zip)をダウンロード 
<br>　2.ファイルをダブルクリックしてワークフローに登録
### 使い方:
　キーワード『favo』で起動
<br>　以下のパラメータを指定できます
<br>　パラメータなし　→ 追加したページをランダムに表示
<br>　aもしくはadd 　→ Safariで閲覧しているページをテキストファイルに追加
<br>　eもしくはedit　→ 追加したテキストファイルを表示（リストの修正や削除ができます）
### 動き方：
　1.テキストファイル（~/Documents/Alfred/favo/favolist.txt）にURLを
<br>　　保存します（1行に1URLとして記述）
<br>　2.ファイルの登録内容を読み込みランダム表示

#### 修正履歴
### ver1.1(2021-04-04)：
　シェルスクリプトをbashからzshに変更しました
<br>
<br>
[トップページに戻る](https://kitanotamotsu.github.io/)

