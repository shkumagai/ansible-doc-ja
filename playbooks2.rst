高度なプレイブック
==================

.. イメージ省略

ここにあるのはプレイブック言語のより高度な機能です。これらの機能のすべてを使う
必要はありませんが、これらの多くが有用であることが分かるでしょう。
これらの機能を今すぐ必要としていないのなら、自由に飛ばしてもらって結構です。
多くの人たちにとっては、 `playbook` に書かれている機能が使用している機能の90％
以上になるでしょう。

.. contents::
   :depth: 2
   :backlinks: top


タグ
````

.. versionadded:: 0.6

巨大なプレイブックの場合、構成の特定の部分を実行できると便利な場合があります。
このような理由から、プレイとタスクは"tags:"属性をサポートしています。

例::

    tasks:

        - action: yum name=$item state=installed
          with_items:
             - httpd
             - memcached
          tags:
             - packages

        - action: template src=templates/src.j2 dest=/etc/foo.conf
          tags:
             - configuration

とても長いプレイブックの"configuretion"と"packages"の部分だけを実行したい場合は
このようにできます::

    ansible-playbook example.yml --tags "configuration,packages"


プレイブックをインクルードするプレイブック
``````````````````````````````````````````

.. versionadded:: 0.6

インクルードファイルの概念をさらに高めるために、プレイブックファイルは他の
プレイブックファイルをインクルードできます。
あなたは"webservers.yml"にすべてのウェブサーバの動作を、"dbservers.yml"に
すべてのデータベースサーバを定義しているとしましょう。このように、あなたの
システム全体を再構築するための"site.yml"を作ることができます::

    ----
    - include: playbooks/webservers.yml
    - include: playbooks/dbservers.yml

この概念はタグと合わせて、まさに実行したいプレイや欲しいプレイの中のパーツを
サッと選択するために、非常に良い働きをします。


コマンドの失敗を無視する
````````````````````````

.. versionadded:: 0.6

通常のプレイブックでは、失敗があったホストではそれ以降の手順の実行がストップ
します。しかし、実行を継続した場合があります。それを行うには、次のように
タスクを記述します::

    - name: this will not be counted as a failure
      action: command /bin/false
      ignore_errors: yes


複雑な変数データにアクセスする
``````````````````````````````

ネットワーク情報のように、提供されるfactの一部は入れ子データ構造のとして
利用できます。それらにアクセスするには、単純に'$foo'では不十分ですが、それでも
やり方は簡単です。これはIPアドレスを取得する方法です::

    ${ansible_eth0.ipv4.address}

また、その要素である配列変数にアクセスすることもできます::

    ${somelist[0]}

そして、配列とハッシュリファレンスの構文を混在させることができます。

テンプレートでは、単純なアクセス形態をいまだ保持していますが、必要であれば
よりPythonネイティブなやり方でJinja2からアクセスできます::

    {{ ansible_eth0["ipv4"]["address"] }}


マジック変数と他ホストの情報にアクセスする方法
``````````````````````````````````````````````

自身で定義をしていなくても、ansibleは自動的にいくつかの変数を提供します。
これらの中で最も重要なのは 'hostvars'、'group_names'、そして'groups'です。

hostvars はそのホストについて収集されたfactを含めて、他のホストの変数について
問い合わせることができます。この時点で、もしまだプレイブックやプレイブックの
セット内の、いずれのプレイでもそのホストに対してやり取りをしていない場合、
変数の取得はできますが、factを見ることはできません。

データベースサーバが別ノードのfactや別ノードにアサインされたインベントリ変数を
使いたい場合、テンプレートやaction行の中でも簡単につかうことができます::

    ${hostvars.hostname.factname}

プレイブックの中では、ホスト名にダッシュやピリオドが含まれている場合には、注意
が必要です。このようにエスケープしてください::

    ${hostvars.{test.example.com}.ansible_distribution}

Jinja2テンプレートでは、このようにも記述できます::

    {{ hostvars['test.example.com']['ansible_distribution'] }}

さらに、 *group_names* は現在のホストを含むすべてのグループ名のリスト(配列)です。
これはテンプレートの中でJinja2の構文を使って、ホストのグループ(やロール)メンバー
シップの変化に対応したテンプレートソースファイルを作成するのに使えます::

   {% if 'webserver' in group_names %}
      # some part of a configuration file that only applies to webservers
   {% endif %}

*groups* はインベントリに含まれる、すべてのグループ(およびホスト)のリストです。
これはグループ毎のすべてのホストを列挙するのに使えます

