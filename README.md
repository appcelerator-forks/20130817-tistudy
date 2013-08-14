# Webサービスと連携したアプリ作成講座

## 本講座の位置づけについて

この講座では、実際のWebサービスと連携するアプリの制作を通じて、端末の通信機能を利用したアプリを作成する方法を学びます。

本講座では[Qiita](http://qiita.com/)というサービスのビューワを制作し、通信機能の作り方を学びます。なお、受講にあたっては、[Qiita](http://qiita.com/)のアカウントは必要ありません。

## 講座概要

- 本講座用にプロジェクトを設定する
- 端末の通信機能を使ってQiitaの情報を取得する
	- Titanium Mobileの通信機能についての解説
		- Titanium Mobileの 通信機能のhttpClientについて
	- Qiitaの投稿情報を取得するWebAPIの解説
		- QiitaのWebAPIの構造について
		- 実装する前にWebブラウザを使ってQiitaのWebAPIにアクセスする		
	- Qiitaの投稿情報を取得する処理を実装する
		- Titanium Mobileの 通信機能を使って実装する
- Qiitaから取得した結果を画面に配置する
	- スマートフォン向けアプリでよくある一覧機能表示する機能についての解説
		- Titanium MobileのTableViewについて
		- 活用事例の紹介
	- サンプルデータを使ってTableViewの使い方について学ぶ
	- 取得したQiitaの投稿情報をTableViewを使って画面表示する
		- 取得した投稿情報のタイトルだけ表示する
		- 取得した投稿情報のタイトルとアイコン画像を表示する
- 参考資料

## 本講座用にプロジェクトを設定する


Titanium Studioを起動して、QiitaViewer用にプロジェクトを作成するためにFile → New → Mobile Project と進みます

![選択メニュー](image/qiitaviewer-project-001.jpg)

Projectのテンプレートを選択する画面が標示されたら、「Default Project」を選択して次に進みます

![Projectのテンプレート選択画面](image/qiitaviewer-project-002.jpg)

Projectの詳細情報を入力する画面が標示されるので以下のように入力/選択して、finishボタンをクリックします

- プロジェクト名: **QiitaViewer**
- AppID: **info.ti.qiitaviewer**
- ターゲットOS:iPhone、Androidのみチェック。iPad、MobileWebはオフにする

![Projectの詳細情報入力画面](image/qiitaviewer-project-003.jpg)

しばらくすると、以下の様な画面が標示されて、Projectの初期設定が完了します

![Project設定完了後の画面](image/qiitaviewer-project-004.jpg)

念のためこの状態でbuildが出来るかどうか確認するため、Titanium StudioのApp Explorerのrunのアイコンを選択します

![App Explorerの画面キャプチャ](image/qiitaviewer-project-005.jpg)

<table>
<th>iPhone起動時の画面キャプチャ</th>
<th>Android起動時の画面キャプチャ</th>
<tr>
<td>
<a href="image/qiitaviewer-project-006.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-project-006.jpg" alt="iPhone Simulator"></a>
</td>
<td>
<a href="image/qiitaviewer-project-007.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-project-007.jpg" alt="iPhone Simulator"></a>
</td>
</tr>
</table>


## 端末の通信機能を使ってQiitaの情報を取得する

QiitaのようなWebAPIと連携するアプリを開発する場合に、Titanium Mobile通信機能を使うことで比較的簡単に実現することができます。
この章では

- Titanium Mobileの通信機能についての解説
- Qiitaの投稿情報を取得するWebAPIの解説
- Qiitaの投稿情報を取得する処理を実装する

という３つのステップに分けて順番に解説していきます

### Titanium Mobileの通信機能についての解説

標準機能のhttpCLientを活用することで簡単に実現できますが、まずはhttpCLientの使い方について解説をします。

QiitaのようなWebAPIを提供するWebサービスは多数あるかと思います。そういったWebAPIにアクセスしてXML形式のファイルやJSON形式のテキストファイルを取得する際にTitanium Mobileの標準機能のhttpCLientを活用します。

なお、インターネット上のHTMLコンテンツを取得すること自体はもちろん可能です。ただし、「すべてのimgタグの要素を抜き出す」というようなHTMLの構造を解析した上で、何か処理をしたい場合には、そもそもTitanium MobileのhttpCLientにはそのような機能がありません。

もしもそのような処理をしたい場合には

- US Yahooが提供してる YQLというサービスを使える仕組みがTitanium Mobile 標準にあるのでそれを活用する
- Titanium MobileのhttpCLientでHTMLファイルを取得した後、外部のライブラリなどを活用してHTMLの構文解析をする

という方法で実現出来ます。今回はHTMLファイルを取得する処理は取り上げませんが興味ある方は最後の参考資料の情報を参考にしてみてください

### Qiitaの投稿情報を取得するWebAPIの解説

Titanium Mobileで実装をはじめる前に、Qiitaの投稿情報を取得する時のWebAPIについて簡単に説明します

#### QiitaのWebAPIの構造について

QiitaのWebAPIは

https://qiita.com/api/v1/[利用したいサービス別のディレクトリ]

という形になってます。

[Qiitaの開発者向けのドキュメント](http://qiita.com/docs)に利用したいサービス別のディレクトリ情報などがまとまってますが、ドキュメントの読み方について簡単に説明します

特定ユーザの情報取得の項では

GET /api/v1/users/:url_name

と記載されています

例えば、私のQiita上のユーザ情報（アカウント名はh5y1m141@github)を取得したい場合には **:url_name** の所を該当ユーザに置き換えることで情報が取得出来ます。

具体的には、以下URLにアクセスすることで私のQiita上のユーザ情報を取得することができます

[https://qiita.com/api/v1/users/h5y1m141@github](https://qiita.com/api/v1/users/h5y1m141@github)

```javascript
{"id":10187,"url_name":"h5y1m141@github","profile_image_url":"https://secure.gravatar.com/avatar/4fdf95707fe9a33f3a1ba8c97315468c?d=https://a248.e.akamai.net/assets.github.com%2Fimages%2Fgravatars%2Fgravatar-user-420.png","url":"http://qiita.com/h5y1m141@github","description":"iPhone用のQiita Viewerアプリ作ってます\r\nhttps://github.com/h5y1m141/TiQiita","website_url":"http://h5y1m141.hatenablog.com/","organization":"","location":"","facebook":"","linkedin":"","twitter":null,"github":"h5y1m141","followers":7,"following_users":9,"items":1,"teams":[],"image_upload":{"limit":2097152,"used":0}}
```

#### 実装する前にWebブラウザを使ってQiitaのWebAPIにアクセスする		

実装する前にWebブラウザを使ってQiitaのWebAPIにアクセスして、どのような結果が取得できるのかを確認してみましょう。

Webブラウザを起動して以下URLにアクセスします

[https://qiita.com/api/v1/items](https://qiita.com/api/v1/items)

以下はMac版のGoogle Chromeでアクセスした時の画面イメージになります。
![ブラウザでアクセスした時のキャプチャ](image/qiitaviewer-webapi-001.jpg)

QiitaのWebAPIの構造についておおまかに理解出来たかと思いますので、実際にQiitaの投稿情報を取得する処理を実装していきます。

### Qiitaの投稿情報を取得する処理を実装する

Qiitaの投稿情報を取得するためにTitanium Mobileの 通信機能を使って実装します

#### Titanium Mobileの 通信機能を使って実装する

1. プロジェクト作成時に自動的に生成された **app.jsの中身のソースコードを全て削除**します。
2. その後に以下を記述します

```javascript
var xhr,qiitaURL,method;
qiitaURL = "https://qiita.com/api/v1/items";
method = "GET";
xhr = Ti.Network.createHTTPClient(); // (1)
xhr.open(method,qiitaURL);  // (2)
xhr.onload = function(){
  var body;
  if (this.status === 200) { // (3)
    body = JSON.parse(this.responseText); // (4)
	Ti.API.info(body);
  } else {
    Ti.API.info("error:status code is " + this.status);
  }
}
xhr.onerror = function(e) { // (5)
  var error;
  error = JSON.parse(this.responseText);
  Ti.API.info(error.error);
}
xhr.timeout = 5000;
xhr.send();
```

1. httpClientを利用するためのオブジェクトを生成します

2. open()メソッドを使ってQiitaのWebAPIにアクセスします。最初の引数にHTTPメソッドを指定しますが、Qiitaの投稿の取得をする場合には、GETメソッドを指定する必要があります（詳しくは[Qiitaのドキュメント](http://qiita.com/docs#13)を参照してください)次の引数で投稿情報を取得するQiitaのエンドポイントとなるURLを指定します。

3. QiitaのWebAPIにアクセスして、接続成功したかどうかを判定して、その後の処理を実施します。具体的にはthis.statusの値を確認して、値が200の場合には接続成功しているため該当する処理を実施します

4. this.responseTextの値を確認することで、サーバから取得できた値をテキスト形式で取得できます。this.responseTextは見た目はJSON形式になっていますが、そのまま変数に代入すると文字列としてその後処理されてしまうため、JSON.parse()を使って、JSON化した状態で変数に格納します

5. 例えば、QiitaのWebAPIにアクセスして、150リクエスト/1時間というAPIの利用制限に引っかかってしまう場合などはエラーになり、その時にはonerrorイベントが呼び出されます。

イメージとしては以下のような対応関係になります

![httpClientのonloadとonerrorの対応関係](image/qiitaviewer-httpClient-overview-001.jpg)

動作確認するために、buildした結果は以下のとおりです

iPhone起動時の画面キャプチャ

![選択メニュー](image/qiitaviewer-httpClient-iphone-001.jpg)

Android起動時の画面キャプチャ

![選択メニュー](image/qiitaviewer-httpClient-android-001.jpg)


シミュレーターの画面には何も標示されずコンソール上に複数の文字が標示されるかと思います。

この段階ではTitanium Mobileの通信機能を使ってデータ取得することを目的に実装しているため、スマートフォンの画面表示するところまでは実装していません。

次の章で、Qiitaから取得した結果を画面に配置する処理について解説していきます

## Qiitaから取得した結果を画面に配置する

Titanium Mobileの通信機能を使ってQiitaの投稿情報を取得できるようになったかと思います。

取得した結果をスマートフォン上で見やすく配置する方法について順番に解説していきます

### スマートフォン向けアプリでよくある一覧機能表示する機能についての解説

一覧機能表示する機能としては、TableView というものがあるのでまずはその解説をします

#### Titanium MobileのTableViewについて

iOS向けの特にニュースリーダー系のアプリケーションに採用されているユーザインタフェースとして、垂直方向にスクロールしながら情報を表示するようなものがあるかと思いますが、そういったユーザインタフェースを実現するための要素としてTableViewというものがあります。

#### 活用事例の紹介

ニュースリーダー系のアプリケーションと言っても、イメージがつかない場合もあるかと思いますので、TableViewの活用事例について紹介します。
<table>
<th>TiQiitaの画面キャプチャ</th>
<th>CraftBeerFanの画面キャプチャ</th>
<tr>
<td>
<a href="image/TiQiita-01.png" target="_blank"><img src="image/TiQiita-01.png" alt="TiQiitaの画面キャプチャその１"></a>
</td>
<td>
<a href="image/CraftBeerFan-01.png" target="_blank"><img src="image/CraftBeerFan-01.png" alt="CraftBeerFanキャプチャその１"></a>
</td>
</tr>
<tr>
<td>
<a href="image/TiQiita-02.png" target="_blank"><img src="image/TiQiita-02.png" alt="TiQiitaの画面キャプチャその2"></a>
</td>
<td>
<a href="image/CraftBeerFan-02.png" target="_blank"><img src="image/CraftBeerFan-02.png" alt="CraftBeerFanキャプチャその2"></a>
</td>
</tr>
</table>

### サンプルデータを使ってTableViewの使い方について学ぶ

Titanium Mobileの標準APIであるhttpCLientを通じてQiitaの投稿情報を取得した後に、TableViewを使って画面に表示する機能を最初から作るとなると、理解が進まない可能性があるかと思います。

そこで、まずはあらかじめ準備してあるサンプルデータでのTableViewの使い方について説明します

これから実装する処理イメージとしては以下のようになります

![ローカルのJSONを活用したTableView](image/localJSONSample.png)

サンプルデータは以下に準備してあるので、Webブラウザでダウンロードして、必ず現在開発中のTitanium MobileのプロジェクトのResourcesフォルダ直下に保存してください

[https://raw.github.com/h5y1m141/20130817-tistudy/master/sample.json](https://raw.github.com/h5y1m141/20130817-tistudy/master/sample.json)

保存が完了したら、以下要領で作業をします

1. 先ほどの **app.jsの中身のソースコードを全て削除**します。
2. その後に以下を記述します

```javascript
var sample, file, body, mainTable, win, i ,len ,row ,rows,textLabel;
// ダウンロードしたJSONファイルを読み込む処理
sample = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "sample.json");
file = sample.read().toString();
body = JSON.parse(file);

mainTable = Ti.UI.createTableView({
  width: 320,
  height:480,
  backgroundColor:"#fff",
  left: 0,
  top: 0
});
win = Ti.UI.createWindow({
  title:'QiitaViewer'
});
rows = []
for (i = 0, len = body.length; i < len; i++) { // (1)
  row = Ti.UI.createTableViewRow({	// (2)
    width: 'auto',
    height:40,
    borderWidth: 0,
	className:'entry',
    color:"#222"
  });
  textLabel = Ti.UI.createLabel({	// (3)
    width:'auto',
    height:30,
    top:5,
    left:5,
    color:'#222',
    font:{
      fontSize:16,
      fontWeight:'bold'
    },
    text:body[i].title
  });
  row.add(textLabel);		// (4)
  rows.push(row);			// (5)
}
mainTable.setData(rows);    // (6)
win.add(mainTable);
win.open();
```

1. body.lengthの値を確認することで投稿件数が確認できるので、その件数分ループして、投稿情報を１つづつ取り出していきます
2. 投稿情報の要素を格納するためにTableViewRowを生成します
3. 投稿情報のタイトル部分を格納するためにLabelを生成します
4. 上記で生成したLabelをTableViewRowに配置します
5. TableViewRowを配列rowsに挿入します
6. 投稿件数分のTableViewRowが配列rowsに格納されているので、その情報をTableViewに表示するために、setDataメソッドを使います

### 取得したQiitaの投稿情報をTableViewを使って画面表示する

Titanium Mobileの標準APIであるhttpCLientを活用して、Qiitaの投稿情報を取得する処理までは実装出来ましたので、今度は取得した情報を画面に表示する部分について解説します

#### 取得した投稿情報のタイトルだけ表示する

Qiitaの開発者向けのAPIを通じて投稿情報を取得した結果をTableViewを活用して表示する方法について解説します。

図にすると以下の様な処理になります。

![概念図１](image/qiitaviewer-tableView-overview-001.jpg)

以下のソースコードをサンプルに順次解説していきます。

```javascript
var xhr,qiitaURL,method,mainTable,win;
mainTable = Ti.UI.createTableView({
  width: 320,
  height:480,
  backgroundColor:"#fff",
  left: 0,
  top: 0
});
win = Ti.UI.createWindow({
  title:'QiitaViewer'
});

qiitaURL = "https://qiita.com/api/v1/items";
method = "GET";
xhr = Ti.Network.createHTTPClient();
xhr.open(method,qiitaURL);
xhr.onload = function(){
  var body,i ,len ,row ,rows,textLabel;
  if (this.status === 200) {
    body = JSON.parse(this.responseText);
    rows = [];
    for (i = 0, len = body.length; i < len; i++) { // (1)
      Ti.API.info(body[i].title);
      row = Ti.UI.createTableViewRow({	// (2)
        width: 'auto',
        height:40,
        borderWidth: 0,
		className:'entry',
        color:"#222"
      });
      textLabel = Ti.UI.createLabel({	// (3)
        width:'auto',
        height:30,
        top:5,
        left:5,
        color:'#222',
        font:{
          fontSize:16,
          fontWeight:'bold'
        },
        text:body[i].title
      });
      row.add(textLabel);		// (4)
      rows.push(row);			// (5)
    }
    mainTable.setData(rows);    // (6)
    win.add(mainTable);
    win.open();

  } else {
    Ti.API.info("error:status code is " + this.status);
  }
};
xhr.onerror = function(e) {
  var error;
  error = JSON.parse(this.responseText);
  Ti.API.info(error.error);
};

xhr.send();
```

1. body.lengthの値を確認することでWebAPIから取得した投稿件数が確認できるので、その件数分ループして、投稿情報を１つづつ取り出していきます
2. 投稿情報の要素を格納するためにTableViewRowを生成します
3. 投稿情報のタイトル部分を格納するためにLabelを生成します
4. 上記で生成したLabelをTableViewRowに配置します
5. TableViewRowを配列rowsに挿入します
6. WebAPIから取得した投稿件数分のTableViewRowが配列rowsに格納されているので、その情報をTableViewに表示するために、setDataメソッドを使います

上記をbuildして、iPhone、AndroidのEmulatorで表示した場合以下の様になります

<table>
<th>iPhone起動時の画面キャプチャ</th>
<th>Android起動時の画面キャプチャ</th>
<tr>
<td>
<a href="image/qiitaviewer-httpClient-iphone-002.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-httpClient-iphone-002.jpg" alt="iPhone Simulator"></a>
</td>
<td>
<a href="image/qiitaviewer-httpClient-android-002.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-httpClient-android-002.jpg" alt="iPhone Simulator"></a>
</tr>
</table>


#### 取得した投稿情報のタイトルとアイコン画像を表示する

先程は、タイトルのみ表示する方法について解説しましたが、タイトルだけではなく投稿したユーザのアイコンをTableViewを活用して表示する方法について解説します

図にすると以下の様な処理になります。

![概念図２](image/qiitaviewer-tableView-overview-002.jpg)

TableViewを使って投稿情報のタイトルと投稿したユーザのアイコンを表示するソースコード全体は以下になります

```javascript
var xhr,qiitaURL,method,mainTable,win;
mainTable = Ti.UI.createTableView({
  width: 320,
  height:480,
  backgroundColor:"#fff",
  left: 0,
  top: 0
});
win = Ti.UI.createWindow({
  title:'QiitaViewer'
});

qiitaURL = "https://qiita.com/api/v1/items";
method = "GET";
xhr = Ti.Network.createHTTPClient();
xhr.open(method,qiitaURL);
xhr.onload = function(){
  var body,i ,len ,row ,rows,textLabel,iconImage,imagePath;
  if (this.status === 200) {
    body = JSON.parse(this.responseText);
    rows = [];
    for (i = 0, len = body.length; i < len; i++) {
      row = Ti.UI.createTableViewRow({
        width: 'auto',
        height:60,
        borderWidth: 0,
		className:'entry',
        color:"#222"
      });
      textLabel = Ti.UI.createLabel({
        width:250,
        height:30,
        top:5,
        left:60,
        color:'#222',
        font:{
          fontSize:16,
          fontWeight:'bold'
        },
        text:body[i].title
      });
      imagePath = body[i].user.profile_image_url;
      iconImage = Ti.UI.createImageView({
        width:40,
        height:40,
        top:5,
        left:5,
        defaultImage:"logo.png",
        image: imagePath
      });

      row.add(textLabel);
      row.add(iconImage);
      rows.push(row);
    }
    mainTable.setData(rows);
    win.add(mainTable);
    win.open();

  } else {
    Ti.API.info("error:status code is " + this.status);
  }
};
xhr.onerror = function(e) {
  var error;
  error = JSON.parse(this.responseText);
  Ti.API.info(error.error);
};
xhr.timeout = 5000;
xhr.send();
```

上記ソースコードをbuildして、iPhone、AndroidのEmulatorで表示した場合以下の様になります

<table>
<th>iPhone起動時の画面キャプチャ</th>
<th>Android起動時の画面キャプチャ</th>
<tr>
<td>
<a href="image/qiitaviewer-tableView-iphone-001.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-tableView-iphone-001.jpg" alt="iPhone Simulator"></a>
</td>
<td>
<a href="image/qiitaviewer-tableView-android-001.jpg" target="_blank"><img src="https://s3-ap-northeast-1.amazonaws.com/tiuitips/qiitaviewer-tableView-android-001.jpg" alt="iPhone Simulator"></a>
</tr>
</table>


なお、投稿したユーザのアイコンを表示する時に１つ意識しておいたほうが良い部分があります。

今回のようなWebAPIを通じて画像を取得して表示するような場合、ネットワークの回線状況によってはすぐに標示されるとは限りません。

そのため、ImageViewのdefaultImageというプロパティに、あらかじめローカルに準備しておいた画像を指定することで、

1. 最初にローカルの画像が表示
2. 画像のダウンロード完了時に、ローカルの画像とリモートから取得した画像が入れ替わる

という処理が自動的にされるので、以下のようにTitaniumのImageViewのdefaultImageのプロパティを適切に設定することをお勧めします

```javascript
// 一部抜粋
imagePath = body[i].user.profile_image_url;
iconImage = Ti.UI.createImageView({
  width:40,
  height:40,
  top:5,
  left:5,
  defaultImage:"logo.png",
  image: imagePath
});
row.add(iconImage);
```

## 参考資料
 
### YQL

インターネット上にあるコンテンツをSQLに似た独自言語で取得することが出来るUS Yahooが提供してるWebサービス。

「YQLをTitaniumで使うと色々捗る」というブログ記事でTitanium MobileでのYQL使ったサンプルが紹介されてます
[http://d.hatena.ne.jp/zebevogue/20120421/1334990568](http://d.hatena.ne.jp/zebevogue/20120421/1334990568)

### HTML parser

私は使ったことないですが、Titanium Mobile向けのHTML parserがGitHubのGistにあります。

[詳しくは https://gist.github.com/zeuxisoo/1016047](https://gist.github.com/zeuxisoo/1016047)を参照してください

