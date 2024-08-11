# mcmp-project-master

## 概要

MultiPaperを使って負荷分散マイクラサーバを立てる。  
worldデータの保持とロードバランサーを行うMasterと実際のサーバ処理を行うslaveがある。これはMaster環境構築リポジトリ。  

今回は[ラズパイ](./raspi.md)にmasterと常時起動用の[slave](https://github.com/unSerori/mcmp-project-slave)をインストールし、自分が参加する際などwin10PCを起動している間は[slave](https://github.com/unSerori/mcmp-project-slave)として負荷分散に参加し負荷を受け持つ。  

## 環境構築

1. Linux上の環境を整える。[ラズパイでの環境構築](./raspi.md)
2. mcmp-project-masterをクローン

    ```bash
    # clone
    cd /home/mcmp
    git clone git@github.com:unSerori/mcmp-project.git
    ```

3. リモートvscodeに拡張機能を入れる

    ```bash
    # 拡張機能をインポート
    cat vscode-ext.txt | while read line; do code --install-extension $line; done
    ```

4. .envファイルをコピーまたは作成

    ```env:.env
    MULTIPAPER_MASTER_URL=multipaper-master-x.y.z-all.jarのDLリンク
    BASE_IMAGE=jkdのベースイメージ。ここではeclipse-temurin:x.y.z_a-jdkを使用。
    ```

## サーバーアップデート

新しいバージョンがリリースされた場合の更新方法

- .env内のMULTIPAPER_MASTER_URLを更新
- .env内のBASE_IMAGEを適切なJKDイメージに変更

```bash
# 再ビルド&再起動
docker compose build && docker compose up -d

# (詳細なログはbuildコマンドに以下を追加)
--progress=plain
```

## マイクラ鯖起動

```bash
# SSHで入った後以下実行でコンテナーup
docker compose up -d

# サーバーシャットダウン
```