たとえば::

   {% for host in groups['app_servers'] %}
      # something that applies to all app servers.
   {% endfor %}

よく使われるイディオムはグループを歩いてグループ内のすべてのIPアドレスを検索する
ものです::

   {% for host in groups['app_servers'] %}
      {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}
   {% endfor %}

これを使った例として、すべてのアプリケーションサーバにフロンドエンドのプロキシ
サーバの向き先を含めたり、正しいファイアウォールルールの設定をサーバ間で設定
させたり、ということができます。

もう少しだけ、他にも'magic'変数が用意されています... 多くはありません。

さらに、 *inventory_hostname* は、ホスト名としてansibleのインベントリホスト
ファイルに設定された名前です。これは発見したホスト名 `ansible_hostname` に
依存したくない場合や、その他の不可解な理由がある場合に便利です。
もし長いFQDNを使っている場合は、 *inventory_hostname_short* には、最初のピリオド
までの部分を含み、残りのドメインは含みません。

あなたが必要だと思わない限り、これらの事は気にする必要はありません。
使うときに分かるでしょう。

あと利用可能なものとして、 *inventory_dir* はansibleのインベントリホストファイル
を保持しているディレクトリのパス名です。


変数ファイルの分割
``````````````````

ソースコード管理下にあなたのプレイブックを保存するのは素晴らしいアイデアだけど、
特定の重要な変数をプライベートに保ちつつ、プレイブックのソースは公開したいと思う
かも知れません。同様に、主となるプレイブックとは切り離して、特定の情報を別の
ファイルに保存したいこともあるでしょう。

これらは外部変数やファイルを使うことで、このようにできます::

    ---
    - hosts: all
      user: root
      vars:
        favcolor: blue
      vars_files:
        - /vars/external_vars.yml
      tasks:
      - name: this is just a placeholder
        action: command /bin/echo foo

これはプレイブックのソースを公開するときに、その他のものと一緒に慎重に扱うべき
データを公開してしまうリスクを取り除きます。

個々の変数ファイルの内容は、このように単純なYAML辞書です::

    ---
    # in the above example, this would be vars/external_vars.yml
    somevar: somevalue
    password: magic

.. note::
   同じようにしてホスト毎、グループ毎の変数をよく似たファイルに保存することも
   できます。これについては :ref:`patterns` で触れています。


慎重に扱うべきデータを指示させる
````````````````````````````````

ユーザに特定の入力を要求したい場合、似たような名前の'vars_prompt'セクションが
使えます。これはセキュリティを高める用途があり、例えば、すべてのソフトウェアの
リリースに同じプレイブックを使い、配信するスクリプトの中の特定のリリース
バージョンは入力を求めるようにすることができます::

    ---
    - hosts: all
      user: root
      vars:
        from: "camelot"
      vars_prompt:
        name: "what is your name?"
        quest: "what is your quest?"
        favcolor: "what is your favorite color?"

これら両方のアイテムの完全なサンプルは、github の examples/playbooks ディレクトリ
にあります。

vars_prompt の代わり形態は、ユーザからの入力を隠すことができ、他のオプションも
サポートしますが、そうでなければ同等に動作します::

   vars_prompt:
     - name: "some_password"
       prompt: "Enter password"
       private: yes
     - name: "release_version"
       prompt: "Product release version"
       private: no

`Passlib <http://pythonhosted.org/passlib/>`_ がインストールされている場合、
vars_promptは入力されたデータを暗号化できるので、例えばuserモジュールを使って
パスワードを定義することができます::

   vars_prompt:
     - name: "my_password2"
       prompt: "Enter password2"
       private: yes
       encrypt: "md5_crypt"
       confirm: yes
       salt_size: 7

'Passlib'でサポートされている暗号化スキームが使えます

- *des_crypt* - DES Crypt
- *bsdi_crypt* - BSDi Crypt
- *bigcrypt* - BigCrypt
- *crypt16* - Crypt16
- *md5_crypt* - MD5 Crypt
- *bcrypt* - BCrypt
- *sha1_crypt* - SHA-1 Crypt
- *sun_md5_crypt* - Sun MD5 Crypt
- *sha256_crypt* - SHA-256 Crypt
- *sha512_crypt* - SHA-512 Crypt
- *apr_md5_crypt* - Apache’s MD5-Crypt variant
- *phpass* - PHPass’ Portable Hash
- *pbkdf2_digest* - Generic PBKDF2 Hashes
- *cta_pbkdf2_sha1* - Cryptacular’s PBKDF2 hash
- *dlitz_pbkdf2_sha1* - Dwayne Litzenberger’s PBKDF2 hash
- *scram* - SCRAM Hash
- *bsd_nthash* - FreeBSD’s MCF-compatible nthash encoding

