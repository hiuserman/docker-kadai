# ToDoアプリ

## 概要

GoとMySQLで作成した、ブラウザーからタスクを管理するアプリです。Docker Composeでアプリとデータベースをまとめて起動できます。

- タスクの一覧表示・追加・削除
- 完了／未完了の切り替え
- MySQLへのデータ保存

## 使用技術

- Go 1.22（net/http、html/template、database/sql）
- MySQL 8.0
- Docker / Docker Compose

## ファイル構成

```text
docker-kadai/
├── app/
│   ├── Dockerfile
│   ├── go.mod
│   └── main.go
├── db/
│   └── init/
│       └── init.sql
├── .env
├── .gitignore
├── docker-compose.yml
└── README.md
```

## 環境構築の手順

### 1. Dockerを準備する

Docker Desktopなど、Docker EngineとDocker Composeが使える環境を用意し、Dockerを起動してください。GoとMySQLはコンテナ内で動くため、PCへの個別インストールは不要です。

次のコマンドで利用できることを確認します。

```powershell
docker --version
docker compose version
```

### 2. プロジェクトのフォルダーに移動する

次はWindowsでの配置例です。保存先に合わせて変更してください。

```powershell
cd C:\Development\docker\docker-kadai
```

### 3. 環境変数を設定する

`docker-compose.yml`と同じフォルダーに`.env`を作成し、次の値を設定してください。パスワードの例は任意の値に変更します。

```dotenv
MYSQL_ROOT_PASSWORD=replace_with_root_password
DB_USER=testuser
DB_PASSWORD=replace_with_app_password
DB_NAME=todo_app_db
```

- `MYSQL_ROOT_PASSWORD`：MySQLのrootユーザーのパスワード
- `DB_USER`：アプリが接続するMySQLユーザー名（root以外）
- `DB_PASSWORD`：アプリ用ユーザーのパスワード
- `DB_NAME`：アプリが使用するデータベース名

`.env`には接続情報が含まれるため、Gitに登録しません。`.gitignore`で`/.env`を指定します。

### 4. ビルドして起動する

```powershell
docker compose up -d --build
```

初回起動では、MySQLがデータベースとアプリ用ユーザーを作成し、`db/init/init.sql`で`todos`テーブルを作成します。アプリはMySQLのヘルスチェックが成功してから起動します。

起動状態とログを確認できます。

```powershell
docker compose ps
docker compose logs app db
```

### 5. ブラウザーでアクセスする

[http://localhost:8080](http://localhost:8080)を開き、タスクの追加・完了切り替え・削除を確認してください。PCの8080番ポートが空いている必要があります。

## 停止と再起動

停止してコンテナを削除する場合：

```powershell
docker compose down
```

再起動する場合：

```powershell
docker compose up -d
```

DBのデータは名前付きボリューム`db-data`に保存されるため、通常の`down`では残ります。ソースコードを変更した場合は`docker compose up -d --build`で再ビルドしてください。

## DB初期化に関する注意

MySQLの環境変数によるDB・ユーザー作成と、`init.sql`の自動実行は、DBの保存領域が空の初回だけ行われます。既存の`db-data`がある場合、`.env`や`init.sql`を変更しても自動では反映されません。

### 既存DBにテーブルを追加する

`todos`テーブルがない場合は、現在のDBを残したまま初期化SQLを実行できます。

```powershell
docker compose exec db mysql -u root -p todo_app_db
```

`todo_app_db`は`.env`の`DB_NAME`に合わせて変更してください。rootパスワードを入力し、MySQLのプロンプトで実行します。

```sql
SOURCE /docker-entrypoint-initdb.d/init.sql;
exit;
```

### DBを作り直す

次の操作は、保存済みのタスクを含むDBデータをすべて削除します。データを削除してよい場合だけ実行してください。

```powershell
docker compose down -v
docker compose up -d --build
```

修正した環境変数と初期化SQLを使ってDBが再作成されます。

## よくあるエラー

- `Access denied ... using password: NO`：アプリに`DB_USER`や`DB_PASSWORD`が渡っているか、Composeの`app.environment`を確認してください。
- `Access denied ... using password: YES`：MySQLにユーザーが存在するか、登録済みパスワードと`.env`の値が一致するか確認してください。既存DBでは環境変数の変更だけでユーザーは更新されません。
- `Table 'todo_app_db.todos' doesn't exist`：初期化SQLが実行されていません。上記のテーブル追加手順を実行してください。

## 参考

- [MySQL公式Dockerイメージ](https://hub.docker.com/_/mysql/)
- [Docker Composeの環境変数設定](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/)
