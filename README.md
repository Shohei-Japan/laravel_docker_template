# laravel_docker_template の使い方

## できること
- PHP を Docker で立ち上げる
  - PHP を動作させるために nginx を準備する
- MySQL を Docker で立ち上げる
- Laravel をインストールする

## 準備

### docker container の確認
過去にこのテンプレートを使っている場合、コンテナ名が重複し、docker-compose が失敗する。また、ポート番号についても適宜修正が必要


#### ポート番号の修正
- `docker-compose.yml` の nginx の ports
- `docker-compose.yml` の db の ports


### docker-compose 起動
```bash
# docker 起動
$ docker-compose up -d

# phpコンテナに入ります
$ docker-compose exec php bash

# Laravelプロジェクト作成
# 完了すれば localhost:8000 で Laravel が開いていることが確認できる
$ laravel new

# MySQL に接続する
$ docker-compose exec mysql bash

# データベースを確認する
$ mysql -u root -p

# MySQL を出る
$ exit

# docker 停止
$ docker-compose stop
```

### Laravel のインストール
トップディレクトリに `server` ディレクトリを追加する。
これは docker-compose.yml の `php`, `nginx` の volumes で記されている通り、開発側のディレクトリとして指定されている名称となる。

## ファイルごとの解説

### ディレクトリ構造
```bash
laravel_docker_template/
  ├ docker/
  │  ├ db/
  │  │  ├ data # 自動生成
  │  │  ├ my.cnf # 自動生成
  │  │  └ sql # 自動生成
  │  ├ nginx/
  │  │  └ default.conf # nginx の web サーバー情報を記述
  │  └ php/
  │     ├ Dockerfile # php のバージョン情報、 compose のインストールなどを記述
  │     └ php.ini # php の初期設定を記述
  ├ docker-compose.yml # Docker の情報を記述。docker/ 内のデータを参照
  └ .env.example
```

#### docker ディレクトリ
`docker/` には Docker 立ち上げに必要な情報を格納する。
- db
  - ディレクトリ内部のデータは docker 起動時に自動的に生成される
- nginx
  - default.conf に web サーバーの情報を記述
  - listen の項目
    - `listen` のあとにホストが接続するポート番号を記述。
    - ここでは 80 としており、 docker-compose.yml の nginx の ports と同じにする必要がある
  - location の項目
    - root
      - この laravel_docker_template 内で laravel を構成する場合、 `var/www/public` を指定する。
- php
  - Dockerfile
    - FROM: 使用する Docker イメージを指定。 PHP のバージョン情報を記述する
    - COPY: PHP の初期設定ファイル名を指定

#### docker-compose.yml
立ち上げる Docker の設定情報。
インデントの付け方によって動作が変わるので、インデントは厳密に記述する。

- version
- services
  - 立ち上げたいコンテナを置く
    - 現状は、 `php` , `nginx` , `db` のコンテナが立ち上がる
- services - php
  - container_name: コンテナ名称
  - build: 立ち上げるイメージの設定を参照
    - ここでは `./docker/php` 内の Dockerfile を見に行く
  - volumes
    - `./server:/var/www`
      - 開発ディレクトリと公開ディレクトリとをマウントする。
      - ここでは `./server` が開発用のディレクトリ、つまり Laravel のソースコードが入る
      - `/var/www` は外部公開用のディレクトリとなる。多分ほぼ固定のはず。
- services - nginx
  - image: ここで　Docker image を指定する。バージョン情報なども。
  - container_name: コンテナ名称
  - ports:
    - `8000:80` ゲストポートとホストポートを指定する
    - `8000` がゲストポートつまり開発側、 `80` がホストポートつまり Docker 側。
    - つまり `localhost:8000` にアクセスすると Docker 側の `:80` ポートにつながる
  - volumes:
    - php と同じく開発ディレクトリと公開ディレクトリをマウントしている。
    - `./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf`
      - nginx の設定ファイルもマウントする
- services - db
  - image: Docker image の指定
  - container_name: コンテナ名称
  - environment: MySQL の情報
    - 主にログインの user 名、パスワード、タイムゾーン
  - command: わからない
  - volumes:
    - 開発ディレクトリと公開ディレクトリとのマウント
    - その他の設定ファイルも同様にマウント
  - ports: ここでもポートの設定

## docker-compose 操作
docker-compose.yml 上で db の environment 周りを変更した場合、再度 Docker のイメージを更新する必要がある
```bash
# docker のコンテナをストップする
$ docker-compose down

# docker のイメージを削除する
$ docker-compose down --rmi all
```