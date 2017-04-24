# Ansible徹底入門
## よく使うモジュール
- lineinfile
  - __regexp__ の正規表現にマッチする行があれば __line__ で指定した値で置き換える
  - マッチしなかった場合は、 __line__ で指定した値を追記する
  - 正規表現はPythonの正規表現記法を採用
  - 正規表現にマッチする行を削除したい場合は、 __line__ の代わりに __state:absent__ を指定する
  - __validate__ 引数では編集済ファイルを検証するコマンドを実行できる
    - 検証がエラーとなった場合は、変更処理は中断される

## Inventory
- [...]で囲った部分にグループ名を記載する
- [...]で囲った部分は __セクション__ と呼ぶ
- [グループ名:vars]という形式でグループ変数を定義できる
- グループ変数とホスト変数で重複して変数を定義できるが、ホスト変数が優先される
- グループ変数でホスト間共通のデフォルト値を設定して、必要に応じてホスト単位で湖月にホスト変数を定義する運用がよさそう
- 注意点
  - 真偽値を値に使う場合はTrue/Falseのように大文字始まりで書く必要がある
  - リストや辞書といった複雑な構造は取り扱えない
- 1つのホストを複数のグループに定義することができる
  - ロールで分けて、リージョンで分けるとか、加えてステージングorプロダクションで分ける
- ホストを複数グループに定義する場合の注意点
  - グループ間で変数定義が重複しないようにする
    - 変数が重複した場合は、後から読み込まれた変数が使われるが、グループの読み込み順は利用者から制御できない
  - ホスト名が同じ場合は常に同一のホストとして扱われる
- Inventory用変数をYAMLファイルで定義する
  - 変数定義ファイルの命名規則はAnsible側で定められている
    - ホスト変数定義は「host_vars/ホスト名.yml」
    - グループ変数定義は「group_vars/グループ名.yml」
    - 「host_vars」「group_vars」はInventoryファイル、もしくはPlaybookファイルと同じディレクトリ内に設置する必要がある
- 複数箇所でInventory変数が定義された場合の優先順位
  1. Playbook配下の変数定義YAMLファイル->ホスト変数、グループ変数で同一名の変数がある場合の優先順位の判断
  1. Inventory配下の変数定義YAMLファイル->ホスト変数、グループ変数で同一名の変数がある場合の優先順位の判断
  1. Inventoryファイル内で定義された変数->ホスト変数、グループ変数で同一名の変数がある場合の優先順位の判断
- allという「Inventory内の全ホストが所属する特別なグループ名」に対しても変数を定義できる
  - [group_vars/all.yml]ファイルに変数を定義する
  - all.ymlの変数は他のグループ変数よりも優先順位が低く設定されている
  - all.yml < その他のグループ変数 < ホスト変数

