---
id: dist
{{if .Meta.Status}}status: {{.Meta.Status}}{{end}}
{{if .Meta.Summary}}summary: {{.Meta.Summary}}{{end}}
{{if .Meta.Author}}author: {{.Meta.Author}}{{end}}
{{if .Meta.Categories}}categories: {{commaSep .Meta.Categories}}{{end}}
{{if .Meta.Tags}}tags: {{commaSep .Meta.Tags}}{{end}}
{{if .Meta.Feedback}}feedback link: {{.Meta.Feedback}}{{end}}
{{if .Meta.GA}}analytics account: {{.Meta.GA}}{{end}}

---
# FriendlyEats

## FriendlyEats について

このcodelabでは、Firebase / Cloud Firestoreを使ったグルメサイトのwebアプリを作成します。
Cloud Firestoreを学習するために最小限のプログラムをするだけでCloud Firestoreを使ったアプリケーションを作ることができます。

<img width="715" alt="sample.jpg" src="https://codelabs.developers.google.com/codelabs/firestore-web/img/89e58de6c41cee60.png">

このチュートリアルでは以下のことを学習します。
- WebアプリケーションからCloud Firestoreへの読み書きをする
- リアルタイムにCloud Firestoreのデータの変更を受け取る
- Firebaseのユーザ認証を使ったり、Security Rulesを使ってCloud Firestoreのデータを安全に読み書きする
- Cloud Firestoreの複雑なクエリーを書く

このチュートリアルを始めるに当たって、必要な開発環境は以下となります。

- Node.jsとnpm \- Nodeはversion 8をお薦めします
- IDEやテキストエディタ。たとえば Emacs, vim, WebStorm, Atom, VS Code, Sublime などからお好きなものを選んでください


## Firebase projectの作成と設定

