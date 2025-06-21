# multibook-ansible

## 前提
- EC2インスタンスは作成済み
- EC2にSSHで接続できる
- Ansible実行端末にAnsibleがインストール済み（[Ansibleのインストールと初期設定](docs/ansible_setup.md)を参照）
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

## Ansibleのインストールと初期設定

詳しくは [docs/ansible_setup.md](docs/ansible_setup.md) を参照してください。

## EC2 Instance ConnectでAnsibleを実行する方法（aws ec2-instance-connect open-tunnel利用）

1. AWS CLI v2がインストールされていることを確認してください。
2. `hosts`ファイルの該当ホストに `ansible_ssh_common_args` を指定します。

例:
```
[web]
your-ec2-ip ansible_user=ec2-user ansible_ssh_common_args='-o ProxyCommand="aws ec2-instance-connect open-tunnel --instance-id i-xxxxxxxxxxxxxxxxx --remote-port 22"'
```
- `--instance-id` には対象EC2のインスタンスIDを指定してください。

3. Ansibleコマンド実行例:

```sh
ansible-playbook -i hosts site.yml
```

- `ansible_ssh_private_key_file` も必要に応じて指定してください。

## 実行方法

```sh
ansible-playbook -i hosts site.yml
```

## 注意
- `.env`ファイルや`laravel_env.yml`はgit管理しないでください。
- EC2のセキュリティグループで80, 3000, 22番ポートを開放してください。