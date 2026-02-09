---
title: Memos で呟く
emoji: "📒"
type: "tech"
topics: [docker, memos]
published: true
---

### 使ってみる

Memos というオープンソースのメモアプリがあります。

https://github.com/usememos/memos

1人でも複数人でも使えて、Markdown 形式で投稿を書くことが出来ます。

ふと頭に浮かんだことを書き残したいと思った時、ちょっとしたコマンドでも、ちょっとしたリンクでも、何でもここで呟いてしまいましょう！

```shell
mkdir memos && cd memos
mkdir .memos
nvim compose.yml  # nano / vim / code (vscode) 等々
```

```yaml:compose.yml
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    user: "${MEMOSUID:-1000}:${MEMOSGID:-1000}"
    volumes:
      - ./.memos/:/var/opt/memos
    ports:
      - "5230:5230"
    restart: unless-stopped
```

> 2026年2月10日 追記
> このアプリは動作するのに root 権限が不要でかつ Docker は何も指定しない場合はデフォルトで root として実行されるため `user` を明確に指定する。

```shell
MEMOSUID=$(id -u) MEMOSGID=$(id -g) sudo docker compose up  # 実行！
sudo docker compose up -d   # 野に放つ
```

:::message
大抵の環境では `1000:1000` だと思うので、環境変数をいちいち指定する必要はないです！
:::

[http://localhost:5230](http://localhost:5230) を開いて、初期設定をして、どんどん呟いていく！

```shell
sudo docker ps  # 様子を見る
sudo docker logs memos  # 不調がないか確認する
sudo docker restart memos   # 設定ファイルとかを更新したら再起動
```

### 外でも使いたい場合

私です。色々と方法を模索しました。結論としては Cloudflare Tunnel を Cloudflare Access と一緒に使う方法がおすすめです。無料です。Cloudflare はとても太っ腹で、私は毎日のようにお世話になってるのにまだ1円も出してません。申し訳ないですほんと...

https://blog.ynr.jp/article/network/cloudflare-tunnel/

こちらの記事にとてもお世話になりました。ありがとうございます。

以下は個人的におすすめのインストール方法です。一時的な `docker run` より何をしてるかを整理できる `compose.yml` のほうが落ち着くためです。手軽さで行ったら `docker run` のほうが良いかもしれません。

```shell
nvim compose.yml  # 追加する
nvim .env   # 機密情報なので扱いに注意
sudo docker compose up -d  # 野に放つ
```

```diff:compose.yml
+  tunnel:
+    image: cloudflare/cloudflared:latest
+    container_name: tunnel
+    command: tunnel --no-autoupdate run
+    environment:
+      TUNNEL_TOKEN: ${TOKEN}
+    network_mode: "host"
+    restart: unless-stopped
```

`TOKEN` には `docker run` の `--token` の後ろの長い文字列を与えてください。

```:.env
TOKEN=abcdefghijklmnopqrstuvwxyz
```

#### ルートを追加する

![黒いダッシュボードの画面でルートを追加している](/images/スクリーンショット_20260210_061653_キリトリ＋もっと.png)
*`localhost` ではなく `0.0.0.0` がポイントです！*

> 6:43
> この記事完成しそうにないのでこれはこれで残して別で切り上げる。
