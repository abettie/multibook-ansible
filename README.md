# multibook-ansible

## 前提
- EC2インスタンスは作成済み
- EC2にSSHで接続できる
- Ansible実行端末にAnsibleがインストール済み
- EC2にgit cloneできるようSSH鍵を設定済み

## 事前準備
1. `vars/laravel_env.yml` を作成し、DBやAWSキー等を記載（リポジトリ管理外推奨）

```yaml
laravel_db_name: "your_db_name"
laravel_db_user: "your_db_user"
laravel_db_password: "your_db_password"
aws_access_key: "your_aws_access_key"
aws_secret_key: "your_aws_secret_key"
aws_s3_bucket: "your_s3_bucket"
```

2. EC2のIPアドレスを`hosts`ファイルに記載

```
[web]
your-ec2-ip
```

## 実行方法

```sh
ansible-playbook -i hosts site.yml
```

## 注意
- `.env`ファイルや`laravel_env.yml`はgit管理しないでください。
- EC2のセキュリティグループで80, 3000, 22番ポートを開放してください。