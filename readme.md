```bash
GOGS_DIR=~/temp/gogs/
GITEA_DIR=./

# cd ./docker-compose/gitea

# 念のためバックアップ
${GOGS_DIR}gogs buckup 
# DBデータ
pg_dump -U gogs --format custom > gogs_latest.sql
#sudo docker exec gogs_gogs_postgres_1 pg_dump -U gogs --format custom > gogs_latest.sql

# cp gogs_latest.sql ./gogs_latest.sql

# リポジトリ情報　コピーするかどうか迷う
# ネットワークドライブでマウントするのはありなのかどうか？
cp -r ~/gogs-repositories ./gitea-repositories

cp ${GOGS_DIR}custom/conf/app.ini ./app_gogs.ini
```

#### app_gogs.ini書き換え

コメントのところはgogsの方にあれば適用する

```bash 
APP_NAME = Gitea
RUN_USER = git
# [server]
# APP_DATA_PATH = /data/gitea
[database]
HOST = gitea_db:5432
PATH = /data/gitea/gitea.db
NAME     = gitea
USER     = gitea
PASSWD   = gitea
[repository]
ROOT = /repositories 
# [session]
# PROVIDER_CONFIG = /data/gitea/sessions
[picture]
AVATAR_UPLOAD_PATH      = /data/gitea/avatars
[attachment]
PATH = /data/gitea/attachments
[log]
ROOT_PATH = /data/gitea/log
```

```bash
cp ${GOGS_DIR}gogs_latest.sql ${GITEA_DIR}


# 止めてデータを削除する
sudo docker-compose down --rmi all --volumes
sudo rm -rf ./gitea/
sudo rm -rf ./postgres/

sudo docker-compose up -d gitea_db

mkdir -p ./gitea/gitea/conf/
cp ./app_gogs.ini ./gitea/gitea/conf/app.ini
# 以下2つはカスタマイズしてなければないかも
cp -r ${GOGS_DIR}custom/public/ ./gitea/gitea/
cp -r ${GOGS_DIR}custom/templates/ ./gitea/gitea/
# メインデータ
cp -r ${GOGS_DIR}data/* ./gitea/gitea/

# gogs/custom/conf に他に必要そうなデータがあれば
# 上記同様に`gitea/gitea/custom/conf/`へコピーする

# ownerをgiteaに直してリストアする
cat gogs_latest.sql | sudo docker exec -i gitea_gitea_db_1 pg_restore  -U gitea -d gitea --no-owner --role=gitea


# gitea起動
sudo docker-compose up -d

# 認証キーの再生成
#  管理者パネル ダッシュボードで 以下を実行
#  '.ssh/ authorized_keys' ファイルを再生成します。（警告：Gogsキー以外は失われます）


# gitea バージョンアップｘｎ
#  マイナーバージョンの変更＝DBの変更
#  マイナーバージョン5~6程度がDBマイグレ可能な範囲
#  https://github.com/go-gitea/gitea/issues/9953
# ./gitea/gitea/logs/gitea.log  でマイグレの成否を確認する

# git hooks 
#  リポジトリのディレクトリが変化している　or gogs を辞めた　ことで機能しなくなる
#
# git側の仕組みなので移行が面倒
# 使ってる人居ないと思っているが、使っている人がいるか確認する
# 　→誰もいない場合　全消しして利用しない
#
# 　　https://forum.cloudron.io/topic/3975/gitea-git-hooks-disabled/6?lang=ja
# 　　セキュリティの観点でv1.12以降デフォルト廃止となった
# 　　　`app.ini`に以下を設定することで有効にできる
# 　　　　[security]
# 　　　　DISABLE_GIT_HOOKS = false
# 
#   →使っている場合　カスタマイズしているやつをバックアップ
#   →全消し、デフォルトの再作成
#   →カスタマイズしたやつの手作業移行
#   https://github.com/go-gitea/gitea/issues/4286
#
# 以下全消し手順
rm -rf ~/temp/gogs-repositories/*/*.git/hooks/pre-receive.d/pre-receive
rm -rf ~/temp/gogs-repositories/*/*.git/hooks/post-receive.d/post-receive

# 管理者パネル ダッシュボードから以下を実行し、デフォルトのhookを整備する
# すべてのリポジトリの pre-receive, update, post-receive フックを更新する。


# 手元のGitからリモートリポジトリを追加設定し
# PUSHできることを確認する

# backup.shの設定

# リバースプロキシ切替作業


## GOGSに戻すときは以下を消さないとhookでエラーになる
rm -rf ~/temp/gogs-repositories/*/*.git/hooks/pre-receive.d/*
rm -rf ~/temp/gogs-repositories/*/*.git/hooks/post-receive.d/*
rm -rf ~/temp/gogs-repositories/*/*.git/hooks/update.d/*
```