しかし、受け入れられるパラメータは'salt'と'salt_size'のみです。独自のソルトを
使う場合は'salt'を、自動的に生成されたものを利用する場合には'salt_size'を
使います。何も指定されていない場合は、サイズ 8 のソルトが生成されます。


コマンドライン上から変数を渡す
``````````````````````````````

`vars_prompt` と `vars_files` に加えて、ansibleのコマンドラインから変数を渡す
ことができます。デプロイするアプリケーションのバージョンを渡せるようにした、
汎用的なリリースプレイブックを書くような場合に、これは特に便利です::

    ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"

これはまた、プレイブックにホストグループやユーザをまたはその他のものを設定する
ような場合にも便利です

例::

    -----
    - user: $user
      hosts: $hosts
      tasks:
         - ...

    ansible-playbook release.yml --extra-vars "hosts=vipers user=starbuck"


条件付き実行
````````````

時に、特定のホストで、特定の手順をスキップしたくなることがあるでしょう。
これは、オペレーティングシステムが特定のバージョンの場合には、あるパッケージを
インストールしないというような単純なものかも知れないし、ファイルシステムが一杯に
なっている時に何かをクリーンアップ手順を実行するものかも知れません。

ansibleでは `only_if` 句を使うと、これを簡単に行えます。これは実際にはPythonの式です。
慌てる必要はありません -- 実際、かなり簡単です::

    vars:
      favcolor: blue
      is_favcolor_blue: "'$favcolor' == 'blue'"
      is_centos: "'$facter_operatingsystem' == 'CentOS'"

    tasks:
      - name: "shutdown if my favorite color is blue"
        action: command /sbin/shutdown -t now
        only_if: '$is_favcolor_blue'


その多くをsetupモジュールが提供する、ansibleから湧き出る変数はここで使えますし、
`facter` や `ohai` などのツールからの変数も、インストールされていれば使えます。
念のためですが、これらの変数はプレフィックスが付きます。
なので `$operatingsystem` ではなく `$facter_operationsystem` です。
ansibleの組み込み変数はプレフィックス `ansible_` が付きます。

only_if 式は実際には小さな小さなPythonの断片なので、変数はクォートし、評価結果が
`True` か `False` になるように気をつけてください。プレイやプレイブックの間で
再利用し易くするには、条件式をすべて'vars'で定義するの代わりに'vars_files'を
使うことをおすすめします。

ここでは'os.path.exists'のように、生のチェックはできませんので、しないでください。

もし必要なら、自分用のfactを提供することもできます。これは :doc:`moduledev` で
触れています。それを実行するには、カスタムのfact収集モジュールをタスクリストの
先頭で呼び出させるだけです。そうすると変数が返り、それ以降のタスクでアクセス
できるでしょう::

    tasks:
        - name: gather site specific fact data
          action: site_facts
        - action: command echo ${my_custom_fact_can_be_used_now}

only_if を使った便利なコツの一つは、最後に実行したコマンドの変更された結果から
キーを取得するやりかたです。例としては::

    tasks:
        - action: template src=/templates/foo.j2 dest=/etc/foo.conf
          register: last_result
        - action: command echo 'the file has changed'
          only_if: '${last_result.changed}'

$last_resultはregisterディレクティブに設定された変数です。これはansible0.8以降を
想定しています。

ansible0.8では、変数が定義済みか否かを確認するショートカットがいくつか使えます::

    tasks:
        - action: command echo hi
          only_if: is_set('$some_variable')

同じように動作する'is_unset'があります。関数内の引数のクォートは必須です。

`only_if` と `with_items` を組み合わせる場合、 `only_if` の文は各項目毎に別々に
処理されることに注意してください。
これは仕様によるものです::

    tasks:
        - action: command echo $item
          with_item: [ 0, 2, 4, 6, 8, 10 ]
          only_if: "$item > 5"

`only_if` は上級ユーザにとってはかなり良いオプションですが、私たちが望んだ以上に
中身を見せてしまっているので、もっとにいいやり方があるはずです。
1.0では、'when'を追加しました。これはこの複雑なレベルを隠蔽するものであり、
`only_if` のシンタックスシュガーのようなものです。詳しくは次をご覧ください。


