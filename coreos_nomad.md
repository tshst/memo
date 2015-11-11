# coreosでHashiCorpのNomadを試す


## Nomadとは
- マシンや実行中アプリケーションのクラスタ管理のためのソフト
- 興味本位で触っているだけなので、技術的なところはもう少し触ってからということで
- coreosのfleatとの違いなどはおいおい

## 環境の準備
- さくらvpsにインストールしたcoreosの環境を使用する
- [Nomad](https://nomadproject.io/)はGo製なので、ファイルを落として解凍するだけで使用できる

```
wget https://releases.hashicorp.com/nomad/0.1.2/nomad_0.1.2_linux_amd64.zip
unzip nomad_0.1.2_linux_amd64.zip
```

## agentの起動

- [Getting Started](https://nomadproject.io/intro/getting-started/running.html)を参考に進めていく
- テスト用途なので、-devオプションでagentの起動

```
$ sudo ./nomad agent -dev
```

- 起動しているか確認

```
$ ./nomad node-status
ID                                    DC   Name      Class   Drain  Status
c2d5a0b7-9df0-a579-03b1-f48797191f49  dc1  tshvm001  <none>  false  ready
```

- 停止はCtrl+C

- DC名を変更してみる

```
$ sudo ./nomad agent -dev -dc sakura-ishikari
```

```
$ ./nomad node-status
ID                                    DC               Name      Class   Drain  Status
224d9fdc-b1c3-eff8-4dbc-c73c357756db  sakura-ishikari  tshvm001  <none>  false  ready
```


## jobを実行してみる

- jobの作成

```
./nomad init
```

- DC名変更したので、example.nomadのdc名を「sakura-ishikari」に修正

- jobの実行

```
$ ./nomad run example.nomad
```

- 実行時のログ

```
    2015/11/01 22:25:21 [DEBUG] driver.docker: docker pull redis:latest succeeded
    2015/11/01 22:25:21 [DEBUG] driver.docker: using image 05babbd460f79692240213d98c6e3d8aeb7d3e391f94a95ac6866e5ab207c8fd
    2015/11/01 22:25:21 [INFO] driver.docker: identified image redis:latest as 05babbd460f79692240213d98c6e3d8aeb7d3e391f94a95ac6866e5ab207c8fd
    2015/11/01 22:25:21 [DEBUG] driver.docker: using 268435456 bytes memory for redis:latest
    2015/11/01 22:25:21 [DEBUG] driver.docker: using 500 cpu shares for redis:latest
    2015/11/01 22:25:21 [WARN] driver.docker: no mode specified for networking, defaulting to bridge
    2015/11/01 22:25:21 [DEBUG] driver.docker: using bridge as network mode
    2015/11/01 22:25:21 [DEBUG] driver.docker: allocated port 153.126.155.210:35096 -> 6379 (mapped)
    2015/11/01 22:25:24 [INFO] driver.docker: created container 39f4ad6e24b51821651734d88b3a4fad3ac223e14eb2a7b441f78143b264a9b9
    2015/11/01 22:25:25 [INFO] driver.docker: started container 39f4ad6e24b51821651734d88b3a4fad3ac223e14eb2a7b441f78143b264a9b9
```

- jobのstatus

```
$ ./nomad status example
ID          = example
Name        = example
Type        = service
Priority    = 50
Datacenters = sakura-ishikari
Status      = <none>

==> Evaluations
ID                                    Priority  TriggeredBy   Status
1ffa48a9-b6bd-f368-4995-78f72bf148a9  50        job-register  complete
6837350e-ed60-09a3-8c0c-d3d373c1de4b  50        job-register  complete
618fb2ed-af65-ff3c-8b02-3f9d03275f1d  50        job-register  complete

==> Allocations
ID                                    EvalID                                NodeID                                TaskGroup   Desired  Status
84ca6861-8c74-3a02-dac0-811173dd9110  1ffa48a9-b6bd-f368-4995-78f72bf148a9  224d9fdc-b1c3-eff8-4dbc-c73c357756db  cache       run      pending
```

- docker psの結果

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                             NAMES
39f4ad6e24b5        redis:latest        "/entrypoint.sh redi   16 minutes ago      Up 16 minutes       153.126.155.210:35096->6379/tcp   kickass_engelbart
```

起動している！    
これだけでもなんとなく便利な感じがしてくる    
クラスタ組んでもっと触ってみよう
