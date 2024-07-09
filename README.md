# mcmp-project-master

## 概要

MultiPaperを使ってマイクラサーバの負荷を分散する  
通常は[ラズパイ](./raspi.md)をmasterとして稼働させておき、自分が参加する際などwin10PCを起動している間のみMultiPaperに接続しslaveとして負荷を受け持つ。  

## 環境構築

1. Linux上の環境を整える。[ラズパイでの環境構築](./raspi.md)
2. mcmp-projectをクローン

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
