# PostgreSQL と PostgREST を Docker Compose で：データベースと REST API を教えるためのラボ

[English](README.md) | [Français](README.fr.md) | [Español](README.es.md) | [日本語](README.ja.md) | [Deutsch](README.de.md)

教室向けにすぐ使える **Postgres + PostgREST** のラボです。
[MitsuruTech 学習プラットフォーム](https://mitsurutech.co.uk/ja/learning)のもので、学校やカレッジでは、
生徒一人ひとりが自分専用のデータベース、コンテナー、ターミナルをブラウザーで使えます。
生徒はそれぞれ自分の PostgreSQL と、自分の [PostgREST](https://postgrest.org) を持ちます。
PostgREST はデータベースのすべてのテーブルを REST API にします。2つの横にあるターミナルには
`psql` と `curl` が入っています。

ラボ全体が `docker-compose` ファイル1つ、`compose.yaml` です。自分のマシンでも同じように動きます。

```
docker compose up
```

## プラットフォームで使う

「コンテナーラボ」の課題を作り、「GitHub リポジトリ」を選んで「サンプルを使う」を押します。変更したいときは、
このリポジトリをフォークして、ラボをそのフォークに向けてください。公開すると、
生徒一人ひとりのコピーは、公開したコミットから始まります。ヘルプマニュアルには
[このラボのページ](https://mitsurutech.co.uk/ja/help/lab-database-api)と、
[コンテナーラボのガイド](https://mitsurutech.co.uk/ja/help/labs)があり、残りの内容、つまり
compose ファイルに書けること、1つのコピーが使えるメモリ量、教員が生徒のターミナルを見たり
引き継いだりする方法を説明しています。

まだカレッジのアカウントがありませんか？[最初の1年は無料](https://mitsurutech.co.uk/ja/pricing)で、
生徒50人まで使えます。[サインアップ](https://mitsurutech.co.uk/ja/signup)に必要なのはメールアドレスだけです。

## このラボが想定している授業

ラボを開き、ターミナルで進めます（シェルが初めてなら、まず
[Linux ターミナルのガイド](https://mitsurutech.co.uk/ja/help/linux-terminal)をどうぞ）。

1. **名前でデータベースに話しかける。** どのサービスも、本番と同じように、ラボ自身の
   ネットワーク上で名前から届きます。

   ```
   psql -h db -U shop
   ```

   パスワードは `shop` です。テーブルを作って、何か入れてみましょう。

   ```sql
   create table notes (id serial primary key, body text not null);
   insert into notes (body) values ('hello');
   notify pgrst, 'reload schema';
   ```

   最後の行は、PostgREST にデータベースをもう一度見るよう頼みます。PostgREST は起動時に
   1回だけ読み込んでおり、そのときテーブルはまだなかったからです。

2. **API を使う。** `\q` で `psql` を抜けます。

   ```
   curl http://api:3000/notes
   curl -X POST -H 'Content-Type: application/json' -d '{"body":"from curl"}' http://api:3000/notes
   curl 'http://api:3000/notes?order=id.desc'
   ```

3. **何が残るかを見る。** ホームフォルダーの中と外に、1つずつファイルを書きます。

   ```
   echo kept > ~/kept.txt
   echo gone > /tmp/gone.txt
   ```

   ラボを止めて、もう一度起動します。データベースはファイルを名前付きボリューム `pgdata` に
   保存しているのでノートは残っており、プラットフォームがホームフォルダーも保存するので
   `~/kept.txt` も残っています。`/tmp/gone.txt` は消えています。ボリュームの外にあるものは、
   コンテナーが止まると消えるからです。これはプラットフォームの癖ではなく、本番でもコンテナーは
   そう動きます。本物のデプロイが、何をボリュームに入れるかを決める理由でもあります。

4. **バックアップして復元する。**

   ```
   pg_dump -h db -U shop shop > ~/backup.sql
   psql -h db -U shop -c 'drop table notes'
   psql -h db -U shop -f ~/backup.sql
   curl http://api:3000/notes
   ```

5. **ブラウザーで開く。** ラボのページの「ブラウザーで」で、**A web API over every table** を押し、
   アドレスに `/notes` を付けます。同じ API をラボの外から開いたもので、開けるのは自分と
   教員だけです。自分のコンピューターの Postman や `curl` からは、同じページに表示される
   ラボのトークンを `X-Lab-Token` ヘッダーに付けて送ります。

## ファイルが示していること

| `compose.yaml` の中 | あるわけ |
| --- | --- |
| `volumes: pgdata` | `db` のうち残る唯一の部分 |
| `db` の `healthcheck` | `api` が、Postgres が起動しただけでなく応答するようになってから始まるように |
| `depends_on: condition: service_healthy` | その残りの半分 |
| `deploy.resources.limits.memory` | 1つのコピーが使えるメモリのうち、各サービスの取り分 |
| `expose: 3000` | API のポート。ラボのネットワーク内だけで、マシン側には何も公開されない |
| `labels: lab.title` | ラボのページが各サービスを呼ぶ名前 |
| `labels: lab.open` | ラボのページがリンクするポート。ブラウザーで開ける |

使うのはイメージだけで、ビルドはありません。ラボは、リポジトリにある `Dockerfile` からサービスを
ビルドすることもでき、ラボを公開するときにプラットフォームが GitHub 上でそれを行います。

## ほかのラボ

- [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea)：Git と Gitea。自分専用の
  Git サーバーにクローン、コミット、プッシュして、Web 画面で確認します。
- [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin)：MySQL と、
  最初からサインイン済みの phpMyAdmin。

## このラボについて

[Mitsuru Technologies Ltd](https://mitsurutech.co.uk/ja)（イングランド・ウェールズで登録、会社番号 16755264）が、
[MitsuruTech 学習プラットフォーム](https://mitsurutech.co.uk/ja/learning)のために作っています。
プラットフォームは、学校とカレッジ向けに、Python、Java、Go、Node、SQL、Linux をブラウザーで使えるようにし、
コンテナーラボ、GitHub からデプロイする Web アプリ、AWS アカウント不要の AWS S3・DynamoDB・SQS、
自動で採点するクイズを備えています。アカウントなしでコードを試すなら
[プレイグラウンド](https://mitsurutech.co.uk/ja/playground)をどうぞ。

## ライセンス

この作品は [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) でライセンスされています。Mitsuru Technologies Ltd のクレジットを表示し、商用に使わず、これをもとに作ったものを同じライセンスで共有するなら、学習と授業のために共有・改変できます。全文は `LICENSE` ファイルにあります。
