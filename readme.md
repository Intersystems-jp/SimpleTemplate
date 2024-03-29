# コンテナ版IRISのシンプルな開発環境テンプレート
このテンプレートでは、コンテナ開始時に任意名のネームスペース、データベースを作成したあと、/src 以下に配置してるソースコードを作成したネームスペースにインポートしています。

設定については、適宜変更してご活用いただけます。

## Gitに含まれるファイルについて

|種類|ファイル|概要|詳細|
|:--|:--|:--|:--|
|接続設定|[settings.json](/.vscode/settings.json)|VSCodeからIRISへ接続するときの設定用ファイル。|[VSCodeを使ってみよう！](https://jp.community.intersystems.com/node/482976/japanese)|
|Dockerfileサンプル|[Dockerfile](/Dockerfile)|コンテナビルド時に実行したい処理を記述しています。|[Dockerfileで実行している内容](#dokerfileで実行している内容)|
|スクリプト|[iris.script](./iris.script)|コンテナビルド時に実行したい処理を記載したファイル（IRISログイン後に実行したいコマンドやメソッドを記述しています）|[スクリプトで実行している内容](#irisscriptで実行している内容)
|インストーラー|[Installer.cls](./Installer.cls)|コンテナビルド時に初期設定用定義が記載されているインストーラークラス（*1）|[インストーラーで実行している内容](#installerclsで実行している内容)|
|ソースコードサンプル|[Person.cls](/src/Test/Person.cls)|コンテナビルド時にIRISにインポートするサンプルクラス定義です。|[クラス定義の内容](#testpersoncls)|
|docker-composeサンプル|[docker-compose.yml](/docker-compose.yml)|<span style="color:green;">**コンテナ内ポート1972**を**ホスト上ポート1972**</span>に、<span style="color:orange;">**コンテナ内ポート52773**を**ホスト上ポート52773**</span>に割り当てるように記載しています。また、<span style="color:darkblue;">**カレントディレクトリ**を**コンテナ内ディレクトリ /ISC**</span> にマウントするように記載しています。


> （*1）インストーラーについて詳しくはドキュメント「[インストール・マニフェストの作成および使用](https://docs.intersystems.com/irislatestj/csp/docbook/DocBook.UI.Page.cls?KEY=GCI_manifest)」をご参照ください。

## コンテナ開始後のイメージ図

以下の図は、Docker for windows でコンテナを開始した時のイメージ図です。
（c:\docker　にいる状態で git clone した時の図です）

![環境イメージ図](/templateimage.png)


## コンテナビルド前の確認

サンプルの[Dockerfile](/Dockerfile)では、最新版CDリリースのInterSystems IRIS コミュニティエディションのイメージを使用するように記述しています（[Dockerfile](/Dockerfile)3行目）。

InterSystems IRIS for Health コミュニティエディションの最新版CDリリースを利用する場合は、[Dockerfile](/Dockerfile)3行目をコメントアウトしてください。

また、その他のイメージをご利用いただく場合は、[コンテナレジストリ](https://containers.intersystems.com/contents)にあるイメージをpullし、[Dockerfile](/Dockerfile)で指定するイメージ名を変更してからビルドを行ってください。


## コンテナ開始までの手順
詳細は、[docker-compose.yml](./docker-compose.yml) をご参照ください。

Git展開後、**./ は コンテナ内 /ISC ディレクトリをマウントしています。**
また、IRISの管理ポータルの起動に使用するWebサーバポートは 52773 が割り当てられています。
既に使用中ポートの場合は、[docker-compose.yml](./docker-compose.yml) の **15行目** を修正してご利用ください。

**≪62773に割り当てる例≫　- "62773:52773"**

```
git clone このGitのURL
```
cloneしたディレクトリに移動後、以下実行します。（コンテナを作成します）

```
$ docker-compose build
```
ビルド後、コンテナを開始します。
```
$ docker-compose up -d
```
コンテナを停止する方法は以下の通りです。
```
$ docker-compose stop
```
コンテナを破棄する方法は以下の通りです（コンテナを消去します）。
```
$ docker-compose down
```



## [Dokerfile](./Dockerfile)で実行している内容
コンテナビルド時の処理が記載されています。

コンテナビルド時に /opt/try　を作成しビルド時の作業で使用したいファイルをこのディレクトリ以下にコピーしています。

また、TRYネームスペースが参照する、TRYデータベースの物理パスのルートとして利用しています。

ディレクトリは変更可能です。変更された場合は、[Installer.cls](./Installer.cls)の **17行目** と、[iris.script](./iris.script)の **3行目**、**12行目** のディレクトリ指定も変更してください。

[Dokerfile](./Dockerfile)の **19行目** は、IRISへログインしています（iris session IRIS）。
ログインと同時に、[iris.script](./iris.script) に記載されたコマンドを入力しています。


## [iris.script](./iris.script)で実行している内容
ObjectScriptのコマンドが記載されているファイルです。

[Dokerfile](./Dockerfile)の中で /opt/try　以下にコピーされたファイルを利用して初期設定を行っています。

### (1)　do $SYSTEM.OBJ.Load("/opt/try/Installer.cls", "ck")

[Installer.cls](./Installer.cls)をIRISログイン時のデフォルトネームスペース＝USERにインポートしています。

### (2)　set sc = ##class(App.Installer).setup()

(1)でインポートしたインストーラーを実行しています。この実行でTRYネームスペース／TRYデータベースが作成されます。


### (3)　ソースコードのインポート

TRYネームスペースに移動し、

    set $namespace="TRY"　

以下の実行で /src 以下にあるファイルをインポートしています（他のバージョン、InterSystems製品からエクスポートしてきたXMLファイルを配置してもインポートされます）。

    do $System.OBJ.LoadDir("/opt/try/src","ck",,1)



### (4)　システム設定の変更
事前定義ユーザ（_systemやSuperUserなど）の初期パスワードの期限を無効に設定しています。
通常、コンテナ版IRISの初回アクセス時に、パスワードを任意設定できるようにパスワード変更画面が開きます。

このテンプレートでは、デフォルトパスワードの **"SYS"** で起動できるように設定しています。

IRIS初回アクセス時に初期パスワードを変更したい場合は、以下の実行をコメント化（スラッシュ2つ //）してください。
 
    //Do ##class(Security.Users).UnExpireUserPasswords("*")

日本語ロケールへの変更を行っています。コンテナ版IRISは英語Ubuntuで起動するため、デフォルトでは英語ロケールで立ち上がります。

日本語が含まれるファイル入出力などを試す場合は、日本語のロケールに変更いただく必要があります。

    Do ##class(Config.NLS.Locales).Install("jpuw")



## [Installer.cls](./Installer.cls)で実行している内容

[Dokerfile](./Dockerfile)の **7行目** で作成したディレクトリ以下にTRYデータベースを作成し、TRYネームスペースから参照するように定義しています。

また、TRYネームスペースのデフォルトウェブアプリケーションパス（/csp/try）も設定しています。

ネームスペース名、データベース名を任意に変更する場合は、[Installer.cls](./Installer.cls)の **8行目** と **10行目** を変更してください。
（ウェブアプリケーションパスは小文字で作成する必要があります。[Installer.cls](./Installer.cls)の **10行目** に設定する文字列は小文字で設定してください。）

慣習として、ネームスペース名、データベース名、ネームスペースのデフォルトウェブアプリケーションパスは、名称を統一（例：TRY）する事が多いため、例では統一しています。

インストーラーについて詳細は、ドキュメントの [インストール・マニフェストの作成および使用](https://docs.intersystems.com/irislatestj/csp/docbook/DocBook.UI.Page.cls?KEY=GCI_manifest) や、開発者コミュニティの記事 [%InstallerでInterSystems Cachéにアプリケーションをデプロイする](https://jp.community.intersystems.com/node/478966/japanese) もご参照ください。


## [Test.Person.cls](/src/Test/Person.cls)
Test.Personクラス（またはテーブル）としてTRYネームスペースにインポートされます。

定義やデータを確認するために、コンテナにログインします。（コンテナからホストに戻る場合は`exit`を入力します。）
```
docker exec -it tryiris bash
```

データを自動生成する場合は、IRISのTRYネームスペースにログインしてから
```
iris session iris -U TRY
```

以下実行するか、
```
do ##class(Test.Person).CreateData(10) //10件作成
```
> ※プロンプトをIRISログイン前に戻す場合は、`halt`コマンドを実行します。

IRISのSQLシェルにログインして（TRYネームスペースに移動した状態でログイン）

```
iris sql iris -U TRY
```
ストアドプロシージャ―を実行します。（プロンプトを元に戻したいときは、`quit`を入力します。）
```
call Test.Person_CreateData(10)
```
VSCodeでのクラス定義の作成やルーチン作成方法については、開発者コミュニティの記事 [VSCodeを使ってみよう！](https://jp.community.intersystems.com/node/482976/japanese) をご参照ください。

IRISの開発環境の準備やクラス定義やオブジェクト操作方法については、開発者コミュニティの記事 [【はじめての InterSystems IRIS】セルフラーニングビデオ：基本その2：InterSystems IRIS で開発をはじめよう！](https://jp.community.intersystems.com/node/478601/japanese) や [【はじめての InterSystems IRIS】セルフラーニングビデオ：基本その3：IRIS でクラス定義を作ろう（オブジェクト操作の練習）](https://jp.community.intersystems.com/node/478606/japanese) をぜひご参照ください！