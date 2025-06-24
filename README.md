# multibook-ansible

## 前提
- EC2インスタンスは作成済み
- EC2にSSHで接続できる
- Ansible実行端末にAnsibleがインストール済み（[Ansibleのインストールと初期設定](docs/ansible_setup.md)を参照）
- EC2でgit cloneできるようSSH鍵をbitbucketに設定済み

## 事前準備

1. ローカルにAnsibleをインストールしてください。

(Ubuntuの場合)
```sh
sudo apt update
sudo apt install -y ansible python3-pymysql
ansible --version
```

2. SSH鍵を作成し、公開鍵をEC2に登録してください（必要に応じて）。

3. Ansibleのインベントリファイル（hostsファイル）を作成し、以下のように記載します。

EC2 instance connectで接続する場合
```
[web]
your-ec2-instance-id ansible_user=ec2-user ansible_ssh_private_key_file=your-key-path ansible_ssh_common_args='-o ProxyCommand="aws ec2-instance-connect open-tunnel --instance-id your-ec2-instance-id"'
```
例:
```
[web]
i-07e7d58adcdc7fa9f ansible_user=ec2-user ansible_ssh_private_key_file=/home/toshi/.ssh/ec2_privatekey_2025.key ansible_ssh_common_args='-o ProxyCommand="aws ec2-instance-connect open-tunnel --instance-id i-07e7d58adcdc7fa9f"'
```

4. 次のコマンドでAnsibleからEC2への接続確認をしてください

```sh
ansible -i hosts all -m ping
```

5. `vars/laravel_env.yml` を作成し、DBやAWSキー等を記載してください（リポジトリ管理外推奨）。

```yaml
laravel_db_name: "your_db_name"
laravel_db_user: "your_db_user"
laravel_db_password: "your_db_password"
aws_access_key: "your_aws_access_key"
aws_secret_key: "your_aws_secret_key"
aws_s3_bucket: "your_s3_bucket"
```

## Ansibleコマンド実行

```sh
ansible-playbook -i hosts site.yml
```

## 注意
- `.env`ファイルや`laravel_env.yml`はgit管理しないでください。