### Firebase projectを作成する
1. [Firebaseのコンソール](https://console.firebase.google.com)上で「プロジェクトを追加」をクリックします
1. プロジェクト名前を入力します。「FriendlyEats」と入力してください
1. 入力したプロジェクト名の下にプロジェクトIDが表示されます（変更可能です）
作成プロジェクトIDは忘れないように！
1. [続行]をクリックします
1. Google アナリティクス画面で「今は必要ない」を選択します
1. [プロジェクトを作成]をクリックします
1. 「新しいプロジェクトの準備ができました」が表示されます。[続行]をクリックします

> 重要: 作成された Firebaseのプロジェクトは「FriendlyEats」という名前ですが、Firebaseは自動的に「friendlyeats-1234」のような固有のプロジェクトIDを割り当てます。この固有のIDは、あなたのプロジェクトを識別するのに必要です（CLIなどで）。「FriendlyEats」は単にプロジェクトの名前です。

これから作成するアプリケーションでは、ウェブ上で使えるFirebaseのサービスのうちいくつかを利用します。

- Firebase Authentication \- ユーザーを簡単に管理/識別します
- Cloud Firestore \- クラウド上に構造化されたデータを保存し、データが更新された時は即座に通知します
- Firebase Hosting \- 静的なコンテンツをホスティングします

以下では、Firebaseコンソールを用いた「Firebase Auth」および「Cloud Firestore」の設定方法について、順を追って説明します。

### Anonymous Auth (匿名認証)を有効にする

認証はこのチュートリアルの焦点ではありませんが、何らかの形式の認証を使用することは重要です。
このアプリでは、匿名ログインを使用します。つまりユーザーは明示的な操作をすることなくログインします。
 
そのためには、匿名ログインを有効にする必要がありまず。

1. ブラウザで、[Firebaseのコンソール](https://console.firebase.google.com)を表示します
1. 左のナビゲーションメニュー「開発」の「Authentication」をクリックします
1. 「ログイン方法」タブをクリックします
1. 「ログインプロバイダ」の「匿名」をクリックし「有効」にしてください
1. 最後に[保存]をクリックします

![Anonymous Auth](https://codelabs.developers.google.com/codelabs/firestore-web/img/fee6c3ebdf904459.png)


これでユーザーがWebアプリにアクセスするときに、匿名でログインできるようになりました。詳細は、[匿名認証のドキュメント](https://firebase.google.com/docs/auth/web/anonymous-auth)をお読みください。
 
### Cloud Firestoreを有効にする
このアプリは、レストランの情報や評価を保存、更新情報を受け取るために、Cloud Firestore（データーベース）を使います。

そのためには、Cloud Firestoreを有効にする必要があります。

1. ブラウザで、Firebaseコンソールを表示します
1. 左のナビゲーションメニュー「開発」の「Database」をクリックします
1. Cloud Firestoreペインで「データベースの作成」をクリックします
![Firestore](https://codelabs.developers.google.com/codelabs/firestore-web/img/8c5f57293d48652.png)
1. オプションの「テストモードで開始」を選択し、セキュリティルールに関する免責事項を読んだ後、[次へ]をクリックします
1. ロケーションを選択し（デフォルトのままでも構いませんが、後から変更することはできません）、[完了]をクリックします

テストモードでは、開発中にCloud Firestoreへ書き込みが自由にできるようになります。セキュリティを強化は、このチュートリアルの後半でおこないます。

![Firestore](https://codelabs.developers.google.com/codelabs/firestore-web/img/620b95f93bdb154a.png)


## サンプルのソースコード取得とインストール

### ソースコードを取得する
以下のコマンドを使って GitHub レポジトリをクローンします

```
git clone https://github.com/firebase/friendlyeats-web
```
* 自分の変更をGitHubで管理したい場合には、Forkしてcloneしてください


サンプルコードは📁friendlyeats-webディレクトリにCloneする必要があります。
以後、このディレクトリ内でコマンドラインを実行してください。


```
cd friendlyeats-web 
```

### スターターアプリをインポートする

IDE（WebStorm、Atom、Sublime、Visual Studio Code ...）を使用している場合、📁friendlyeats-web ディレクトリを開くかインポートします。このディレクトリには、レストラン情報とオススメ情報を表示するアプリの未完成なモックコードが含まれています。チュートリアルを通してこのアプリを実装していくので、このモックコードを編集できる必要があります。


## Firebase CLI (コマンドラインツール)のインストール

Firebaseコマンドラインインターフェイス（CLI）を使用すると、Webアプリをローカルで開発したり、Firebase Hostingにデプロイすることができます。

Note: CLIをインストールするには、通常NodeJSに付属しているnpmをインストールする必要があります。

1 . 次のnpmコマンドを実行して、CLIをインストールします。

```
npm -g install firebase-tools
```

> 動作しませんか？ npmのpermissionを変更する必要がある場合があります。

2 . 次のコマンドを実行して、CLIが正しくインストールされたことを確認します。

```
firebase --version
```

Firebase CLIのバージョンがv6.2.0以降であることを確認してください。

3 . 次のコマンドを実行して、Firebase CLIを認証します。

```
firebase login
```

Firebase Hostingのアプリの設定をアプリのローカルディレクトリとファイルから取得するように、ウェブアプリテンプレートを設定しました。ただし、これを行うには、アプリをFirebaseプロジェクトに関連付ける必要があります。


4 . コマンドラインが、先ほどcloneしたディレクトリーになっているか確認してください

5 . 次のコマンドを実行して、アプリをFirebaseプロジェクトに関連付けます。

```
firebase use --add
```

6 . プロンプトが表示されたら、本プロジェクトのプロジェクトIDを選択し、Firebaseプロジェクトにエイリアスを指定します。
エイリアスは、複数の環境（本番、ステージングなど）を切り替える場合に役立ちます。ただし、このチュートリアルでは、`default`というエイリアス名を入力します（スペルを間違えると後にインデックスをデプロイする操作等でエラーが発生するので注意してください）。

7 . コマンドラインの残りの指示に従ってください。

## ローカルでサーバを起動する
アプリで実際に作業を開始する準備ができました！アプリをローカルで実行しましょう！

1 . 次のコマンドをローカルのCLIで実行します:

```
firebase serve --only hosting
```

2 . 成功すると次の文を含むメッセージが表示されます

```
hosting: Local server: http://localhost:5000
```

Firebase Hosting エミュレータがローカルで起動しています。 ブラウザ http://localhost:5000 を開くとサンプルアプリを見ることができます。

3 . ブラウザで http://localhost:5000 を見る

クラウド上のFirebaseプロジェクトに接続されているFriendlyEatsアプリが表示されます（初回起動時はしばらく時間がかかる場合があります）。

アプリは自動的にクラウド上のFirebaseプロジェクトに接続し、匿名ユーザーとしてサインインしました。

![WebApp](https://codelabs.developers.google.com/codelabs/firestore-web/img/6a5dd9ed3f3188e3.png)


## Cloud Firestoreへデータの書き込み

このセクションでは、Cloud Firestoreにデータを書き込みます。Firebaseコンソール上で手動でデータ入力を行うこともできますが、Cloud Firestoreの基本的な書き込みを学習する為に、アプリ自体でデータ生成/入力を行います。

### データモデル

Firestoreデータは、コレクション、ドキュメント、フィールド、およびサブコレクションで構成されています。各レストラン情報をドキュメントとして、`restaurants`と呼ばれる最上位のコレクションに保存します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/25071/d45258ad-7389-7cc4-60ec-98f218f1a9a5.png" width=50%>

そして、各レストランのレビューを`ratings`と名付けたサブコレクションに保存します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/25071/a94dc013-8ea3-8b84-a074-a730ad75da76.png" width=50%>

> Tip: Firestoreデータモデルの詳細については、ドキュメントのドキュメントとコレクションをご覧ください。

### Firestoreにレストラン情報を追加する

このアプリの主なモデルオブジェクトはrestaurantです。`restaurants`コレクションにレストランのドキュメントを追加するコードを書きましょう。

1. cloneしたソースコードの `scripts/FriendlyEats.Data.js` ファイルを開きます
1. `FriendlyEats.prototype.addRestaurant` 関数を探します
1. 関数全体を以下のコードに置き換えます

[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L18-L22.js)

```
FriendlyEats.prototype.addRestaurant = function(data) {
  var collection = firebase.firestore().collection('restaurants');
  return collection.add(data);
};
```

上記のコードにより、`restaurants`コレクションに新しいドキュメント(データ)が追加されます。ドキュメントのデータはJavaScriptオブジェクトです。

この関数は、次のような処理をします。

1. レストランのデータを引数として取得します
1. Cloud Firestoreの`restaurants`コレクションへの参照を取得します
1. 引数で受け取ったデータは、レストランオブジェクトとしてランダムに自動生成し、ドキュメントに追加します


### restaurants情報を追加しよう!

1. ブラウザのFriendlyEatsアプリに戻り、画面を更新します
1. 「IMPORT DATA」をクリックします

まだ画面には何も表示されませんが、Cloud Firestoreにはデータが登録されているはずです。

実際にみてみましょう。

Firebaseコンソールの「Cloud Firestore」タブに移動すると、`restaurants`コレクションに新しいドキュメントが表示されます。

![console](https://codelabs.developers.google.com/codelabs/firestore-web/img/f06898b9d6dd4881.png)

おめでとうございます！！WebアプリからCloud Firestoreにデータを書き込みが成功しました！！

次のセクションでは、Cloud Firestoreからデータを取得してアプリに表示する方法を学習します。


## Cloud Firestore のデータを表示

このセクションでは、Cloud Firestoreからデータを取得してアプリに表示する方法を学習します。 2つの重要な手順は、クエリの作成とスナップショットリスナーの追加です。このリスナーには、クエリに一致するすべての既存データが通知され、更新をリアルタイムで受信します。

最初に、レストランのデフォルトのフィルタリングされていないリストを提供するクエリを作成しましょう。

1. `scripts/FriendlyEats.Data.js` ファイルを開きます
1. `FriendlyEats.prototype.getAllRestaurants` 関数を探します
1. 関数全体を以下のコードに置き換えます


[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L24-L28.js)

```
FriendlyEats.prototype.getAllRestaurants = function(renderer) {
  var query = firebase.firestore()
      .collection('restaurants')
      .orderBy('avgRating', 'desc')
      .limit(50);

  this.getDocumentsInQuery(query, renderer);
};
```

上記のコードでは、`restaurants`という名のトップレベルコレクションから最大50件のレストランを取得するクエリを作成しています。これらは評価の平均順（現在はすべてゼロ）に並べられています。このクエリを定義後、データの読み込みとレンダリングを行う`getDocumentsInQuery`関数にこのクエリを渡します。

これを行うには、スナップショットリスナーを追加します。

- `scripts/FriendlyEats.Data.js` を開きます
- `FriendlyEats.prototype.getDocumentsInQuery` 関数を探します
- 関数全体を以下のコードに置き換えます

[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L30-L34.js)

```
FriendlyEats.prototype.getDocumentsInQuery = function(query, renderer) {
  query.onSnapshot(function(snapshot) {
    if (!snapshot.size) return renderer.empty(); // Display "There are no restaurants".

    snapshot.docChanges().forEach(function(change) {
      if (change.type === 'removed') {
        renderer.remove(change.doc);
      } else {
        renderer.display(change.doc);
      }
    });
  });
};
```

上記のコードでは、クエリの結果に変更があるたびに`query.onSnapshot`をコールバックで呼び出します。

- 最初のコールバックは、クエリの結果、全体のデータを`snapshot`として渡します。これは、Cloud Firestoreの`restaurants`コレクション全体(50件)を意味します。そして`change`には、全ての個々のドキュメントが渡され、それを`renderer.display`関数に渡します。
- ドキュメントが削除された時には、`change.type`は`removed`となります。したがって、この場合、UIからレストランを削除する関数を呼び出します。

両方のメソッドを実装したので、アプリを更新し、Firebaseコンソールで前に表示したレストラン情報がWebアプリに表示されていることを確認します。このセクションを正常に完了した場合、WebアプリはCloud Firestoreでデータを読み書きできています。

レストランのリストが変更されると、このリスナーは自動的にデータを更新します。
Firebaseコンソールに移動して、レストランを手動で削除するか、名前を変更してみてください。サイト上のデータも更新されます。

Note: `Query.get()`メソッドを使用することにより、更新通知を常時リアルタイムに受け取るのではなく、Cloud Firestoreからドキュメントを一度だけ取得することもできます。

![FriendlyEats](https://codelabs.developers.google.com/codelabs/firestore-web/img/89e58de6c41cee60.png)

## データを取得する

ここまでは、`onSnapshot`を使用して更新をリアルタイムで取得する方法を実装しました。
つぎは、アプリ内の特定のレストランをクリックした時にトリガーされる機能を実装しましょう。

1. `scripts/FriendlyEats.Data.js` を開きます
1. `FriendlyEats.prototype.getRestaurant`関数を探します
1. 関数全体を以下のコードに置き換えます

[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L36-L40.js)

```
FriendlyEats.prototype.getRestaurant = function(id) {
  return firebase.firestore().collection('restaurants').doc(id).get();
};
```

このメソッドを実装すると、各レストランのページを表示できるようになります。リスト内のレストランをクリックするだけで、レストランの詳細ページが表示されます。

![SpotBest](https://codelabs.developers.google.com/codelabs/firestore-web/img/15fc40f2fd41a8dc.png)

現時点では評価を追加することはできませんが、この機能はチュートリアルの後半で実装します。


## データのソートと絞り込み

今のところ、アプリにはレストランのリストが表示されていますが、ユーザーがニーズに基づいてフィルタリングする方法はありません。このセクションでは、Cloud Firestoreの高度なクエリを使用してフィルタリングを有効にします。

すべての「点心（Dim Sum）」レストランを取得する簡単なクエリの例を次に示します。

```
var filteredQuery = query.where('category', '==', 'Dim Sum')
```

その名前が示すように、`where()` メソッドは、条件に一致するフィールドを持つコレクション内のドキュメントを取得します。この場合、カテゴリが「点心（Dim Sum）」のレストランのみを取得しています。

このアプリでは、ユーザーは複数のフィルターをチェーンして、「サンフランシスコのピザ」や「人気のあるロサンゼルスのシーフード」などの特定のクエリを作成できます。

それでは、ユーザーが選択した複数の条件に基づいてレストランをフィルタリングするクエリを作成するメソッドを作成してみましょう。

1. `scripts/FriendlyEats.Data.js` を開きます
1. `FriendlyEats.prototype.getFilteredRestaurants`を探します
1. 関数全体を以下のコードに置き換えます

[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L42-L46.js)

```
FriendlyEats.prototype.getFilteredRestaurants = function(filters, renderer) {
  var query = firebase.firestore().collection('restaurants');

  if (filters.category !== 'Any') {
    query = query.where('category', '==', filters.category);
  }

  if (filters.city !== 'Any') {
    query = query.where('city', '==', filters.city);
  }

  if (filters.price !== 'Any') {
    query = query.where('price', '==', filters.price.length);
  }

  if (filters.sort === 'Rating') {
    query = query.orderBy('avgRating', 'desc');
  } else if (filters.sort === 'Reviews') {
    query = query.orderBy('numRatings', 'desc');
  }

  this.getDocumentsInQuery(query, renderer);
};
```

上記のコードは、複数の`where`フィルターと1つの`orderBy`を追加して、ユーザー入力に基づいて複合クエリを作成します。このクエリは、ユーザーの要件に一致するレストランのみを返します。

ここで、ブラウザでFriendlyEatsアプリを更新し、価格や都市などのカテゴリでフィルタリングできることを確認しようとしても、まだ完全には動きません。検索結果は「Your Cloud Firestore has no documents in /restaurants/」と表示されます。また、ブラウザのJavaScriptコンソールに次のようなエラーが表示される場合があります。

```
The query requires an index. You can create it here: https://console.firebase.google.com/project/.../database/firestore/indexes?create_index=...
```

このエラーが発生する理由は、Cloud Firestoreでほとんどの複合クエリにインデックスが必要なのですが、それをまだ用意していないためです。クエリの際にインデックスを必要とすることで、規模が拡大してもCloud Firestoreを高速に保ちます。

次のセクションでは、このアプリケーションに必要なインデックスを作成してデプロイします。

## Cloud Firestoreにindexを追加

アプリ内のすべてのパスを探索し、各インデックス作成リンクをたどる必要がない場合は、Firebase CLIを使用して多数のインデックスを一度に簡単に展開できます。

ダウンロードしたソースコードのルートディレクトリに、firestore.indexes.jsonファイルがあります。このファイルには、フィルターに必要なすべてのインデックスが記述されています。

[firestore.indexes.json](https://github.com/firebase/friendlyeats-web/blob/master/firestore.indexes.json)

```
{
 "indexes": [
   {
     "collectionGroup": "restaurants",
     "queryScope": "COLLECTION",
     "fields": [
       { "fieldPath": "city", "order": "ASCENDING" },
       { "fieldPath": "avgRating", "order": "DESCENDING" }
     ]
   },

   ...

 ]
}
```

次のコマンドでこれらのインデックスをデプロイします。

```
firebase deploy --only firestore:indexes
```
数分後、インデックスが有効になり、エラーメッセージが消えます。


> Tip: Cloud Firestoreのインデックスの詳細については、ドキュメントをご覧ください。

## トランザクションを使ってデータの書き込み
このセクションでは、ユーザーがレストランにレビューを書き込みする機能を実装します。

今までのところ、書き込みはすべてアトミックで比較的単純です。もし書き込みエラーが発生した場合でも、おおむね単にユーザーに再試行を促すか、でなければアプリ自身が自動的に再試行するでしょう。

しかし、このアプリにはレストランの評価を追加したいユーザーが多数いるため、読み取りと書き込みが複数回あった場合、それらの調整する必要があります。つまり、最初にレビューが作成されなければならず、次いでレストランの評価数 `count` と平均評価 `average rating` を更新する必要があります。そしてこれらの操作のうち、どれか1つが失敗し、他が成功した場合、データベースのある部分のデータが別の部分のデータと一致しない、矛盾した状態になります。

幸いなことに、Cloud Firestoreには、単一のアトミック操作で複数の読み取りと書き込みを可能にするトランザクション機能が用意されており、これによりデータの一貫性を維持できます。

1. `src/FriendlyEats/FriendlyEats.Data.js` を開く
1. `addRating` 関数を探す
1. 関数全体を以下のコードに置き換えます

[FriendlyEats.Data.js](https://github.com/firebase/friendlyeats-web/blob/master/scripts/FriendlyEats.Data.js#L48-L52.js)

```
FriendlyEats.prototype.addRating = function(restaurantID, rating) {
  var collection = firebase.firestore().collection('restaurants');
  var document = collection.doc(restaurantID);
  var newRatingDocument = document.collection('ratings').doc();

  return firebase.firestore().runTransaction(function(transaction) {
    return transaction.get(document).then(function(doc) {
      var data = doc.data();

      var newAverage =
          (data.numRatings * data.avgRating + rating.rating) /
          (data.numRatings + 1);

      transaction.update(document, {
        numRatings: data.numRatings + 1,
        avgRating: newAverage
      });
      return transaction.set(newRatingDocument, rating);
    });
  });
};
```

上記のブロックでは、`restaurants`ドキュメントの`averageRating`と`ratingCount`の数値を更新するトランザクションを呼び出します。同時に、`ratings`サブコレクションに新しい`rating`を追加します。

>注：評価の追加にトランザクションを使うことは、このチュートリアルのケースでは良い使用例です。ただし、実運用アプリでは、ユーザーによる不正操作を避けるために、平均評価の算出は信頼できるサーバーで行う必要があります。これを行う良い方法は、クライアントから直接評価ドキュメントを作成し、[Cloud Functions](https://firebase.google.com/docs/functions/)を利用して新しいレストランの平均評価を更新することです。

>警告：サーバーでトランザクションが失敗すると、コールバックも繰り返し再実行されます。アプリの状態を変更するロジックをトランザクションコールバック内に配置しないでください。

## データを守る

このチュートリアルの最初に、アプリのセキュリティルールをテストモードに設定し、自由に読み書きできるようにしました。
実際のアプリケーションでは、望ましくないデータの読み込みや変更を防ぐために、よりきめ細かいルールを設定する必要があります。

1. Firebase console を開き、開発 ＞ Database を選択します
1. Cloud Firestore ＞ ルール タブをクリックします
1. `rules_version = '2';` より下のコードを以下のルールに置き換えて「公開」をクリックします

[firestore.rules](https://github.com/firebase/friendlyeats-web/blob/master/firestore.rules)

```
service cloud.firestore {
  match /databases/{database}/documents {

        // Restaurants:
        //   - Authenticated user can read
        //   - Authenticated user can create/update (for demo)
        //   - Validate updates
        //   - Deletes are not allowed
    match /restaurants/{restaurantId} {
      allow read, create: if request.auth != null;
      allow update: if request.auth != null
                    && request.resource.data.name == resource.data.name
      allow delete: if false;
      
      // Ratings:
      //   - Authenticated user can read
      //   - Authenticated user can create if userId matches
      //   - Deletes and updates are not allowed
      match /ratings/{ratingId} {
        allow read: if request.auth != null;
        allow create: if request.auth != null
                      && request.resource.data.userId == request.auth.uid;
        allow update, delete: if false;
        
        }
    }
  }
}
```

これらのルールはアクセスを制限して、クライアントが安全な変更のみ行えることを保証します。例えば：

- レストランのドキュメントの更新では、評価のみが変更でき、名前やその他の不変なデータは変更できません
- ユーザーIDがサインインしているユーザーと一致する場合にのみ評価を作成できます。これにより、なりすましが防止できます

FirebaseのConsoleを使うかわりに、Firebase CLIを使用してルールをFirebaseプロジェクトに展開できます。作業ディレクトリの[`firestore.rules`](https://github.com/firebase/friendlyeats-web/blob/master/firestore.rules)ファイルには、上記のルールが既に含まれています。これらのルールをローカル環境からFirebaseにデプロイするには、次のコマンドを実行します。

```
firebase deploy --only firestore:rules
```

> 重要：セキュリティルールの詳細については、[セキュリティルールのドキュメント](https://firebase.google.com/docs/firestore/security/get-started)をご覧ください。

## まとめ
このチュートリアルでは、Cloud Firestoreで基本的な、そして高度な読み取りと書き込みを行う方法と、セキュリティルールでデータアクセスを保護する方法を学びました。
完全なソリューションは [quickstarts-js](https://github.com/firebase/quickstart-js/tree/master/firestore) リポジトリで見つけることができます。

Cloud Firestore の詳細については、次のリソースをご覧ください:

- [Cloud Firestore 入門](https://firebase.google.com/docs/firestore/)
- [データ構造の選択](https://firebase.google.com/docs/firestore/manage-data/structure-data)
- [Cloud Firestore のウェブサンプル](https://firebase.google.com/docs/firestore/client/samples-web)
