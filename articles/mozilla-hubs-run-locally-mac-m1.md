---
title: "Mozilla Hubs Cloudをローカル（M1搭載Macbook）で実行してみた"
emoji: "🦆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Mozilla Hubs", "Hubs Cloud", "WebXR", "VR"]
published: false
---

# はじめに

## Mozilla Hubsとは

ブラウザFirefoxで有名なMozilla社が開発したソーシャルWebVRサービスであり、PCからもVRデバイスからもブラウザを開いてアクセスできます。しかもオープンソース！

[Mozilla Hubs](https://hubs.mozilla.com)

詳しい紹介は割愛します。もっと知りたい方は[REALITY社 GREE VR Studio Laboratory 白井先生のスライド](https://vr.gree.net/wp-content/uploads/2020/07/Hubs-VRStudioLab-20200513-Shirai.pdf)や[東京大学VRセンターの資料](https://vr.u-tokyo.ac.jp/wp-content/uploads/2020/04/2020_0419_東大vrセンターが教えるMozillaHubsの使い方_rev.pdf)がおすすめです。

ユーザーが自分で管理できるサーバー（AWSなど）でHubsをホストしたければ、[Hubs cloud](https://hubs.mozilla.com/cloud)が使えます。Hubs cloudを使えば、Hubsのフロントエンドやシーン作成ツールなど、一部のコンポーネントをカスタマイズすることもできます。

## なぜこの記事を書く？

私が所属しているサークルUT-virtualも新歓活動にMozilla Hubsを使っていましたが、自分は全く別の経緯でMozilla Hubsを触り始めました。ここ数日、自分のパソコンでローカルで環境構築していまして、ようやく実行できるようになり、せっかくなので記事にして共有させていただきたいなと思いました。

## なぜローカルで実行したい？

前述したように、Hubs cloudは一部のコンポーネントしかカスタマイズできず、もしバックエンドの方などにもカスタマイズしたければ、ローカルで実行できる環境を構築して、そこで開発しないと対応できないのではないかと思います。私もちょうど今、Hubs cloudをカスタマイズする需要のある学校のプロジェクトに携わり始めたところです。

あと勉強のためにもなりそうですね。GREE VR Studio Laboratoryの方がHubs cloudを「解剖」するためにローカルで走らせたそうです（記事は[こちら](https://note.com/reality_eng/n/n6f6f58caa7da#c6ea6e9b-ef7e-4a4a-a30f-417a83ae9675)）。

# 環境構築を始めましょう

[同じ内容の英語版](https://gist.github.com/YHhaoareyou/199410454695d804db5fe7f569d055f0)も書きました

## 参考

今回は主に[このalbirrkarimさんの説明](https://github.com/albirrkarim/mozilla-hubs-installation-detailed#readme)通りにやりつつ、Hubs cloudを構成するそれぞれのコンポーネントのレポジトリのREADMEを参考し：

- Reticulum: https://github.com/mozilla/reticulum.git
- Dialog: https://github.com/mozilla/dialog.git
- Hubs: https://github.com/mozilla/hubs.git
- Spoke: https://github.com/mozilla/spoke.git

そしてこれらの説明に言及されていないが自分が遭遇してしまった問題を対応してきました。

プロジェクト全体像は、[albirrkarimさんが作ったグラフ](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)、もしくは[GREE VR Studio Laboratoryさんのスライド（p.35から）](https://note.com/reality_eng/n/n6f6f58caa7da#c6ea6e9b-ef7e-4a4a-a30f-417a83ae9675)）をご覧いただければと思います。

# 言っておきますが...

- なるべく手順を細かく記述するように頑張ってみましたし、ここで書かれた手順はちゃんと自分のM1搭載のMacbookで動作できていますが、他のパソコンの環境で（例え同じくらいのスペックでも）完璧に動作することは保証できません。
- 公式のアップデートにより、ここで書かれた手順が動作しなくなる可能性もあります。そうなりましたら、なるべくアップデートに追いついてこの記事を更新します。
- この記事で間違ったところがありましたら、ご指摘いただければ幸いです！

# スペック

### ハード

- チップ: Apple M1
- メモリ: 16GB（8GB以上であれば大丈夫らしい）

### ソフト

- NodeJS: v16.16.0
- npm: 9.1.3

# 1. インストール＆初期設定

## 1.1 Reticulum

ElixirとPhoenix Frameworkで構築されたバックエンド

### 1.1.1 Clone

```bash
git clone https://github.com/mozilla/reticulum.git
cd reticulum
```

### 1.1.2 PostgreSQL

ReticulumのデータベースはPostgreSQLを使用している

1. PostgreSQLをインストール:
```
brew install postgresql
```

2. バージョンを確認 (**11**以上がおすすめ):
```
psql --version
```

3. データベース `postgres` にログイン:
```
psql postgres
```

4. ユーザを新規作成し（ユーザネームもパスワードも'postgres'）、Superuserにする:
```
CREATE USER postgres WITH ENCRYPTED PASSWORD 'postgres';
ALTER USER postgres WITH SUPERUSER;
```

5. `\du`でユーザ一覧を見て確認し、`\q`でpsqlシェルを終了する

#### Elixir と Erlang

今cloneしたReticulumのレポジトリにある `.tool-versions` というファイルに、必要なElixir と Erlangのバージョンが記載されています。

[このチュートリアル](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf) に従い、該当するバージョンのElixir と Erlangをインストールしてください。

### 1.1.3 Reticulumのセットアップ

1. 必要なパッケージをインストールする
```
mix deps.get
```

2. PostgreSQLにデータベースを作成
```
mix ecto.create
```
`psql -l` でデータベース一覧を見て `ret_dev` が作られたかどうかを確認しましょう。

もしここでデータベースの作成が失敗したら、PostgreSQLのユーザのパスワードが間違った可能性があります。パスワードは `config/dev.exs` で記載されたのと同じ（つまり`postgres`）になければならないため、ここで`psql postgres`でpsqlシェルに入って、パスワードを再設定します：
```
ALTER USER postgres WITH PASSWORD 'postgres';
```

もし「データベース`ret_dev`が存在しない」のようなエラーが出ましたら、psqlシェルに入って`create database ret_dev;`を実行してみてください。

3. `mkdir -p storage/dev`を実行してReticulumレポジトリの中にフォルダーを作成します

### 1.1.4 ローカルのDialogインスタンスに対してReticulumをセットアップ

1. `config/dev.exs`でJanusのホストを変更します：
```elixir
# Around the top of this file
dev_janus_host = "localhost"
```
2. `config/dev.exs`でJanusのポートを変更します：
```elixir
# line 216
config :ret, Ret.JanusLoadStatus, default_janus_host: dev_janus_host, janus_port: 4443
```
3. `lib/ret_web/plugs/add_csp.ex`でCSPルールを書き換える
```elixir
# line 107
default_janus_csp_rule =
   if default_janus_host,
      do: "wss://#{default_janus_host}:#{janus_port} https://#{default_janus_host}:#{janus_port} https://#{default_janus_host}:#{janus_port}/meta",
      else: ""
```

## 1.2 Dialog

mediasoupに基づき、カメラストリーミングや画面共有など、音声や映像のリアルタイム通信を担う WebRTC SFU サービスです。

### 1.2.1 Cloneしてパッケージをインストール
```bash
git clone https://github.com/mozilla/dialog.git
cd dialog
npm ci
```

### 1.2.2 RSA暗号の設定

ReticulumはDialogインスタンスに対して実行する場合、RSA暗号を使用し、鍵はプライベートなリポジトリに保存されているそうです（[参考](https://github.com/mozilla/hubs/discussions/3323)）。

ローカルで実行するため、別途暗号を作って設定しましょう

1. [こちら](https://travistidwell.com/jsencrypt/demo/)でRSAの公開鍵と秘密鍵を作成
2. Dialogのフォルダーの中にファイル`certs/perms.pub.pem`を作成し、公開鍵をそのまま貼り付ける
3. Reticulumのフォルダーの中にBashファイル`scripts/run-local.sh`を作成し、以下を貼り付ける：
```bash
PERMS_KEY="-----BEGIN RSA PRIVATE KEY----- ...秘密鍵の各行の前に \n を追加し、鍵全体を1行にし、コピーしてここに貼り付けてください... -----END RSA PRIVATE KEY-----"
PERMS_KEY="$PERMS_KEY" iex -S mix phx.server
```
秘密鍵を他のところに保存して`scripts/run-local.sh`に読み込まれるようにしても大丈夫です。
4. `chmod +x ./scripts/run-local.sh`を実行し、`scripts/run-local.sh`を実行可能なファイルにする

## 1.3 Spoke

Hubsの3Dシーンを作成・編集するためのGUI
![](/images/mozilla-hubs-run-locally-mac-m1/03-spoke.png)

### 1.3.1 Cloneしてパッケージをインストール

```
git clone https://github.com/mozilla/Spoke.git
cd Spoke
yarn install
```

もしこのエラー `The chromium binary is not available for arm64`によって`yarn install`が失敗したら、それは`puppeteer`で設定した Chrome の実行パスが Apple M1 と一致しないからです（[参考](https://github.com/puppeteer/puppeteer/issues/6641)）。

以下のようにするとエラーが回避されます：

1. （もしご自分のMacに入っていなければ）Chromiumをインストールします
```
brew install chromium --no-quarantine
```
2. 正しい Chrome の実行パスでもう一度パッケージをインストール
```
PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true PUPPETEER_EXECUTABLE_PATH=`which chromium` yarn install
```

## 1.4 Hubs

一般ユーザ用・管理者用のHubsのフロントエンド

![](/images/mozilla-hubs-run-locally-mac-m1/03-hubs.png)
![](/images/mozilla-hubs-run-locally-mac-m1/03-admin.png)

### 1.4.1 Cloneしてパッケージをインストール

```
git clone https://github.com/mozilla/hubs.git
cd hubs
npm ci
```

### 1.4.2 adminフォルダーに移動してパッケージをインストール

```
cd admin
npm ci

/* `npm ci` が失敗したら */
npm install
```

# 2. ドメインをlocalhostにする

全てのフォルダー（reticulum・dialog・spoke・hubs・hubs/admin）で`hubs.local`を`localhost`に置き換えましょう。

# 3. HTTPS (SSL)のセットアップ

HTTPS用の証明書と鍵を作成しましょう。

## 3.1 証明書を生成して設定する

Reticulumで：

```bash
cd reticulum
mix phx.gen.cert
```

鍵 `selfsigned_key.pem` と証明書 `selfsigned.pem` がフォルダー `priv/cert` の中に生成されます。

`selfsigned_key.pem` を `key.pem`に改名し、`selfsigned.pem` を `cert.pem`に改名します。

#### `key.pem` と `cert.pem` ができました

![](/images/mozilla-hubs-run-locally-mac-m1/03-01.png)

Finderで `cert.pem` をダブルクリックし、Keychain Access 画面を開きます。

`cert.pem` に当たる証明書をダブルクリックし、`Trust` タブを開き、**When using this certificate** を **Always Trust**にします。

![](/images/mozilla-hubs-run-locally-mac-m1/03-02.png)

Finderで `cert.pem` と `key.pem` をコピーしておきます。

次のステップからこの二つのファイルをhubs、hubs admin、spoke、dialog、そしてreticulumにコピペします。

## 3.2 reticulumで証明書と鍵のパスを修正する

`reticulum/config/dev.exs`の中のこの部分を：
```elixir
# line 19
config :ret, RetWeb.Endpoint,
  ...
  https: [
    ...
    keyfile: "#{File.cwd!()}/priv/dev-ssl.key",
    certfile: "#{File.cwd!()}/priv/dev-ssl.cert"
  ],
  ...
```
以下のように、keyfileとcertfileのパスを修正します：
```elixir
# line 19
config :ret, RetWeb.Endpoint,
  ...
  https: [
    ...
    keyfile: "#{File.cwd!()}/priv/cert/key.pem",
    certfile: "#{File.cwd!()}/priv/cert/cert.pem"
  ],
  ...
```

## 3.3 それぞれのレポジトリでHTTPSをセットアップ

1. Hubs: `reticulum/priv/cert/cert.pem` と `reticulum/priv/cert/key.pem`をフォルダー`hubs/certs`にコピペします
2. Hubs admin: `reticulum/priv/cert/cert.pem` と `reticulum/priv/cert/key.pem`をフォルダー`hubs/admin/certs`にコピペします
3. Spoke: `reticulum/priv/cert/cert.pem` と `reticulum/priv/cert/key.pem`をフォルダー`Spoke/certs`にコピペします
4. `reticulum/priv/cert/cert.pem` と `reticulum/priv/cert/key.pem`をフォルダー`dialog/certs`にコピペして、
  - `cert.pem` を `fullchain.pem`に改名し、
  - `key.pem` を `privkey.pem`に改名します。

# 4. 実行

reticulum・dialog・spoke・hubs・hubs adminを同時に実行するために、五つのターミナルを開きます。

## 4.1 reticulum を実行する

```bash
cd reticulum
./scripts/run-local.sh
```

## 4.2 dialog を実行する

`package.json` を開いて以下のように編集：
```
"scripts": {
    ...
    "start": "MEDIASOUP_LISTEN_IP=127.0.0.1 MEDIASOUP_ANNOUNCED_IP=127.0.0.1 DEBUG=${DEBUG:='*mediasoup* *INFO* *WARN* *ERROR*'} INTERACTIVE=${INTERACTIVE:='true'} node index.js"
    ...
  },
```

そして `npm start` を実行します。

`package.json` を編集せず、以下のようにそのまま実行しても同じです。
```
MEDIASOUP_LISTEN_IP=127.0.0.1 MEDIASOUP_ANNOUNCED_IP=127.0.0.1 npm start
```

もし以下のようなエラーが出ましたら：
```
Error: listen EADDRINUSE: address already in use 7000
```
`config.js`でadminHttpのポートを7000から7001や他のポートにします：
```
adminHttp:
{
   listenIp   : '0.0.0.0',
   listenPort : process.env.ADMIN_LISTEN_PORT || 7001
},
```

## 4.3 spoke を実行する

```bash
cd Spoke
./scripts/run-local-reticulum.sh
```

## 4.4 hubs と hubs admin を実行する

```bash
cd hubs
npm run local
```

```bash
cd hubs/admin
npm run local
```

# 5. ローカルでMozilla Hubsにアクセス

| Server     | URL                                                          |
| ---------- | ------------------------------------------------------------ |
| Hubs       | [https://localhost:4000](https://localhost:4000)             |
| Hubs admin | [https://localhost:4000/admin](https://localhost:4000/admin) |
| Spoke      | [https://localhost:4000/spoke](https://localhost:4000/spoke) |

## ログイン

[https://localhost:4000/](https://localhost:4000/) にアクセスします
![](/images/mozilla-hubs-run-locally-mac-m1/05-01.png)

右上のボタンをクリックし、メールアドレスを入力してログインします
![](/images/mozilla-hubs-run-locally-mac-m1/05-02.png)

入力されたメールアドレスに認証メールが送られるはずですが、Reticulumはローカルで動作しているため、メールは送ることが実際にはできません。
![](/images/mozilla-hubs-run-locally-mac-m1/05-03.png)

Reticulum が起動しているターミナルを開き、メールアドレス検証用のリンクを探します。そのリンクをコピーして、新しいブラウザのタブを開いて貼り付ける
![](/images/mozilla-hubs-run-locally-mac-m1/05-04.png)

検証ができたらタブを閉じて大丈夫です
![](/images/mozilla-hubs-run-locally-mac-m1/05-05.png)

Hubsのトップページに戻ると、右上にメールアドレスが表示されます
![](/images/mozilla-hubs-run-locally-mac-m1/05-06.png)

## アカウントを管理者として設定する

Reticulum が起動しているターミナルを開き、Enter キーを押して Elixir's interactive shell (iex) にアクセスします
![](/images/mozilla-hubs-run-locally-mac-m1/05-07.png)

次のコマンドを実行して、最初に登録されたアカウント（先ほどサインインに使用したアカウント）を管理者として設定します：
```elixir
Ret.Account |> Ret.Repo.all() |> Enum.at(0) |> Ecto.Changeset.change(is_admin: true) |> Ret.Repo.update!()

# Enum.at(0) の '0' は最初に登録されたアカウントを指します。他の番号を使用すると、他のアカウントを管理者として設定することができます
```

Hubsのウィンドウを更新すると、ウィンドウの上部に利用可能なリンク（Scene editor・Admin）が表示されます
![](/images/mozilla-hubs-run-locally-mac-m1/05-08.png)

`Admin`リンクをクリックすると管理者向け画面（hubs/adminでホストされています）に遷移されます。[https://localhost:4000/admin](https://localhost:4000/admin)からアクセスすることも可能です。
![](/images/mozilla-hubs-run-locally-mac-m1/05-09.png)

`Scene editor`リンクをクリックすると **Spoke**（Hubs用の3Dシーンを作成・編集するアプリケーション）の画面に遷移されます。 [https://localhost:4000/spoke](https://localhost:4000/admin)からアクセスすることも可能です。
![](/images/mozilla-hubs-run-locally-mac-m1/05-10.png)

## Spokeを使ってみる

`New project` ボタンをクリックしてシーン作成画面を開きます

シーン作成画面の読み込み中、CORSの問題のせいか、一部のアセットが正常に読み込まれないことがあります。その場合は、リンクをハイライトし、URLバーにドラッグすることでアセットをダウンロードすることができます。その後、またパソコンからSpokeにインポートすることができます。
![](/images/mozilla-hubs-run-locally-mac-m1/05-11.png)

次に、先ほどインポートしたアセットを含む、任意のアセットをシーンにドラッグします。位置や回転などの属性は、GUI上で編集することができます。
![](/images/mozilla-hubs-run-locally-mac-m1/05-12.png)

完了したら、右上のPublishボタンをクリックしてシーンを公開します。公開したら、'View Your Scene'ボタンをクリックしてこの新しいシーンで直接ルームを作成することができます。（一旦Hubsのホームページへ戻ってそこからRoomを作成することもできます）
![](/images/mozilla-hubs-run-locally-mac-m1/05-13.png)

## Create a room and interact with other accounts

Hubsのホームページ[https://localhost:4000](https://localhost:4000)から、'Create Room'ボタンをクリックすると、ルーム作成画面へ遷移されます。
![](/images/mozilla-hubs-run-locally-mac-m1/05-14.png)

'Options'ボタンをクリックし、シーンを先ほど作ったものに設定します
![](/images/mozilla-hubs-run-locally-mac-m1/05-15.png)
![](/images/mozilla-hubs-run-locally-mac-m1/05-16.png)
![](/images/mozilla-hubs-run-locally-mac-m1/05-17.png)

下までスクロールして'Apply'ボタンを押し忘れずに。

それではJoin Roomボタンをクリックしてルームに入りましょう！

別のブラウザで[https://localhost:4000](https://localhost:4000)にアクセスし、別のメールアドレスで上記と同じ手順でログインし、新しく作成した部屋と同じURLにアクセスすることができます。

両方のアカウントが入室したら、WASDキーや矢印キーでアバターを動かし、マウスでドラッグして視点を回転させると、互いのアバターが見えるようになるはずです！
![](/images/mozilla-hubs-run-locally-mac-m1/05-18.png)

# 終わりに

今回はMozilla Hubsをローカル（M1 MacOS）で実行する手順を共有させていただきました。
ただこれらの手順は他のパソコンでもうまく動作することが保証されません。
それでも自分の経験が誰かに役に立てれば嬉しいです。
拙い日本語で出来上がった長文なので、もし読みづらいところがありましたら、どうかお許しください。
この記事でもしどこか間違ったところがありましたら、ぜひご指摘いただければ幸いです。

Mozilla Hubsについてまた他の記事を書くかもしれません（どこかにデプロイするとか、どれかのコンポーネントを置き換えてみるとか）。
XR（主にVRかな）に関連する他の記事もこれから書かせていただく予定です！