## Dynamic Inventory
- [公式GitHubで各種スクリプトを配布している](https://github.com/ansible/ansible/tree/devel/contrib/inventory)
  - 各種IaaS(Azure, AWS EC2, OpenStack)
  - 監視システム(Nagios, Zabbix)
  - オーケストレーションツール(Serf, Consul)
- 静的なInventoryファイルかDynamic Inventoryファイルかの判定はファイルの実行権限の有無で行う
  - 実行権限がないファイルは静的Inventoryとして扱う
  - 実行権限が付いていればDynamicInventoryスクリプトとみなす
- playbook実行時は、DynamicInventoryのスクリプトファイルがあるディレクトリで実行しないとうまく動かなかった
  - ディレクトリ構成
    ```
    .
    ├── ansible.cfg
    ├── group_vars
    ├── host_vars
    │   └── vagrant-machine.yml
    ├── hosts
    ├── site.retry
    ├── site.yml
    └── tutorial
        ├── Vagrantfile
        ├── vagrant.py
        └── vagrant.py.bak
    ```

  - site.ymlのあるディレクトリでansible実行

    ```
    gaia% ansible-playbook -i tutorial/vagrant.py site.yml

    ERROR! Attempted to execute "tutorial/vagrant.py" as inventory script: Inventory script (tutorial/vagrant.py) had an execution error: b'A Vagrant environment or target machine is required to run this\ncommand. Run `vagrant init` to create a new Vagrant environment. Or,\
    nget an ID of a target machine from `vagrant global-status` to run\nthis command on. A final option is to change to a directory with a\nVagrantfile and to try again.\nTraceback (most recent call last):\n  File "/Users/tshst/OneDrive/src/github.com/tshst/develop/ansible/
    tutorial/vagrant.py", line 110, in <module>\n    ssh_config = get_ssh_config()\n  File "/Users/tshst/OneDrive/src/github.com/tshst/develop/ansible/tutorial/vagrant.py", line 72, in get_ssh_config\n    return dict((k, get_a_ssh_config(k)) for k in list_running_boxes())\n
      File "/Users/tshst/OneDrive/src/github.com/tshst/develop/ansible/tutorial/vagrant.py", line 77, in list_running_boxes\n    output = subprocess.check_output(["vagrant", "status"]).decode().split(\'\\n\')\n  File "/Users/tshst/.anyenv/envs/pyenv/versions/3.5.2/lib/pytho
    n3.5/subprocess.py", line 626, in check_output\n    **kwargs).stdout\n  File "/Users/tshst/.anyenv/envs/pyenv/versions/3.5.2/lib/python3.5/subprocess.py", line 708, in run\n    output=stdout, stderr=stderr)\nsubprocess.CalledProcessError: Command \'[\'vagrant\', \'statu
    s\']\' returned non-zero exit status 1\n'
    ```
  - vagrant.py(DynamicInventory Script)のあるディレクトリで実行
    ```
    gaia% ansible-playbook -i vagrant.py ../site.yml

    PLAY [Playbookチュートリアル] ************************************************************************************************************************************************************************************************************************************************
    *******
    
    TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************
    ok: [default]
    -略-
    PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
    default                    : ok=13   changed=1    unreachable=0    failed=0
    ```

- 複数のInventoryファイルを同時に扱う
  - __inventories__ ディレクトリを作成し、配下に静的Inventoryファイル、DynamicInventoryスクリプトを設置する
  - 以下の通り実行すると __inventories__ 配下の情報を1つのInventoryファイルとして扱うことが可能となる

    ```
    ansible-playbook -i inventories site.yml
    ```

## Playbook内でのInventory操作
- Ansible実行時点ですでに存在するホストについては、静的、動的に関わらずInventoryファイルに定義することができた
- Ansible実行時点で存在しないホストに関して、どのように対応すればよいか？
  - Playbook内でInventoryを操作するモジュール __add_host__ を使用する
- [ ]  __検証環境ができてから動作検証する__  
- __group_by__ モジュールでは既存のホストを新たな基準でグループ分けできるようになる
  - __group_by__ の引数は、 __key__ 1つだけ
  - __key__ にスペースが含まれるときは自動で __-__ に変換される

## 変数
- Ansibleの変数名に使える文字は __半角英数字(a-z, A-Z, 0-9)とアンダースコア(_)__
- 先頭に __数字__ を使うことはできない
- 変数名にハイフン(-)やスペースを含めたり、日本語を使ったりすることもできない
- 変数名はAnsible内部でPythonのアトリビュートとして処理されるため、Pythonの予約語 __and, from, to__ 等は使えない

### 変数の定義方法
1. モジュールが自動取得(Facts)
1. Inventory(グループ/ホスト)レベルでの定義
1. Ansible実行時に定義
1. Playbook内で定義

- ansibleやansible-playbookコマンドには実行時に変数定義を渡すためのオプション __--extra-vars (省略形 -e)__ がある
- 以下のように __key=value__ 形式で変数を渡すことが可能

```
ansible-playbook -i hosts -e "nginx_version=1.10.2 nginx_user=nginx" site.yml
```

- 上記のように __=__ で変数を定義すると値がすべて文字列として扱われるため、他の型を使いたい場合は、以下のようにJSON形式で定義する

```
ansible-playbook -i hosts -e '{"nginx_port": 8080}' site.yml
```

- インラインで変数の数が増えたり複雑な値を渡したい場合は、変数の頭に __@__ をつけてYAML形式の変数定義ファイルを読み込ませることも可能

```
ansible-playbook -i hosts -e '@extra-vars.yml' site.yml
```

- エクストラ変数定義オプションで指定した変数は他のどこで定義した変数よりも優先度が高いものになる

### Playbook内での変数定義
- __vars__ ディレクティブによる直接定義

```
---
- name: ポート番号を使うPlay
  hosts: all
  vars:
    nginx_http_port: 80
    mysql_port: 3306
```

- __vars__ ディレクティブで定義した変数情報は __そのPlay,Taskの中でのみ有効__
- varsに類似の __vars_files__ ディレクティブはPlayにしか定義できない

```
vars_files:
  - some_vars.yml
  - another_vars.yml
```

### 変数の優先順位

1. エクストラ変数(コマンド実行時に-eで渡すもの)
1. タスク変数(指定したTask内でのみ有効)
1. ブロック変数(タスクをまとめたBlock内でのみ有効)
1. Roleのvars内変数定義ファイル/ include_varsモジュールで読み込んだ変数定義ファイル
1. Play内vars_filesディレクティブで読み込んだ変数定義ファイル
1. Play内vars_promptディレクティブを使って入力した変数
1. Playのvarsで定義した変数
1. set_factで指定した値
1. registerで保存した値
1. HostのFact情報(setupが収集する情報)
1. Playbookディレクトリ上host_vars内ホスト変数ファイル
1. Playbookディレクトリ上group_vars内グループ変数ファイル
1. Inventoryディレクトリ上host_vars内ホスト変数ファイル
1. Inventoryディレクトリ上group_vars内グループ変数ファイル
1. Inventoryファイル(Dynamic含む)内で定義された変数
1. Roleのdefaults内のデフォルト変数定義ファイル

### Jinja2による変数の展開
- {{と}}で変数を括るとAnsible実行時に値が展開される
- YAMLの文法上の制約から、文字列の先頭もしくは終わりに変数を埋め込む場合は、次のようにダブルクォートで括る必要がある
- 構造化された変数へのアクセスは、 __admin_user.name__ のようにドットで繋いでアクセスできるが、アクセスすべき属性の名前自体が変数になっている場合だとドットでのアクセスができないため、 __admin_user['name']__ のようにしてアクセスする必要がある

### Jinja2の様々な機能
- ifによる条件分岐
  - 制御構造を表す際は{%と%}で括る
  - 制御構造の最後には{% end... %}が必要
    - ...の部分は制御名に対応し、if文の終わりであれば{% endif %}になる

    ```
    {% if ansible_os_family == "RedHat" %}
    "このマシンのディストリビューションはRedHatです"
    {% elif ansible_os_family == "Debian" %}
    "このマシンのディストリビューションはDebianです"
    {% else %}
    "このマシンはRedHatでもDebianでもありません"
    {% endif %}
    ```
- forによる繰り返し
  ```
  {% for user in admin_users %}
  ユーザ名 {{ user.name }} の UIDは {{ user.uid }} です
  {% endfor %}
  ```

- for文の中でif文を使う
 ```
 {% for user in admin_user if user.name == 'taro' %}
 ```

- 繰り返し中でのみ使える特殊変数loop
  - loop.index
    - 1から始まる現在の繰り返し回数
    - 0始まりにするにはindex0と指定する
  - loop.revindex
    - 後ろから数えた繰り返し回数
  - loop.first
    - 繰り返しの最初の要素である場合にTrue
  - loop.last
    - 繰り返しの最後の要素である場合にTrue

### フィルターによる値の加工
- {{ ~ }}で括った変数評価の中で値を加工するための機能
- アルファベット文字列を小文字に変換する __lower__ フィルター

```
{{ 'SAMPLE STRING' | lower }}
```

- フィルターはパイプ(|)を複数個つなげることも可能
- フィルターはFilter PluginというAnsible機能拡張プラグインのひとつとして実装されているため自作可能

## Playbook中のタスク実行制御とディレクティブ
### タスク実行結果を変数として保存
- __register__ ディレクティブ
- __register__ ディレクティブに変数名を指定するだけでその変数にタスク実行結果が代入される

### タスクの条件付き実行
- 指定した条件を満たした場合にのみタスクを実行する __when__ ディレクティブ
- Playbookに新たな変数を発行するためには、 __set_fact__ モジュールを使う
- __when__ 中にはJinja2テンプレートで書いた条件式しか入れられないため、Jinja2の式であることを宣言する必要がない
- when系ディレクティブの __changed_when__ と __failed_when__ も同様
- __skipping__ というタスク実行ステータスは「タスクを実行せずにスキップした」という意味

### タスク実行結果ステータスを条件にする
- Ansibleにはregisterした実行結果のステータスを簡単に判定できる次の __フィルタ__ が用意されている
  - __succeeded/success__
    - タスクが成功した(変更の有無は問わない)
  - __changed/change__
    - タスクが変更を伴う処理を行った
  - __failed/failure__
    - タスクが失敗した
  - __skipped/skip__
    - タスクがスキップされた

### __changed_when__ と __failed_when__
- どちらもタスクの実行結果をステータスを書き換えるもので、それぞれの名前のとおり __changed__ と __failed__ の条件を上書きできる
- 通常はこのようなステータスの書き換えを行う必要はない
- これらのディレクティブが必要となるのは __command__ 系モジュールを使う際に限られる
- タスクの実行結果を用いた判定をする場合には、後続のタスクから実行結果を使わない場合でも常に __register__ を使って実行結果を保存する必要がある

### __with_items__ を使ったタスクのループ
- __with_items__ ディレクティブにループしたいリストを渡すと、 __item__ という名前の変数にループ内の各要素が入る
- __user__ モジュール
  - ユーザが存在するかチェックを行う
- 通常のプログラミング言語におけるループと異なる動きとして、複数のリストをまとめてループできる
- __item__ 以外の変数名の使用は、 __loop_control__ ディレクティブの __loop_var__ に指定することで可能

```
yum:
  name: "{{ package_name }}"
with_items:
  - "{{ development_packages }}"
  - "{{ openssl_packages }}"
loop_control:
  loop_var: package_name
```

### その他のループ
- Ansibleには __with_items__ 以外にも __with_*__ という名前がついたループ方式がたくさんある
  - __with_sequence__
    - 数字で範囲(1-10までなど)を指定してループ
  - __with_dict__
    - リストではなく辞書式のデータをループ
  - __with_fileglob__
    - 存在するファイルをシェルのglob形式(*でのワイルドカード指定)を追加って辿れる
  - __with_nested__
    - 入れ子構造のループを作れる


## マジック変数
- __hostvars__
  - Inventory内に含まれる全ホストの変数情報を含んでいる
  - __hostvars__ を参照すればPlaybookのどこからでもホストの変数を参照可能
  - 変数へのアクセス方法

  ```
  {{ hostvars['ホスト名'] }}
  ``` 

- __groups__ __group_names__
  - __groups__
    - Inventory内に存在するグループとそのグループに所属するホスト一覧
  - __group_names__
    - ホストが所属しているグループのリスト

- その他のマジック変数
  - Ansible実行マシン内の情報
    - __playbook_dir__ : Plyabookファイルがあるディレクトリのパス
    - __inventory_dir__ : Inventoryファイルがあるディレクトリのパス
    - __inventory_file__ : Inventoryファイル自身のパス(Inventoryをディレクトリで指定した場合はディレクトリ)
    - __ansible_playbook_python__ : Ansible実行マシン中のansible-playbookコマンドを実行しているPythonのパス(ansible2.3から利用可能)
    - __ansible_check_mode__ : ansible-playbookコマンドに--checkオプションを付けてチェcっくモードで動かしている場合は、Trueとなり、そうでない場合はFalseとなる
  - Inventory関連の情報
    - __inventory_hosetname__ : Inventory上で付けたホスト名、捜査対象マシンの実ホスト名(hostnameコマンドの実行結果)が入り、 __ansible_hostname__ とは異なるので注意
    - __ansible_play_hosts__ : Play内でデプロイ対象になっているホスト名のリスト
    - __ansible_play_batch__ : ローリングアップデートを行っている場合の現在デプロイ中であるホスト名のリストで、通常デプロイ時は __ansible_play_hosts__ と同じ値になる

## Roleを作る
- Roleとは
  - Playbookの中身を機能単位で分割し、共通部品として管理/再利用するための仕組み
  - Role化する単位には制約はなく、一連のまとまった工程であればなんでもRole化することができるが、汎用性と再利用性を考えると以下のポイントがRole化する単位の大まかな目安になりそう
    - 複数のシステムで用いられる
    - 独立して使いまわせる閉じた機能である
- ansible-galaxy
  - Role管理用コマンド
  ```
  ansible-galaxy init --init-path="roles" nginx
  ```

  - コマンド実行後のディレクトリ構成
  ```
  ├── roles
  │   └── nginx
  │       ├── README.md
  │       ├── defaults
  │       │   └── main.yml
  │       ├── handlers
  │       │   └── main.yml
  │       ├── meta
  │       │   └── main.yml
  │       ├── tasks
  │       │   └── main.yml
  │       ├── tests
  │       │   ├── inventory
  │       │   └── test.yml
  │       └── vars
  │           └── main.yml
  ```

  - 各ディレクトリの役割
    - defaults
      - Role内で使う変数のデフォルト値定義
      - 最も優先順位が低い(書き換える前提の)変数定義
    - files
      - 捜査対象ホストにコピーするファイル配置用ディレクトリ
    - handlers
      - Handlerとして使用するタスクの定義
    - tasks
      - Roleで実行するタスクの定義
    - templates
      - 対象ホストに展開するJinja2テンプレートを配置
    - tests
      - RoleをテストするためのPlaybook(test.yml)とInventory定義(inventory)を配置
    - vars
      - Role内で使う変数定義
      - Inventory変数ではかきかえられない(個別に書き換えることを想定しない)変数定義

- グローバルRoleの配置
  - RoleをPlaybook配下ではなく、マシン上のどこからでも使えるグローバルなものにしたい場合は、 __/etc/ansible/roles__ 内に配置すればOK

### Roleの実行
- Playの中でRoleの呼び出しとタスク定義を共存させたい場合は、 __pre_tasks__ __post_tasks__ というディレクティブを使って、Role実行前後のタスクを定義できる
- __pre_tasks__
  - Role実行開始前に実行されるタスク
- __post_tasks__
  - 全Role実行終了後に実行されるタスク

## Nginx起動用ポート変数化
- Nginxのデフォルト設定をベースに以下の変更を行っている
  - 先頭に __#{{ ansible_managed }}__ という行を追加
  - user (Nginx起動ユーザ/グループ設定)をnginxから __{{ nginx_user }} {{ nginx_group }}__ に変更
  - Nginx設定ファイル配置ディレクトリを /etc/nginx/conf.dから __{{ nginx_config_dir }}__ に変更
  - デフォルトポート番号を80から __{{ nginx_default_port }}__ に変更

- __{{ ansible_managed }}__ という変数は、テンプレート展開時にAnsible managedというメッセージに変換され、そのファイルがAnsibleによって設置されたものであることが一目瞭然となる
- 他の4つの変数について、先頭が __nginx__ という名前で始まる変数にしているのは、Roleを作るにあたってかなり重要なポイントのひとつである

### Role内変数はRole名を先頭に付ける
- Ansible側で決められている必須のルールではないが、Role側からどんなPlaybookから呼び出されるか、どんなRoleと組み合わせて使われるかわからないため、変数に同じ名前が付けられ、定義がバッティングする恐れがある
- Role名はユニークであるため、Role内変数にRole名を付けることで重複をさけることができる

### 変数のデフォルト値を指定
- デフォルト値を設定しておかないと、Inventory、もしくはPlaybook本体側で値を設定していない場合に、変数未定義エラーが発生する
- 確実に変数の値をPlaybook側で設定させたい場合は、デフォルト値を未設定のままにしておく方法もある
- 変数のデフォルト値の設定は、 __defaults__ 内に記載する

### 設定ファイル/テンプレート展開タスク実装
- __templates__ ディレクトリ内に入っているファイルであれば、何もディレクトリパスを指定せずとも、 __template__ モジュールの __src__ オプションに指定できる

### __validate__ によるファイル検証コマンドの実施
- nginxには設定のエラーチェックを行う機能が備わっており、 __nginx -t -c /etc/nginx/nginx.conf__ でチェックできる
- __template__ や __copy__ モジュールでは、 __validate__ オプション引数を使って、ファイルを実際に設置する前にチェックコマンドを実行できる
- __validate__ で設定されたチェックコマンドが失敗したら、 __dest__ で指定されたパスに実際にファイルがコピーされる前に、タスクが失敗させられる
- __%s__ にはPlaybook実行時にAnsibleが自動的に生成する一時ファイルのパスが入る
- __validate__ で指定したコマンドが実行されるのは、実際の配置パスにファイルが展開される前なので、直接パスを指定しても正しく動作しない

## Nginxリロード用Handlerの追加
- Handlerは遅延実行専用のタスク定義方法
- Handlerを使うことで、リロード/再起動系処理を最後にまとめて効率的に実行できる  
- Handlerの性質
  - Roleの __handlers/main.yml__ もしくはPlay内の __handlers__ ディレクティブ内で定義する
  - Handlerは通常タスクの __notify__ ディレクティブから登録できる
  - Handlerが登録されるのは __notify__ したタスクが変更系処理を行った場合のみ
  - 登録されたHandlerはPlay実行の最後に __まとめて1回だけ実行__ される