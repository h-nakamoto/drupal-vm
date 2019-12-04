## コンテナイメージ

| Container | Service name | Image | Exposed port |
| --------- | ------------ | ----- | ------------ |
| Nginx Proxy | nginx-proxy | <a href="https://hub.docker.com/r/jwilder/nginx-proxy/" target="_blank">jwilder/nginx-proxy</a> | 80 |
| Nginx | web | <a href="https://hub.docker.com/_/nginx/" target="_blank">nginx</a> | |
| MariaDB | db | <a href="https://hub.docker.com/_/mariadb/" target="_blank">mariadb</a> | 3306 |
| PHP-FPM 5.6 / 7.0 / 7.1 | php | <a href="https://hub.docker.com/r/blauerberg/drupal-php/" target="_blank">blauerberg/drupal-php</a> | |
| mailhog | mailhog | <a href="https://hub.docker.com/r/mailhog/mailhog/" target="_blank">mailhog/mailhog</a> | 8025 (HTTP server) |
| phpmyadmin | phpmyadmin | <a href="https://hub.docker.com/r/phpmyadmin/phpmyadmin/" target="_blank">phpmyadmin/phpmyadmin</a> | 8080 (HTTP server) |
| dozzle | dozzle | <a href="https://hub.docker.com/r/amir20/dozzle" target="_blank">amir20/dozzle</a> | 8081 (HTTP server) |

## 早見表
| 機能 | アクセス先 | 説明 |
| --------- | -------| ------------ |
| Web(Drupal) |http://localhost:5580 | ウェブ |
| Dozzle log | http://localhost:8081 | ログビューア |


コンテナの一括停止
docker stop $(docker ps -q)
イメージの一括削除
docker rm -f $(docker ps -a -q)
停止コンテナ・未利用イメージ・未利用ボリュームを一括削除する
docker system prune

停止コンテナ一括削除
docker container prune
未使用イメージ一括削除
docker image prune
未使用ボリューム一括削除
docker volume prune
未使用ネットワーク一括削除
docker network prune
コンテナの一括停止

コンテナの内部にアクセスする
docker exec -i -t コンテナ名 bash






## 動作環境

