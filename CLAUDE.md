# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

Amazon Linux 2023のEC2インスタンスに、LaravelバックエンドとReactフロントエンドをデプロイするAnsible Playbookです。

## 実行コマンド

```sh
# 接続確認
ansible all -m ping

# Playbook実行
ansible-playbook site.yml

# 詳細ログ付き実行
ansible-playbook site.yml -v

# 特定のroleのみ実行
ansible-playbook site.yml --tags <role_name>

# 構文チェック
ansible-playbook site.yml --syntax-check

# ドライラン（変更なし確認）
ansible-playbook site.yml --check
```

## 必須ファイル（git管理外）

実行前に以下2ファイルを手動作成する必要があります：

**`hosts`** — インベントリファイル
```ini
[web]
<EC2インスタンスIDまたはSSH Configのエイリアス>
```

**`vars/private.yml`** — 秘密変数
```yaml
laravel_db_name: "your_db_name"
laravel_db_user: "your_db_user"
laravel_db_password: "your_db_password"
google_client_id: "xxx.apps.googleusercontent.com"
google_client_secret: "xxxxxxx"
google_redirect_uri: "https://app.example.com/api/auth/google/callback"
app_domain: "app.example.com"
img_bucket: "your_s3_bucket"
img_endpoint: "https://img.example.com"
```

## アーキテクチャ

### Role実行順序

`common` → `mariadb` → `php` → `nginx` → `laravel` → `react`

### 各Roleの役割

| Role | 主な処理 |
|------|---------|
| `common` | タイムゾーン・ロケール設定、基本パッケージ（git, nodejs, npm等）インストール |
| `mariadb` | MariaDB 10.5インストール・起動、Laravel用DB・ユーザー作成 |
| `php` | PHP + 拡張機能インストール、Composerインストール、PHP-FPM起動 |
| `nginx` | Nginxインストール・設定（`nginx.conf.j2`をデプロイ）・起動 |
| `laravel` | `git@github.com:abettie/multibook-backend.git` → `/var/www/api` にclone、`.env`生成、`composer install`、`php artisan migrate` |
| `react` | `git@github.com:abettie/multibook-frontend.git` → `/var/www/frontend` にclone、`npm install`、`npm run build` |

### Nginxルーティング

- ポート 80: Reactフロントエンド（`/var/www/frontend/dist`）
- `/api/*`: ポート3000（Laravelバックエンド）へプロキシ
- ポート 3000: Laravel（PHP-FPM `/run/php-fpm/www.sock` へルーティング）

### テンプレートファイル

- `roles/laravel/templates/env.j2` — Laravel `.env` ファイル生成元
- `roles/nginx/templates/nginx.conf.j2` — Nginx設定生成元

## 前提条件

- EC2インスタンス（Amazon Linux 2023）が起動済み
- `composer install` 実行のためEC2にパブリックIPv4が一時的に付与されていること（完了後に解除）
- EC2の `~/.ssh/id_rsa` にGitHub SSH秘密鍵が配置済み
- ローカルの `~/.ssh/config` にEC2への接続設定済み
