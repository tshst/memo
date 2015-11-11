# coreosにHashicorpのTerraformを試す

いろいろと調べているとなんとなく、[前回の記事](http://tshst.hateblo.jp/entry/2015/11/02/080017)のNomadよりも    
こっちを先にやった方が良いのではないかと漠然と思ったので、触ってみる

# Terraformって何？
- 「インフラの構築、変更、バージョン管理を安全かつ効率的に行うツール」らしいです
- 何かふわっとしていて、しっくりこないのは、以下の疑問があるからかも
  - うちのチームではpuppet使っているけど、puppetと何か違いがあるの？puppetに勝る点はどこ？
  - サイトを読んでいるとProviderとしてpuppetを指定することも可能という記載もある。？共存するの？
  - どういうふうに使えば一番メリットを享受できるのかがわからない

# 悩んでいてもしょうがないので、手を動かそう
## 何をするのか
- dockerでイメージを起動するまでをやってみる
- Terraformを使ってpuppetを実行してみる

# dockerでイメージを起動するまでをやってみる

### 構成
- Mac Book Air (Manager)
- さくらVPS    (agent) 

### coreosのdockerの設定変更
- remoteから接続を受け付けられるようにする
- [coreosのマニュアル](https://coreos.com/os/docs/latest/customizing-docker.html)

- vim /etc/systemd/system/docker-tcp.socket

```
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=2375
BindIPv6Only=both
Service=docker.service

[Install]
WantedBy=sockets.target
```

- 設定反映

```
systemctl enable docker-tcp.socket
systemctl stop docker
systemctl start docker-tcp.socket
systemctl start docker
```

- 動作テスト

```
docker -H tcp://127.0.0.1:2375 ps
```

- coreosでiptables
    - 今回はテストなので、設定自体は行わないが、本番運用では設定必須かな

### terraformでイメージインストール
- trファイルの作成
[https://github.com/tshst/terraform/blob/master/docker.tf:embed:cite]

- terraform plan実行

```
Refreshing Terraform state prior to plan...


The Terraform execution plan has been generated and is shown below.
Resources are shown in alphabetical order for quick scanning. Green resources
will be created (or destroyed and then created if an existing resource
exists), yellow resources are being changed in-place, and red resources
will be destroyed.

Note: You didn't specify an "-out" parameter to save this plan, so when
"apply" is called, Terraform can't guarantee this is what will execute.

+ docker_container.foo
    bridge:           "" => "<computed>"
    gateway:          "" => "<computed>"
    image:            "" => "${docker_image.ubuntu.latest}"
    ip_address:       "" => "<computed>"
    ip_prefix_length: "" => "<computed>"
    must_run:         "" => "1"
    name:             "" => "foo"

+ docker_image.ubuntu
    latest: "" => "<computed>"
    name:   "" => "ubuntu:latest"


Plan: 2 to add, 0 to change, 0 to destroy.
```

- terraform apply実行

```
[7:37] % terraform apply
docker_image.ubuntu: Creating...
  latest: "" => "<computed>"
  name:   "" => "ubuntu:latest"
docker_image.ubuntu: Creation complete
docker_container.foo: Creating...
  bridge:           "" => "<computed>"
  gateway:          "" => "<computed>"
  image:            "" => "ca4d7b1b9a51f72ff4da652d96943f657b4898889924ac3dae5df958dba0dc4a"
  ip_address:       "" => "<computed>"
  ip_prefix_length: "" => "<computed>"
  must_run:         "" => "1"
  name:             "" => "foo"
Error applying plan:

1 error(s) occurred:

* docker_container.foo: Container a81909ca4572a0ee7547abab5c02d0d6c85cca483b70abaa96d004aa80837b7f exited after creation, error was:

Terraform does not automatically rollback in the face of errors.
Instead, your Terraform state file has been partially updated with
any resources that successfully completed. Please address the error
above and apply again to incrementally change your infrastructure.
```

- 実行前のdocker images

```
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
matsumotory/ngx-mruby   latest              6b082de49287        12 days ago         578.7 MB
redis                   latest              05babbd460f7        2 weeks ago         109.1 MB
```

- 実行後のdocker images

```
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                  latest              ca4d7b1b9a51        46 hours ago        187.9 MB
matsumotory/ngx-mruby   latest              6b082de49287        12 days ago         578.7 MB
redis                   latest              05babbd460f7        2 weeks ago         109.1 MB
```

- imageはあるみたい
- どこが問題なのか、引き続きエラーについて調べる
