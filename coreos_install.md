# さくらのVPSにcoreosをインストール
## coreosのインストール
### 1. イメージのダウンロード
- [このページ](https://coreos.com/os/docs/latest/booting-with-iso.html)からインストールしたいイメージをローカルにダウンロードする
  - 私は[stable](http://stable.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso)にしました

### 2. sftpでさくらのVPSにuploadする
  - 2-1.「ISOイメージインストール」を選択

      [https://gyazo.com/a3ed17a68c499e8852c01cd193818d08:embed:cite]

  - 2-2. sftpのアカウントを発行し、ファイルをアップロードする

```
  sftp アカウント@ホスト
  sftp> cd iso
  sftp> put coreos_production_iso_image.iso
```

### 3. coreosのboot
  - 3-1. 「ISOイメージインストール」を選択後、下の方にスクロールすると「設定内容を確認する」ボタンがあるので、クリックする

      [https://gyazo.com/c936a0cf6ce10d5e4f8132228433d757:embed:cite]

  - 3-2. 「インストールを実行する」ボタンをクリックしてbootする

      [https://gyazo.com/37f4d0c8868e3e44e8a77e14326cfd56:embed:cite]


### 4. 初期設定
  - 4-1. コンソール接続を行い、インストールを行うための設定を行う

      [https://gyazo.com/9e0a99465463fd34400122ebab66c63d:embed:cite]

  - 4-2. 起動後のコンソール画面

      [https://gyazo.com/01ffb836d995f7d03193a58fc6ca5318:embed:cite]

  - 4-3. IPアドレス/デフォルトゲートウェイ/resolvの設定を行う
    - 設定するIPアドレスの情報はさくらVPSの管理画面の「IPv4」の箇所に記載されている

```
sudo ifconfig eth0 xxx.xxx.xxx.xxx netmask xxx.xxx.xxx.xxx
sudo route add default gw xxx.xxx.xxx.xxx
sudo vi /etc/resolv.conf
```

  - 4-4. ターミナルから接続を行う

```
ssh core@xxx.xxx.xxx.xxx
```

### 5. coreosインストール
- 5-1. cloud-config.ymlファイルの作成

```
vim cloud-config.yml
```

```
#cloud-config

hostname: tshvm001

write_files:
  - path: /etc/systemd/network/static.network
    permissions: 0644
    content: |
      [Match]
      Name=eth0

      [Network]
      Address=xxx.xxx.xxx.xxx
      Gateway=xxx.xxx.xxx.xxx
      DNS=xxx.xxx.xxx.xxx
      DNS=xxx.xxx.xxx.xxx

users:
  - name: xxx
  - passwd: `openssl passwd -l`
  - groups:
    - sudo
    - docker
  ssh_authorized_keys:
    - ssh-rsa xxxxx
```

- 5-2. coreosインストール

```
## fdiskでインストール対象のディスクを確認
sudo fdisk -l

## インストール
sudo coreos-install -d /dev/vda -c cloud-config.yml -C stable
```

- インストール完了のログ

```
2015-11-01 04:47:30 URL:http://stable.release.core-os.net/amd64-usr/766.4.0/coreos_production_image.bin.bz2 [195077671/195077671] -> "-" [1]
gpg: Signature made Wed Sep 16 23:11:07 2015 UTC using RSA key ID 1CB5FA26
gpg: key 93D2DCB4 marked as ultimately trusted
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: Good signature from "CoreOS Buildbot (Offical Builds) <buildbot@coreos.com>" [ultimate]
Installing cloud-config...
Success! CoreOS stable 766.4.0 is installed on /dev/vda
```

- 5-3. インストールが成功したら、強制再起動

      [https://gyazo.com/5e1c2e609d7574b955982eee740fa450:embed:cite]

  - 強制再起動後正常にIPアドレスの設定が反映されなかったので、 
    コンソールから作成したユーザでログインし、再度再起動を実施した

```
reboot
```
