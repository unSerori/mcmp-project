# ラズパイでの環境構築

OSインストールから作業用PCから公開鍵SSH接続できるまで。  
ここではラズパイ:Raspberry Pi 4 Model B Rev 1.5 /4GBを使用した。  
ハードやOSはほかでもある程度代用可能だとは思う。

## 環境

- Hard master: Raspberry Pi 4 Model B Rev 1.5 /4GB
- OS: DietPi v9.5.1 - DietPi_RPi-ARMv8-Bookworm.img.xz
- Debian version: 12.6

## 環境構築手順

### インストールと諸設定

1. EtcherでDietPiをストレージに書き込み
2. wi-fi設定など
3. ユーザーを追加

    ```bash
    useradd mcmp
    passwd mcmp-pass
    ```

### SSH接続関連

1. エディターを追加

    ```bash
    # install
    sudo apt install -y vim
    ```

2. ホスト名変更
    1. /etc/hostname, /etc/hostsのhostnameを変更。今回は`mcmp-srv`にした

        ```bash
        vim /etc/hostname
        vim /etc/hosts
        ```

    2. mDNSサーバインストール

        ```bash
        # install
        sudo apt install -y avahi-daemon
        # 起動
        sudo systemctl start avahi-daemon
        # 自動起動
        sudo systemctl enable avahi-daemon

        # よくわからない
        sudo apt install -y  libnss-mdns

        # ufwを一度有効化
        sudo apt install ufw
        # STOP
        systemctl stop ufw
        # START
        systemctl start ufw

        # これでusername@hostname.localでsshできるはず。
        ```

3. 作業用PC, GitHubとのSSH確立
    1. Gitのインスコ

        ```bash
        # パッケージリストを更新
        sudo apt update
        # software-properties-commonパッケージをインストール
        sudo apt install software-properties-common
        # PPAを追加
        sudo add-apt-repository ppa:git-core/ppa
        # パッケージリストを更新
        sudo apt update
        # Gitをアップグレード
        sudo apt upgrade git

        # 初期設定
        git config --global user.email "example@email.com"
        git config --global user.name "example"
        ```

    2. 作業用PCとのSSH接続

        ```bash
        # sshサーバが入ってなかった
        sudo apt install openssh-server

        # 22ポートのバインドに失敗した。。

        # netstatでポートの使用者確認
        sudo apt install net-tools
        sudo netstat -tulpn | grep :22

        # 邪魔しているタスクを止めて
        sudo systemctl stop dropbear
        sudo systemctl disable dropbear
        # open ssh を開始
        sudo systemctl start ssh
        sudo systemctl enable ssh
        sudo systemctl status ssh

        # sshの許可設定
        sudo vim /etc/ssh/sshd_config 
        # PubkeyAuthentication yes
        # AuthorizedKeysFile .ssh/authorized_keys
        # PasswordAuthentication no
        # PermitEmptyPasswords no

        # 鍵ペアの生成
        ssh-keygen -t rsa
        # 鍵の送信
        scp id_rsa.pub mcmp@mcmp-srv.local:~/.ssh

        # パーミッション
        chmod 700 /home/mcmp/.ssh
        chmod 600 /home/mcmp/.ssh/authorized_keys
        ```

    3. GitHubとのSSH接続

        ```bash
        # githubと通信するために鍵ペア作成
        cd ~/.ssh
        ssh-keygen -t rsa
        # パーミッション
        chmod 600 id_rsa
        chmod 600 id_rsa.pub

        # 公開鍵をgithubに渡す
        cat id_rsa.pub
        # githubに登録

        # 疎通ができなかったため設定ファイル追加
        Host github github.com
            HostName github.com
            IdentityFile ~/.ssh/id_rsa # 秘密鍵
            User git

        # 疎通
        ssh -T github
        ssh -T git@github.com
        ```

4. 作業用pcからSSH接続
    1. 作業用PCのVS Codeに`Remote Development`を追加
    2. リモートエクスプローラーからSSHの歯車マークの設定をする
    3. 接続

### Dockerインスコ

1. リポジトリのセットアップ

    ```bash
    # 念のため古いリポジトリを削除 # removeでなくpurgeで設定・関連ファイルまで削除
    apt purge lxc-docker*
    apt purge docker.io*
    sudo apt remove docker docker-engine docker.io containerd runc

    # aptが HTTPS 経由でリポジトリにアクセスしパッケージをインストールできるようにリポジトリをセットアップ
    apt install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release

    # GPG鍵を追加
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    # 安定板を追加
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

2. Docker Engineのインスコ

    ```bash
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    # 確認
    docker run hello-world
    ```

3. dockerコマンドに対する権限を付与

    ```bash
    # sudoなしでの実行権限付与
    usermod -aG docker mcmp

    # 起動時に自動起動
    systemctl enable docker.service
    systemctl enable containerd.service

    # 再起動
    sudo systemctl restart docker.service
    sudo systemctl restart containerd.service
    ```
