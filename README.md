# multibook-ansible

LaravelバックエンドおよびReactフロントエンドをAmazon Linux 2023のEC2インスタンスに構築するAnsible Playbookです。

以下のroleを順に実行します：`common` → `mariadb` → `php` → `nginx` → `laravel` → `react`

## 前提

- EC2インスタンス（Amazon Linux 2023）が作成済みであること
- ローカルマシンからEC2にSSH接続できること
- EC2上の `~/.ssh/id_rsa` にGitHubへのSSH秘密鍵が配置済みであること
  - Gitリポジトリのcloneに使用します（`/home/ec2-user/.ssh/id_rsa`）
  - 対応する公開鍵がGitHubアカウントに登録済みであること
- EC2インスタンスに一時的にパブリックIPv4アドレスが付与済みであること
  - `composer install` 実行時にGitHubへのアクセスにIPv4が必要です
  - 通常の運用ではパブリックIPv4を付与しない構成を想定しているため、Ansible実行前に一時的に付与し、完了後は解除してください

## 事前準備

### 1. Ansibleのインストール

ローカルマシン（Ubuntu）にAnsibleをインストールします。

```sh
sudo apt update
sudo apt install -y ansible python3-pymysql
ansible --version
```

> macOSの場合は `brew install ansible` でインストールできます。

### 2. SSH鍵の準備

EC2への接続に使用するSSH鍵ペアを用意します。既存の鍵を使用する場合はこの手順をスキップしてください。

```sh
# 鍵ペアの生成（例）
ssh-keygen -t ed25519
```

生成した公開鍵（`~/.ssh/id_ed25519.pub`）をEC2に登録するか、EC2 Instance Connect用のキーペアを使用してください。

### 3. SSH configの設定

`~/.ssh/config` に以下を追記しておくと、hostsファイルの記述をシンプルにできます。

記載例：

```
Host i-*
  User ec2-user
  IdentityFile /home/yourname/.ssh/ec2_key
  ProxyCommand aws ec2-instance-connect open-tunnel --instance-id %h
```
```
Host hogeserver
  HostName i-xxxxxxxxxx
  User ec2-user
  IdentityFile /home/yourname/.ssh/ec2_key
  ProxyCommand aws ec2-instance-connect open-tunnel --instance-id %h
```

### 4. インベントリファイル（`hosts`）の作成

リポジトリルートに `hosts` ファイルを作成します（このファイルはgit管理外です）。

SSH configの設定が済んでいれば、インスタンスIDを書くだけで済みます：

```ini
[web]
i-07e7d58adcdc7fa9f
```
```ini
[web]
hogeserver
```

### 5. 接続確認

AnsibleからEC2へ疎通できるかを確認します。

```sh
ansible all -m ping
```

成功すると以下のように表示されます：

```
i-07e7d58adcdc7fa9f | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> `ansible.cfg` で `inventory = hosts` が設定されているため、`-i hosts` の指定は不要です。

### 6. 秘密変数ファイル（`vars/private.yml`）の作成

リポジトリルートに `vars/` ディレクトリを作成し、`private.yml` を配置します（このファイルはgit管理外です）。

```sh
mkdir -p vars
```

`vars/private.yml` の内容：

```yaml
laravel_db_name: "your_db_name"
laravel_db_user: "your_db_user"
laravel_db_password: "your_db_password"
google_client_id: "xxx.apps.googleusercontent.com"
google_client_secret: "xxxxxxx"
google_redirect_uri: "http://app.example.com/api/auth/google/callback"
app_domain: "app.example.com"
img_bucket: "your_s3_bucket"
img_domain: "img.example.com"
```

## Ansible実行

```sh
ansible-playbook site.yml
```

実行ログを詳細に表示したい場合：

```sh
ansible-playbook site.yml -v
```

## 注意

- `hosts` および `vars/private.yml` はgit管理外（`.gitignore` に記載済み）です。誤ってcommitしないよう注意してください。
- `.env` ファイルや `laravel_env.yml` もgit管理しないでください。
- Ansible実行後、EC2のパブリックIPv4アドレスが不要であれば解除してください。