- macOS Sierra上で動作する[Docker for MAC](https://docs.docker.com/docker-for-mac/)の最新版
- Windows 10上で動作する[Docker for Windows](https://docs.docker.com/docker-for-windows/)の最新版
- Linux上で動作する[Docker engine](https://docs.docker.com/engine/installation/linux/ubuntulinux/)の最新版
- Docker for Windowsを使う場合は、 [共有ドライブ](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/) を有効にしてください。

## Getting started

### コンテナーを起動する

初めにレポジトリをクローンします。
```bash
$ git clone https://github.com/blauerberg/dropfabrik.git
$ cd dropfabrik
```

次にDrupalのソースコードをマウントするためのディレクトリを作成します。
デフォルトの設定では、ホストマシンの `volumes/drupal` をコンテナーのデータボリュームとしてマウントします。

```bash
$ mkdir -p volumes/drupal
```

Drupalのソースコードをダウンロードして展開します。
```bash
# Note: replace "X.Y.Z" in below to  Drupal's version you'd like to use.
$ curl https://ftp.drupal.org/files/projects/drupal-X.Y.Z.tar.gz | tar zx --strip=1 -C volumes/drupal
```

macOSを使っている場合、[パフォーマンスの問題](https://github.com/docker/for-mac/issues/77) を回避するために [docker-sync](https://github.com/EugenMayer/docker-sync/) を利用することを強く推奨します。
[Use docker-sync](#docker-sync-を使う) を参照してください。

コンテナーを生成して起動します。
```bash
$ docker-compose up -d
```

ホストマシンがLinuxの場合は、ディレクトリの権限を修正する必要があります。
```bash
$ docker-compose exec php chown -R www-data:www-data /var/www/html/sites/default
```

Drupalにアクセスします。
```bash
$ open http://localhost # もしくはブラウザで http://localhost へアクセス
```

### Drupalのインストール

データベースの認証情報は `docker-compose.override.yml` で設定されています。
デフォルト値は以下になります。

- Database Name: `drupal`
- Username: `drupal`
- Password: `drupal`

詳細については https://hub.docker.com/_/mariadb/ の "Environment Variables" を参照してください。

Dropfabrikではnginx、mariadb、php-fpmは全て別々のコンテナーで動作します。
そのため、インストール時にデータベースサーバーのホスト名に `localhost` ではなく `db` を指定する必要があります。

ブラウザからウィザードでインストールする代わりに、以下のようにDrushを使ってインストールすることもできます。

```bash
$ docker-compose exec php drush -y --root="/var/www/html" site-install standard --site-name="Drupal on Docker" --account-name="drupal" --account-pass="drupal" --db-url="mysql://drupal:drupal@db/drupal" --locale=ja
# Drupal 8の場合は以下も実行
$ docker-compose exec php drush -y config-set system.theme admin bartik
```
## コンテナを停止する

```
$ docker-compose stop
```

## Other tips

### うまく動きません！

Dropfabrikにはコンテナ内のログをブラウザ上で表示するために
[Dozzle](https://hub.docker.com/r/amir20/dozzle)が組み込まれています。
http://localhost:8081 にアクセスしてコンテナ内で何が起こっているのが確認してみましょう。
周りにサポートしてくれるエンジニアがいる場合、Dozzleのログをそのまま見せるのは良い方法です。

### webサーバーのホスト名

http://localhost:5580

### 他の名前でアクセスしたい場合
1.`docker-compose.override.yml` の`VIRTUAL_HOST` の値を変更する
2.hosts ファイルを編集し、`127.0.0.1` で `VIRTUAL_HOST` で指定したホスト名を解決する

### PHPのバージョンを変更する
`docker-compose.yml` を編集し、使いたいPHPのバージョンに対応したイメージを使うように変更してください。

### drupal-composer/drupal-project で管理されたコードを利用する

Drupal.orgで配布されているアーカイブをそのまま利用する場合は、ルートディレクトリがそのままDrupalのソースコードの最上位ディレクトリになりますが、
[drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-project) でDrupalのプロジェクトを管理している場合は、 `web` ディレクトリ以下がDrupalのソースコードとなります。

そのため、drupal-composer/drupal-projectで管理されているコードを利用する場合はwebサーバーのドキュメントルートを変更する必要があります。
`docker-compose.override.yml` を編集し、`web` サービスのボリュームで `nginx/default.conf` の代わりに `nginx/default_for_drupal_composer.conf` をマウントするように変更してください。

### 秘密鍵を含んだphpイメージをビルドする

drushなどで秘密鍵を使うケースを考慮し、 `docker-compose.override.yml` に `php` イメージにホストOS側の秘密鍵をマウントするサンプルを用意しています。
ホストOSがlinuxとMacOSの場合はこの方法で問題ありませんが、ホストOSがWindowsの場合、[ホスト側のファイルシステムの問題](https://github.com/docker/for-win/issues/167) で秘密鍵のパーミッションが正しく設定されません。

この問題のワークアラウンドとして、 `php/php-fpm/id_rsa` に秘密鍵をコピーし `docker-compose build` すると秘密鍵入りのカスタムイメージをビルドすることができます。

### コンテナの内部にアクセスする

sshではなく `docker-compose exec` を使いましょう。

```bash
$ docker-compose exec {Service name} /bin/bash
# ex. docker-compose exec php /bin/bash
```

### Drushを使う

Drushは`php`コンテナーにインストールされています。

```bash
$ docker-compose exec php drush st
```

### 既存のサイトのデータベースをリストアする

gzipで圧縮されたSQLのダンプファイルを `initdb.sql.gz` という名前で配置し、`docker-compose.override.yml` の以下の行のコメントアウトを解除してください。
コンテナの生成時に一度だけこのファイルがロードされ、データベースが復元されます。

```
- ./initdb.sql.gz:/docker-entrypoint-initdb.d/initdb.sql.gz
```

### データベースに接続する

Drush経由で接続する:
```bash
$ docker-compose exec php drush sqlc
```

データベースコンテナは 127.0.0.1上でport 3306をlistenします。そのため、[MysqlWorkbench](https://www.mysql.com/products/workbench/) や [Sequel Pro](https://www.sequelpro.com/) のようなホストOS上で動作するGUIアプリケーションからコンテナー内のデータベースに接続することができます。

また、 `http://localhost:8080` にアクセスすると
 [phpmyadmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) 
 を利用してデータベースにアクセスすることもできます。

### docker-sync を使う

macOSを使っている場合、[パフォーマンスの問題](https://github.com/docker/for-mac/issues/77) を回避するために [docker-sync](https://github.com/EugenMayer/docker-sync/) を利用することを強く推奨します。インストール方法は [docker-sync.io](http://docker-sync.io/) を参照してください。

docker-syncを使う場合は `docker-compose.override-for-docker-sync.yml` を `docker-compose.override.yml` としてコピーしてください。
また、`docker-compose` の代わりに `docker-sync-stack` でイメージを起動する必要があります。

```bash
$ docker-sync-stack start
```

詳細は https://github.com/EugenMayer/docker-sync/wiki を参照してください。

## Supporting Organizations
- https://annai.co.jp
