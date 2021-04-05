# CakePHP3.xローカル開発環境

Dockerとcomposerを使ってCakePHP3.xの環境を構築します。

## 事前準備

### 1. Docker for macのインストール

公式サイトからDockerのアカウントを作ってログインし、DockerHubからダウンロードしてインストール  
https://hub.docker.com/editions/community/docker-ce-desktop-mac

### 2. リポジトリのクローン

ファイルを展開したいディレクトリで  
`$ git clone https://github.com/asutoko-mashi/cakephp3-env.git`

## 早速動かしてみよう

### 1. イメージ作成、コンテナ立ち上げ

`cakephp3-env`ディレクトリ（任意の名前にも変更可能）へ移動し
1. `$ docker-compose build`　でイメージを作成し

2. `$ docker-compose up -d`　でバックグラウンドでコンテナを立ち上げ

3. `$ docker ps`　で起動したコンテナを確認します（phpfpmのコンテナIDまたはコンテナ名を確認）

### 2. CakePHPのインストール、プロジェクトの作成
1. `$ docker exec -it コンテナID(またはコンテナ名) /bin/sh`　でphpfpmコンテナに入り、ターミナルを立ち上げる

2. `# curl -s https://getcomposer.org/installer | php` でcomposerのinstallerを取得

3. `# php composer.phar create-project --prefer-dist cakephp/app:^3.8 プロジェクト名`

    （途中「Set Folder Permissions ? (Default to Y) [Y,n]?」はYを選択）

4. `# exit`　でコンテナを抜ける

    （3.でバージョンを指定しない場合、最新の4.xがインストールされます）

##### 細かくバージョンを指定したい場合

例えば3.2をインストールしたい場合は以下を実行します。

`# php composer.phar create-project --prefer-dist cakephp/app:3.2.* プロジェクト名`


#### "Action required!"と言われたら

cakephp/plugin-installerのバージョンアップによりpostAutoloadDump()の実行が必要なくなったそうです。

作成したプロジェクトフォルダの中にある`composer.json`の該当行の削除が求められています。

気にならないのであれば、対処不要です・・・

```text
    "scripts": {
        "post-install-cmd": "App\\Console\\Installer::postInstall",
        "post-create-project-cmd": "App\\Console\\Installer::postInstall",
        "post-autoload-dump": "Cake\\Composer\\Installer\\PluginInstaller::postAutoloadDump",
         ↑この行を削除します
```


## ファイルの書き換え

### 1. 接続先データソース

`data/htdocs/プロジェクト名/config/app_local.php`　の40行目あたりの `host` を `mysql` に変更
  ```php
  'Datasources' => [
      'default' => [
          'host' => 'mysql',
  …以下略
  ```
  ※あくまでもローカル環境の設定です。
  本番の設定は`data/htdocs/プロジェクト名/config/app.php`の265行目あたりで行います。

  古いバージョンを指定しインストールすると、app_local.phpがない場合があります。
  その場合は、app.phpの該当箇所を編集してください。


### 2. プロジェクト名

`docker-compose.yml` の下部にある`PRJ`を上で作成したプロジェクト名に変更
  ```text
    host:
      build: ./data/htdocs
      environment:
        TZ: "Asia/Tokyo"
        PRJ: "プロジェクト名"
      volumes:
  ```


## 起動

1. `$ docker-compose up -d`

2. `localhost:8765`にアクセス cakephp3 のページが表示されるか確認

3. `localhost:8080` にアクセス phpmyadminのトップ画面が表示されるか確認


## フォルダ構成

最終的にこのようになります

```text
.
└── data
    ├── htdocs
    │   └── プロジェクト名のフォルダ
    │       ├── bin
    │       ├── config
    │       ├── logs
    │       ├── plugins
    │       ├── src
    │       ├── tests
    │       ├── tmp
    │       ├── vendor
    │       └── webroot
    ├── mysql
    │   └── db
    │       ├── my_app
    │       ├── mysql
    │       ├── performance_schema
    │       └── sys
    ├── nginx
    │   └── conf
    │       └── conf.d
    ├── phpfpm
    └── phpmyadmin
        └── sessions
```

## 開発時の手順

1. 開発環境の起動  
  `$ docker-compose up -d`

2. 開発終了  
  コンテナ停止 `$ docker-compose stop`  
  コンテナ削除 `$ docker-compose down`

3. サーバー設定やDockerfileを書き換えたときは再ビルドが必要です  
  `$ docker-compose build`


## 参考URL

- docker-cakephp3-template
   https://github.com/km42428/docker-cakephp3-template

- Cake2.xローカル開発環境
   https://github.com/oaxisstudio/cake-env