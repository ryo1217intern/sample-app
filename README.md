## docker-compose.ymlを書く.
以下が今回作成したファイルです.
```yml
version: '3'
services:
  react:
    build:
      context: .
      dockerfile: ./docker/react/Dockerfile
    container_name: react_container
    tty: true
    volumes:
      - ./react-app/react-app:/app/react
    command: sh -c "cd /app/react && yarn install && yarn start"
    ports:
      - 3000:3000
  go:
    build:
      context: .
      dockerfile: ./docker/golang/Dockerfile
    container_name: go_container
    ports:
      - 8000:8000
    tty: true
    volumes:
      - ./go-app:/app/go
  mysql:
    build:
      context: .
      dockerfile: ./docker/mysql/Dockerfile
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: react-go-app
      TZ: 'Asia/Tokyo'
    volumes:
      - ./docker/mysql/initdb.d:/dockr-entrypoint-initdb.d
      - ./docker/mysql/conf.d:/etc/mysql/conf.d
      - ./docker/mysql/mysql_data:/var/lib/mysql
    ports:
      - 3306:3306
    links:
      - go
```
順を追って説明していきます.
```yml
version: "3"
services:
  hogehoge:
  fugafuga:
  .
  .
  .
```
このようにまず最初に`version`と`services`を書きます.

versionはこのdocker-composeのバージョンです.

またservicesとは扱う複数コンテナをリスト化したものです.

それではそれぞれのサービスの設定をみていきます.

## Reactコンテナの詳細

### 1. Buildの設定

- **context:** 
  - 現在のディレクトリ (`.`) をDockerのビルドコンテキストとして指定。

- **dockerfile:** 
  - `./docker/react/Dockerfile` というパスにDockerfileが配置されている。このDockerfileには、Reactアプリを動かすための指示が書かれている。

### 2. Container Name

- 名前: `react_container`
  - この名前は、Dockerでコンテナを操作や管理する際に参照される名前。

### 3. TTY

- 値: `true`
  - TTY（Teletype）オプションがtrueに設定されている。これにより、このコンテナはインタラクティブなシェルを持つことができる。

### 4. Volumes

- パス: `./react-app/react-app:/app/react`
  - ホストマシンの `./react-app/react-app` ディレクトリとコンテナの `/app/react` ディレクトリを同期する。これにより、ホスト上の変更がコンテナ内にすぐ反映される。

### 5. Command

- 実行コマンド: `sh -c "cd /app/react && yarn install && yarn start"`
  - コンテナ起動時に実行されるコマンド。コンテナ内でReactアプリのディレクトリに移動し、必要なパッケージをインストールした後、アプリを起動する。

### 6. Ports

- ポートマッピング: `3000:3000`
  - ホストマシンの3000番ポートとコンテナの3000番ポートを結びつける。これにより、ホストマシンから `localhost:3000` でReactアプリにアクセスできる。

## Goコンテナの詳細

### 1. Buildの設定

- **context:** 
  - 現在のディレクトリ (`.`) をDockerのビルドコンテキストとして指定しています。

- **dockerfile:** 
  - ビルドに使用されるDockerfileは `./docker/golang/Dockerfile` というパスに配置されています。このDockerfileには、Goアプリケーションを実行するための手順や設定が記述されています。

### 2. Container Name

- 名前: `go_container`
  - Docker内でのコンテナを識別するための名前です。

### 3. Ports

- ポートマッピング: `8000:8000`
  - ホストマシンの8000番ポートとコンテナ内の8000番ポートが結びつけられています。これにより、ホストマシンから `localhost:8000` でGoアプリケーションにアクセスすることができます。

### 4. TTY

- 値: `true`
  - TTYオプションがtrueに設定されているため、このコンテナはインタラクティブなシェルをサポートします。

### 5. Volumes

- パス: `./go-app:/app/go`
  - ホストマシンの `./go-app` ディレクトリとコンテナ内の `/app/go` ディレクトリが同期されます。この設定により、ホスト上でのファイルの変更が即座にコンテナに反映されるようになります。

## MySQLコンテナの詳細

### 1. Buildの設定

- **context:** 
  - ビルドのコンテキストとして現在のディレクトリ (`.`) が指定されています。

- **dockerfile:** 
  - ビルドに使用されるDockerfileは `./docker/mysql/Dockerfile` のパスにあります。このファイルにはMySQLの設定や初期化に関する手順が記述されています。

### 2. Container Name

- 名前: `mysql_container`
  - Docker内でこのコンテナを識別するための名前です。

### 3. Environment Variables

- **MYSQL_ROOT_PASSWORD:** 
  - MySQLのrootユーザーのパスワードを `root` として設定しています。
  
- **MYSQL_DATABASE:** 
  - 初期に作成されるデータベースの名前として `react-go-app` が指定されています。
  
- **TZ:**
  - コンテナのタイムゾーンを `Asia/Tokyo` として設定しています。

### 4. Volumes

- **初期化用のスクリプト:** 
  - ホストの `./docker/mysql/initdb.d` ディレクトリとコンテナの `/dockr-entrypoint-initdb.d` ディレクトリが同期されています。MySQLの初期化時にこのディレクトリ内のスクリプトが実行されるため、特定のテーブルやデータの初期化を行いたい場合は、こちらのディレクトリにスクリプトを追加することができます。

- **設定ファイル:** 
  - `./docker/mysql/conf.d` ディレクトリとコンテナの `/etc/mysql/conf.d` が同期されています。このディレクトリにはMySQLのカスタム設定が入ることが多いです。

- **データボリューム:** 
  - データの永続化のため、ホストの `./docker/mysql/mysql_data` とコンテナの `/var/lib/mysql` が同期されています。これにより、コンテナを再起動してもデータが維持されます。

### 5. Ports

- ポートマッピング: `3306:3306`
  - ホストマシンの3306番ポートとコンテナ内の3306番ポートが結びつけられています。これにより、ホストマシンから `localhost:3306` でMySQLにアクセスすることができます。

### 6. Links

- 値: `- go`
  - MySQLコンテナはGoコンテナとリンクされており、これによりGoコンテナからMySQLコンテナに簡単にアクセスすることが可能になっています。
