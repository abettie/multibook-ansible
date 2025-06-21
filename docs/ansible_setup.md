# Ansibleのインストールと初期設定

## Ansibleのインストール

### Macの場合

```sh
brew install ansible
```

### Ubuntuの場合

```sh
sudo apt update
sudo apt install -y ansible
```

### Amazon Linuxの場合

```sh
sudo yum install -y ansible
```

## 初期設定

1. SSH鍵を作成し、EC2に登録してください（必要に応じて）。
2. Ansibleのインベントリファイル（例: `hosts`）を作成し、EC2のIPアドレスを記載します。

例:
```
[web]
your-ec2-ip
```

3. 必要に応じて、`ansible_user`や`ansible_ssh_private_key_file`を指定してください。

例:
```
[web]
your-ec2-ip ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

4. Ansibleのバージョン確認

```sh
ansible --version
```

5. AnsibleからEC2への接続確認

```sh
ansible -i hosts all -m ping
```