条件付き実行 (簡易版)
`````````````````````

.. versionadded: 0.8

ansible 0.9で、私たちは only_if は文法的に少し複雑なこと、そしてユーザに対して
Pythonの部分を露呈させ過ぎたことに気づきました。その結果、 'when' キーワードの
セットが追加されました。'when'文はクォートしたり、特定の型にキャストする必要は
ありませんが、使用されるすべての引数を半角スペースで区切る必要があります。
ほとんどの場合、ユーザは'when'を利用できますが、より複雑なケースでは依然として
'only_if'が必要とされるでしょう。

これは'when'の様々な使い方の例です。同一タスク内で、'when'は'onli_if'と互換性
はありません::

    - name: "do this if my favcolor is blue, and my dog is named fido"
      action: shell /bin/false
      when_string: $favcolor == 'blue' and $dog == 'fido'

    - name: "do this if my favcolor is not blue, and my dog is named fido"
      action: shell /bin/true
      when_string: $favcolor != 'blue' and $dog == 'fido'

    - name: "do this if my SSN is over 9000"
      action: shell /bin/true
      when_integer: $ssn > 9000

    - name: "do this if I have one of these SSNs"
      action: shell /bin/true
      when_integer:  $ssn in [ 8675309, 8675310, 8675311 ]

    - name: "do this if a variable named hippo is NOT defined"
      action: shell /bin/true
      when_unset: $hippo

    - name: "do this if a variable named hippo is defined"
      action: shell /bin/true
      when_set: $hippo

    - name: "do this if a variable named hippo is true"
      action: shell /bin/true
      when_boolean: $hippo

when_boolean は、'True'や'true'のような文字列、非ゼロの数などのように、真と考え
られる変数を探します。

.. versionadded: 1.0

1.0では、when_changedとwhen_failedも追加し、ユーザは先に登録されたタスクの状態を
元にタスクを実行できます。例としては::

    - name: "register a task that might fail"
      action: shell /bin/false
      register: result
      ignore_errors: True

    - name: "do this if the registered task failed"
      action: shell /bin/true
      when_failed: $result

    - name: "register a task that might change"
      action: yum pkg=httpd state=latest
      register: result

    - name: "do this if the registered task changed"
      action: shell /bin/true
      when_changed: $result

いくつかのタスクが同じ条件文を共有している場合は、タスクのインクルード文に条件を
付与できます。これはプレイブックのインクルードでは機能せず、タスクのインクルード
だけ機能することに注意してください。すべてのタスクは評価されますが、条件文は
それぞれすべてのタスクに適用されます::

    - include: tasks/sometasks.yml
      when_string: "'reticulating splines' in $output"


条件付きインポート
``````````````````

ループ
``````

参照プラグイン - 外部データにアクセスする
`````````````````````````````````````````

環境設定 (とプロキシ経由での動作)
`````````````````````````````````

ファイルから値を取得する
````````````````````````

変数によってファイルとテンプレートを選択する
````````````````````````````````````````````

非同期アクションとポーリング
````````````````````````````

ローカルプレイブック
````````````````````

fact をオフにする
`````````````````

Pullモードプレイブック
``````````````````````

変数を登録する
``````````````

ローリングアップデート
``````````````````````

デリゲーション (移譲)
`````````````````````

Fireballモード
``````````````

変数の優先順位を理解する
````````````````````````

チェックモード ("Dry Run") --check
``````````````````````````````````

--diff で差分を表示する
```````````````````````

辞書＆入れ子(複合)引数
``````````````````````

スタイルのポイント
``````````````````

ansibleプレイブックは色付けされています。これが好きでない場合、環境変数
ANSIBLE_NOCOLOR=1 を設定してください。

ansibleはcowsayがインストールされているとより素晴らしい出力ができるので
このパッケージのインストールを推奨しています。


.. seealso::

   :doc:`YAMLSyntax`
       YAML の構文について学ぶ
   :doc:`playbooks`
       Review the basic playbook features
   :doc:`bestpractices`
       Various tips about playbooks in the real world
   :doc:`modules`
       Learn about available modules
   :doc:`moduledev`
       Learn how to extend Ansible by writing your own modules
   :doc:`patterns`
       Learn about how to select hosts
   `Github examples directory <https://github.com/ansible/ansible/tree/devel/examples/playbooks>`_
       Complete playbook files from the github project source
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
