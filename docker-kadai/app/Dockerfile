# ベースイメージの指定（Goのバージョン1.22を使用）
FROM golang:1.22

# 作業ディレクトリの作成と移動
WORKDIR /app

# go.modファイルをコピー（MySQLドライバのダウンロードに必要）
COPY go.mod ./

# ライブラリをダウンロード（キャッシュを活用）
RUN go mod download

# ホストのファイルをコンテナ内にコピー
COPY . .

# 必要な依存関係が揃うように調整
RUN go mod tidy

# アプリをビルド
RUN go build -o /todo-app .

# コンテナ起動時に実行するコマンド
CMD ["/todo-app"]