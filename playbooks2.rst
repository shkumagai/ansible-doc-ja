高度なplaybook
==================

.. イメージ省略

ここにあるのはplaybook言語のより高度な機能です。これらの機能のすべてを使う
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

巨大なplaybookの場合、構成の特定の部分を実行できると便利な場合があります。
このような理由から、playとタスクは"tags:"属性をサポートしています。

例::

    tasks:

        - action: yum name={{ item }} state=installed
          with_items:
             - httpd
             - memcached
          tags:
             - packages

        - action: template src=templates/src.j2 dest=/etc/foo.conf
          tags:
             - configuration

とても長いplaybookの"configuretion"と"packages"の部分だけを実行したい場合は
このようにできます::

    ansible-playbook example.yml --tags "configuration,packages"


playbookをインクルードするplaybook
``````````````````````````````````````````

.. versionadded:: 0.6

インクルードファイルの概念をさらに高めるために、playbookファイルは他の
playbookファイルをインクルードできます。
あなたは"webservers.yml"にすべてのウェブサーバの動作を、"dbservers.yml"に
すべてのデータベースサーバを定義しているとしましょう。このように、あなたの
システム全体を再構築するための"site.yml"を作ることができます::

    ---
    - include: playbooks/webservers.yml
    - include: playbooks/dbservers.yml

この概念はタグと合わせて、まさに実行したいplayや欲しいplayの中のパーツを
サッと選択するために、非常に良い働きをします。


コマンドの失敗を無視する
````````````````````````

.. versionadded:: 0.6

通常のplaybookでは、失敗があったホストではそれ以降の手順の実行がストップ
します。しかし、実行を継続した場合があります。それを行うには、次のように
タスクを記述します::

    - name: this will not be counted as a failure
      action: command /bin/false
      ignore_errors: yes


複雑な変数データにアクセスする
``````````````````````````````

ネットワーク情報のように、提供されるfactの一部は入れ子データ構造のとして
利用できます。それらにアクセスするには、単純に {{ foo }} では不十分ですが、
それでもやり方は簡単です。これはIPアドレスを取得する方法です::

    {{ ansible_eth0["ipv4"]["address"] }}

同様に、これは配列の最初の要素にアクセスする方法です::

    {{ foo[0] }}

マジック変数と他ホストの情報にアクセスする方法
``````````````````````````````````````````````

自身で定義をしていなくても、ansibleは自動的にいくつかの変数を提供します。
これらの中で最も重要なのは 'hostvars'、'group_names'、そして'groups'です。これらの
名前は予約済みとして、ユーザは使うべきではありません。'environment'も予約済みです。

hostvars はそのホストについて収集されたfactを含めて、他のホストの変数について
問い合わせることができます。この時点で、もしまだplaybookやplaybookの
セット内の、いずれのplayでもそのホストに対してやり取りをしていない場合、
変数の取得はできますが、factを見ることはできません。

データベースサーバが別ノードのfactや別ノードにアサインされたインベントリ変数を
使いたい場合、テンプレートやaction行の中でも簡単につかうことができます::

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

ソースコード管理下にあなたのplaybookを保存するのは素晴らしいアイデアだけど、
特定の重要な変数をプライベートに保ちつつ、playbookのソースは公開したいと思う
かも知れません。同様に、主となるplaybookとは切り離して、特定の情報を別の
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

これはplaybookのソースを公開するときに、その他のものと一緒に慎重に扱うべき
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
リリースに同じplaybookを使い、配信するスクリプトの中の特定のリリース
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
汎用的なリリースplaybookを書くような場合に、これは特に便利です::

    ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"

これはまた、playbookにホストグループやユーザをまたはその他のものを設定する
ような場合にも便利です

例::

    ---
    - user: {{ user }}
      hosts: {{ hosts }}
      tasks:
         - ...

    ansible-playbook release.yml --extra-vars "hosts=vipers user=starbuck"

Ansible 1.2 以降では、このように括られたJSON形式で追加の変数を渡せます::

    --extra-vars "{'pacman':'mrs','ghosts':['inky','pinky','clyde','sue']}"

key=value の形式は明らかに簡単ですが、必要な場合にはこれがあります。


条件付き実行
````````````

(注意: このセクションでは、1.2の条件式をカバーしています。もし以前のバージョンを
使用しているなら以前のバージョンのドキュメントを参照してください。
`Ansible 1.1 Docs <http://ansible.cc/docs/released/1.1>`_ 古い条件式は1.2でも
運用が続いていますが、新しい形式の方がよりクリーンです。)

時に、特定のホストで、特定の手順をスキップしたくなることがあるでしょう。
これは、オペレーティングシステムが特定のバージョンの場合には、あるパッケージを
インストールしないというような単純なものかも知れないし、ファイルシステムが一杯に
なっている時に何かをクリーンアップ手順を実行するものかも知れません。

ansibleでは `when` 句を使うと、これを簡単に行えます。これは実際にはPythonの
式です。慌てる必要はありません -- 実際、かなり簡単です::

    tasks:
      - name: "shutdown Debian flavored systems"
        action: command /sbin/shutdown -t now
        when: ansible_os_family == "Debian"

たくさんのJinja2の "filters" がwhenステートメントでも使うことができ、そのうちの
幾つかはAnsibleが独自に提供しているものです。あるステートメントでのエラーを
無視して、成功か失敗かに基づいた条件判定で何かを実行したいとします::

    tasks:
      - action: command /bin/fail
        register: result
        ignore_errors: True
      - action: command /bin/something
        when: result|failed
      - action: command /bin/something_else
        when: result|success

備忘録として、派生変数が利用できることを見ておきましょう。こうできます::

    ansible hostname.example.com -m setup

Tip: 時には、文字列の変数を手に入れ、それを比較したいことがあるでしょう。
このようにしてできます::

    tasks:
      - shell: echo "only on Red Hat 6, derivatives, and later"
        when: ansible_os_family == "RedHat" and ansible_lsb.major_version|int >= 6

playbookやインベントリの中で定義された変数も使えます。

もし、必須の変数が設定されていない場合、Jinja2の `defined` テストを使ってスキップ
させたり失敗させることができます。例えば::

    tasks:
        - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
          when: foo is defined

        - fail: msg="Bailing out: this play requires 'bar'"
          when: bar is not defined

これは変数ファイルのインポート条件 (下記参照) との組み合わせで、特に便利です。

もし必要なら、自分用のfactを提供することもできます。これは :doc:`moduledev` で
触れています。それを実行するには、カスタムのfact収集モジュールをタスクリストの
先頭で呼び出させるだけです。そうすると変数が返り、それ以降のタスクでアクセス
できるでしょう::

    tasks:
        - name: gather site specific fact data
          action: site_facts
        - action: command echo {{ my_custom_fact_can_be_used_now }}

*when* 使った便利なコツの一つは、最後に実行したコマンドの変更された結果からキーを
取得するやりかたです。例としては::

    tasks:
        - action: template src=/templates/foo.j2 dest=/etc/foo.conf
          register: last_result
        - action: command echo 'the file has changed'
          when: last_result.changed

{{ last_result }} はregisterディレクティブによって設定される変数です。これは
Ansible 0.8以降を想定しています。

`when` と `with_items` を組み合わせる場合、 `when` の文は各項目毎に別々に処理
されることに注意してください。
これは仕様によるものです::

    tasks:
        - action: command echo {{ item }}
          with_items: [ 0, 2, 4, 6, 8, 10 ]
          when: item > 5

いくつかのタスクが同じ条件文を共有している場合は、タスクのインクルード文に条件を
付与できます。これはplaybookのインクルードでは機能せず、タスクのインクルード
だけ機能することに注意してください。すべてのタスクは評価されますが、条件文は
それぞれすべてのタスクに適用されます::

    - include: tasks/sometasks.yml
      when: "'reticulating splines' in output"

条件付きインポート
``````````````````

時には、特定の基準に基いて、１つのplaybookで違うことをやりたいことがある
でしょう。複数のプラットフォームやOSバージョンで動作するplaybookを作るのが
良い例です。

例のように、Apacheのパッケージ名はCentOSとDebianでは異なるかもしれませんが、
ansibleplaybookでは最小限の構文で簡単に処理できます::

    ---
    - hosts: all
      user: root
      vars_files:
        - "vars/common.yml"
        - [ "vars/{{ ansible_os_family }}.yml", "vars/os_defaults.yml" ]
      tasks:
      - name: make sure apache is running
        action: service name={{ apache }} state=running

.. note::
   変数 'ansible_os_family' がvars_filesに定義されているファイル名の
   リストに補完されています。

念のためですが、各YAMLファイルにはキーと値だけが含まれています::

    ---
    # for vars/CentOS.yml
    apache: httpd
    somethingelse: 42

どのように動作するでしょうか？オペレーティング・システムがCentOSであった場合、
１つ目のファイルに、ansibleは'vars/CentOS.yml'をインポートしようとし、それがもし
存在しない場合には'vars/os_default.yml'でフォローしようとします。リスト内の
ファイルが見つからない場合、エラーが発生するでしょう。
Debianの場合は'vars/os_default.yml'に行く前に、'vars/CentOS.yml'の代わりに
'vars/Debian.yml'を最初に見に行きます。かなりシンプルですね。

この条件付きインポート機能を使うには、playbookを実行する前にfacterやohaiの
インストールが必要ですが、これはもちろんこのようにしてansibleに任せてしまえます::

    # for facter
    ansible -m yum -a "pkg=facter ensure=installed"
    ansible -m yum -a "pkg=ruby-json ensure=installed"

    # for ohai
    ansible -m yum -a "pkg=ohai ensure=installed"

ansibleの設定に対するアプローチ -- 変数をタスクから分離し、醜くネストしたif文や
条件文によってplaybookが無秩序なコードになってしまうことを防ぐ、など - その
結果として、より合理的かつ検査可能構成ルールをもたらす -- は、特に意思決定の要点
の最小値を追求するものです。


ループ
``````

タイプ量を抑えるため、繰り返しのタスクは次のように短く記述できます::

    - name: add several users
      action: user name={{ item }} state=present groups=wheel
      with_items:
         - testuser1
         - testuser2

変数ファイルや'vars'セクションでYAMLリストを定義している場合、このようにも
できます::

    with_items: somelist

上記は次のように評価されます::

    - name: add user testuser1
      action: user name=testuser1 state=present groups=wheel
    - name: add user testuser2
      action: user name=testuser2 state=present groups=wheel

yumやaptのモジュールは少数のパッケージマネージャトランザクションを実行するのに
with_itemsを利用します。

'with_items'でイテレートする項目の種類は、必ずしも単純な文字列のリストである
必要はありません。もしハッシュのリストがあれば、このようにしてサブキーを参照
できます::

    - name: add several users
      action: user name={{ item.name }} state=present groups={{ item.groups }}
      with_items:
        - { name: 'testuser1', groups: 'wheel' }
        - { name: 'testuser2', groups: 'root' }


参照プラグイン - 外部データにアクセスする
`````````````````````````````````````````

.. versionadded: 0.8

さまざまな *lookupプラグイン* で、データをイテレートする方法が追加できます。
ansibleは、時間とともにより多くこれらの機能を持つでしょう。APIの節で説明されて
いるように、自分で記述できます。それぞれ通常はリストや１つ以上のパラメータを
受け取れます。

``with_fileglob`` は、単一ディレクトリ内でパターンに一致するすべてのファイルに
非再帰的にマッチします。これはこのように使えます::

    ---
    - hosts: all

      tasks:

        # first ensure our target directory exists
        - action: file dest=/etc/fooapp state=directory

        # copy each file over that matches the given pattern
        - action: copy src=$item dest=/etc/fooapp/ owner=root mode=600
          with_fileglob:
            - /playbooks/files/fooapp/*

``with_file`` は、ファイルディレクトリからデータを読み込みます::

        - action: authorized_key user=foo key=$item
          with_file:
             - /home/foo/.ssh/id_rsa.pub

.. versionadded: 0.9

新しいlookup機能の多くは0.9で追加されました。lookupプラグインは *管理する* マシンの
上で実行されることを覚えておいて下さい::

    ---
    - hosts: all

      tasks:

         - action: debug msg="{{ lookup('env', 'HOME') }} is an environment variable"

         - action: debug msg="{{ item }} is a line from the result of this command"
           with_lines:
             - cat /etc/motd

         - action: debug msg="{{ lookup('pipe', 'date') }} is the raw result of running this command"

         - action: debug msg="{{ lookup('redis_kv', 'redis://localhost:6379,somekey') ]} is value in Redis for somekey"

         - action: debug msg="{{ lookup('dnstxt', 'example.com') }} is a DNS TXT record for example.com"

         - action: debug msg="{{ lookup('template', './some_template.j2') }} is a value from evaluation of this template"

代替手段として、変数にlookupプラグインを割り当てたり、他の場所でそれらを使うこと
ができます。このマクロはそれらがタスク (またはテンプレート) で使用されるたびに、
評価されます::

    vars:
      motd_value: "{{ lookup('file', '/etc/motd') }}"

    tasks:
        - debug: msg="motd value is {{ motd_value }}"

.. versionadded: 1.0

``with_sequence`` は、昇順の数値を含むアイテムのシーケンスを生成します。開始と
終了、およびオプションでステップ値を指定できます。

引数は、キーと値のペアで指定します。渡す場合、'format'はprintfスタイルの文字列
です。

数値は10進数、16進数 (0x3f8)、または8進数(0600)が指定できます。負の数はサポート
されません。これは次のように動作します::

    ---
    - hosts: all

      tasks:

        # create groups
        - group: name=evens state=present
        - group: name=odds state=present

        # create some test users
        - user: name={{ item }} state=present groups=odds
          with_sequence: start=0 end=32 format=testuser%02x

        # create a series of directories with even numbers for some reason
        - file: dest=/var/stuff/{{ item }} state=directory
          with_sequence: start=4 end=16 stride=2

        # a simpler way to use the sequence plugin
        # create 4 groups
        - group: name=group{{ item }} state=present
          with_sequence: count=4

.. versionadded: 1.1

``with_password`` と、関連するlookupマクロはランダムに平文のパスワードを生成し、
与えられたファイルにそれを保存します。(vars_promptのような) 暗号化保存モードは
保留されています。
ファイルが既に存在する場合、単に'with_file'のように振る舞い、ファイルの内容を
取得します。ファイルパスに
"{{ inventory_hostname }}"のような変数の使い方は、ホストごとにランダムな
パスワードを設定 ('host_vars'変数でパスワード管理を簡素化) するために使えます。

生成されたパスワードは、ASCII文字の大文字と小文字、0-9の数字、記号(".,:-_") を
ランダムな組み合わせを含みます。生成されたパスワードのデフォルトの長さは30文字
です。この長さは、追加のパラメータを渡すことで変更できます::

    ---
    - hosts: all

      tasks:

        # create a mysql user with a random password:
        - mysql_user: name={{ client }}
                      password="{{ lookup('password', 'credentials/' + client + '/' + tier + '/' + role + '/mysqlpassword length=15') }}"
                      priv={{ client }}_{{ tier }}_{{ role }}.*:ALL

        (...)

        # dump a mysql database with a given password (this example showing the other form).
        - mysql_db: name={{ client }}_{{ tier }}_{{ role }}
                    login_user={{ client }}
                    login_password={{ item }}
                    state=dump
                    target=target=/tmp/{{ client }}_{{ tier }}_{{ role }}_backup.sql
          with_password: credentials/{{ client }}/{{ tier }}/{{ role }}/mysqlpassword

        (...)

        # create an user with a given password
         - user: name=guestuser
                 state=present
                 uid=5000
                 password={{ item }}
           with_password: credentials/{{ hostname }}/userpassword encrypt=sha2     56_crypt

環境設定 (とプロキシ経由での動作)
`````````````````````````````````

.. versionadded: 1.1

プロキシを介してパッケージの更新を取得する必要があるとか、いくつかのパッケージ
はプロキシを介してアップデートを入手しつつ、他のパッケージはプロキシを介さずに
パッケージにアクセスする、ということも充分に可能です。ansibleは'environment'
キーワードを使うことによってあなたの環境を簡単に構成できるようにします。
次に例を示します::

    - hosts: all
      user: root

      tasks:

        - apt: name=cobbler state=installed
          environment:
            http_proxy: http://proxy.example.com:8080

environmentは変数に格納できるので、このようにアクセスできます::

    - hosts: all
      user: root

      # here we make a variable named "env" that is a dictionary
      vars:
        proxy_env:
          http_proxy: http://proxy.example.com:8080

      tasks:

        - apt: name=cobbler state=installed
          environment: "{{ proxy_env }}"

上ではプロキシを設定を示しているだけですが、任意の数の設定を提供できます。
環境設定のハッシュを定義するのに最も理に適っている場所は、group_varsファイル
かも知れません::

    ---
    # file: group_vars/boston

    ntp_server: ntp.bos.example.com
    backup: bak.bos.example.com
    proxy_env:
      http_proxy: http://proxy.bos.example.com:8080
      https_proxy: http://proxy.bos.example.com:8080


ファイルから値を取得する
````````````````````````

.. versionadded:: 0.8

時には、ファイルの内容を直接、playbookの中にインクルードしたことがある
でしょう。マクロを使えばそれはできます。
この構文は今後のバージョンでも残るでしょうが、我々はlookupプラグインを使って
同じように実行する方法("複数のループ"を参照のこと)を提供する予定です。
以下は、authorized_keysモジュールを使った例で、パラメータとして、実際の
SSHキーの実際のテキストを必要とします::

    tasks:
        - name: enable key-based ssh access for users
          authorized_key: user={{ item }} key="{{ lookup('file', '/keys/' + item ) }}"
          with_items:
             - pinky
             - brain
             - snowball


変数によってファイルとテンプレートを選択する
````````````````````````````````````````````

設定ファイルをコピーしたかったり、使用するテンプレートが変数に依存するような
場合があります。
次の構文は、特定のホストの変数として適している、利用可能な最初のファイルを
選択しますが、これはしばしばテンプレートの中で沢山のif条件文を書くよりもずっと
簡潔です。

次の例は、曰くCentOSとDebianの間で全く異なる設定ファイルをテンプレート出力する
方法を示しています::

    - name: template a file
      action: template src={{ item }} dest=/etc/myapp/foo.conf
      first_available_file:
        - /srv/templates/myapp/{{ ansible_distribution }}.conf
        - /srv/templates/myapp/default.conf

first_avaiable_file はcopyとtemplateモジュールでのみ使えます。


非同期アクションとポーリング
````````````````````````````

デフォルトでは、playbook内のタスクブロックは、各ノードでタスクが完了する
まで接続を開いたまま保持することを意味します。小さい並列度の値でplaybookを
実行する場合 (別名 ``--forks``)、実行時間の長い操作がもっと早く終ったらいい
のに、と思うかも知れません。これを実現する最も簡単な方法は、一度にすべてを
キックして、それらが終了するまでポーリングすることです。

また、タイムアウトの対象となる可能性のある非常に実行時間の長い操作に、非同期
モードを使いたいとも思うでしょう。

非同期タスクを起動するには、タスクの最大実行時間とステータスをポーリングしたい
頻度を指定します。 `poll` に値をしていなかった場合、デフォルトのポーリング間隔は
10秒です::

    ---
    - hosts: all
      user: root
      tasks:
      - name: simulate long running op (15 sec), wait for up to 45, poll every 5
        action: command /bin/sleep 15
        async: 45
        poll: 5

.. note::
   非同期時間制限にデフォルト値はありません。'async'キーワードを付けなかった
   場合、タスクはansibleのデフォルトで、同期的に実行されます。

また、タスクの完了を待つ必要がない場合は、pollの値に0を指定して"点けっ放し"に
することができます::

    ---
    - hosts: all
      user: root
      tasks:
      - name: simulate long running op, allow to run for 45, fire and forget
        action: command /bin/sleep 15
        async: 45
        poll: 0

.. note::
   あなたが同じリソースに対して、playbookの後の方で他のコマンドを
   実行しようとするなら、yumトランザクションのような排他的ロックが必要な
   操作は"点けっ放し"にするべきではありません。

.. note::
   ``--forks`` に高い値を使うと、結果として実行した非同期タスクの開始が
   より高速になります。またポーリングの効率がよくなります。


ローカルplaybook
````````````````````

SSH越しに接続するよりも、playbookをローカルで使うと有用な場合があります。
これはcrontabにplaybookを入れて、システムの構成を保証するのに役立ちます。
これはまた、Anacondaキックスタートのような、OSの中でplaybookを実行させる
ためにも使えます。

playbookを完全にローカルで実行するには、単純に"hosts:"行に
"hosts:127.0.0.1"を設定してからそのplaybookを実行します::

    ansible-playbook playbook.yml --connection=local

また、local接続は単独playbookのplayに使うことができ、そのplaybookの
他のplayがデフォルトのリモート接続を使っていても使用可能です::

    hosts: 127.0.0.1
    connection: local


fact をオフにする
`````````````````

一元的に自分のシステムについてすべてを把握していて、各ホストについていずれの
factデータも必要ないことが分かっている場合は、factの収集をオフにできます。
これは非常に台数の多いシステムに対してプッシュモードでansibleをスケールさせたり
、主に実験的なプラットフォームでansibleを使っている場合に利点があります。
どんなplayでも、こうするだけです::

    - hosts: whatever
      gather_facts: no


Pullモードplaybook
``````````````````````

ローカルモード(上記)でのplaybookの使用は、 `ansible-pull` を加えると
非常に強力になります。ansible-pull を設定するスクリプトは、Githubから
チェックアウトしたソースの examples/playbooks ディレクトリの中に提供されて
います。

基本的な発想は、それぞれの管理対象のノードにansibleのリモートコピーを設定して、
それぞれのセットでcronによる実行とgitによるplaybookソースの更新を行える
ようにするために、ansibleを使用するものです。これはデフォルトでプッシュ・
アーキテクチャのansibleをプル・アーキテクチャに反転させるもので、無限に近い
可能性を秘めています。セットアップのためのplaybookは、cronの実行頻度、
ログの出力場所、ansible-pullのためのパラメータを設定できます。

これは極端なスケールアウトのためだけではなく、定期的な修復にも有効です。
ansible-pullの実行したログを取得するための'fetch'モジュールの使い方は
ansible-pullのリモートログを収集・分析するための優れた方法でしょう。


変数を登録する
``````````````

.. versionadded:: 0.7

playbookの中で、与えられたコマンドの結果を変数に格納し、後でそれにアクセス
することが役に立つ場合がしばしばあります。コマンドモジュールのこのような使い方
は、例えば、特定のプログラムの存在をテストすることができるので、多くの場合、
サイト特有のfactを記述する必要を排除することができます。

'register'キーワードは、結果を保存する変数を決定します。結果の入った変数は、
テンプレート、アクション行および *when* 文で使えます。(本当にちょっとした例ですが)
このようになります::

    - name: test play
      hosts: all

      tasks:

          - action: shell cat /etc/motd
            register: motd_contents

          - action: shell echo "motd contains the word hi"
            when: motd_contents.stdout.find('hi') != -1


ローリングアップデート
``````````````````````

.. versionadded:: 0.7

デフォルトでは、ansibleは並行してplayの中で参照されているすべてのマシンを
管理しようとします。ローリングアップデートの場合は、"serial"キーワードを使う
ことで、ansibleが一度にいくつのマシンを管理すべきかを定義できます::

    - name: test play
      hosts: webservers
      serial: 3

上の例では、ホストが100台ある場合、'webservers'グループに含まれる3台のホストは
次の3台のホストに移る前に、完全にplayを完了します。

デリゲーション (移譲)
`````````````````````

.. versionadded:: 0.7

他のホストを参照して、あるホスト上でタスクを実行したい場合は、そのタスクに
'delegate_to'キーワードを使います。
これは負荷分散されたプールにノードを追加したり、外したりする場合に理想的です。
また、停止期間を制御するのにも非常に便利です。一度に実行するホストの数を制御
するために'serial'キーワードと一緒に使うのも良いアイデアです::

    ---
    - hosts: webservers
      serial: 5

      tasks:
      - name: take out of load balancer pool
        action: command /usr/bin/take_out_of_pool {{ inventory_hostname }}
        delegate_to: 127.0.0.1

      - name: actual steps would go here
        action: yum name=acme-web-stack state=latest

      - name: add back to load balancer pool
        action: command /usr/bin/add_back_to_pool {{ inventory_hostname }}
        delegate_to: 127.0.0.1

これらのコマンドはansibleを実行しているマシン、127.0.0.1で実行されます。これらの
タスクごとの単位で使える省略構文: 'local_action' もあります。
こちらは上のplaybookと同じですが、127.0.0.1に移譲するための省略構文を使って
います::

    ---
    # ...
      tasks:
      - name: take out of load balancer pool
        local_action: command /usr/bin/take_out_of_pool {{ inventory_hostname }}

    # ...

      - name: add back to load balancer pool
        local_action: command /usr/bin/add_back_to_pool {{ inventory_hostname }}

一般的なパターンは、管理対象サーバに対してファイルを再帰的にコピーするのに、
'rsync'を呼び出すために、ローカルアクションを使うことです。次に例を示します::

    ---
    # ...
      tasks:
      - name: recursively copy files from management server to target
        local_action: command rsync -a /path/to/files {{ inventory_hostname }}:/path/to/target/

これを実行するためには、パスフレーズなしのsshかsshエージェントが必要なことに
注意してください。そうでないとrsyncはパスフレーズの確認を必要とします。


Fireballモード
``````````````

.. versionadded:: 0.8

ansibleの'local'、'paramiko'および'ssh'のコア接続タイプに、バージョン0.8以降
では 'fireball'と呼ばれる接続タイプが拡張されました。これはplaybookとだけ
使用でき、 ansibleの通常の"起動処理不要"の哲学から外れた、いくつか追加の設定を
必要とします。 ansibleを使うのにfireballモードの使用は必須ではありませんが、
一部のユーザは喜ぶかも知れません。

fireballモードは、デフォルトではシャットダウン前の30分の間だけ、ssh経由で
一時的に0mqデーモンを起動することで動作します。fireballモードは一度起動すると
セッションの暗号化のために一時的なAESキーを使用し、設定されたポート上での、
特定のノードとの直接通信を必要とします。デフォルトは5099です。
fireballデーモンは設定を変更すると、任意のユーザで実行します。なので、自分でも
rootとしても実行できます。
複数のユーザが、同じホスト群でansibleを使っている場合は、固有のポートを使う
ように気をつけてください。

fireballモードは、paramikoを使ったノード間通信よりもだいたい10倍程度速く、
たくさんのホストがある場合には良い選択肢となるでしょう::

    ---

    # set up the fireball transport
    - hosts: all
      gather_facts: no
      connection: ssh # or paramiko
      sudo: yes
      tasks:
          - action: fireball

    # these operations will occur over the fireball transport
    - hosts: all
      connection: fireball
      tasks:
          - action: shell echo "Hello {{ item }}"
            with_items:
                - one
                - two

fireballモードを使うためには、両方のホストで特定の依存関係のインストールが必要
です。任意のプラットフォーム上で、最初の起動処理のための基礎として、このplay
ブックが使えます。またパッケージマネージャで、gccとzeromq-develのインストールが
必要ですが、これももちろんansibleでインストールできます::

    ---
    - hosts: all
      sudo: yes
      gather_facts: no
      connection: ssh
      tasks:
          - action: easy_install name=pip
          - action: pip name={{ item }} state=present
            with_items:
              - pyzmq
              - pyasn1
              - PyCrypto
              - python-keyczar

FedoraおよびEPELには、fireballの依存ライブラリに使えるサブパッケージもあります。

また、モジュールのドキュメントの節も参照してください。


変数の優先順位を理解する
````````````````````````

すでに、インベントリ変数、'vars'、'vars_files'については学びました。イベント内で
同じ変数名が複数の場所で発生すると、何が起こるでしょうか？実は３つの順位の層があり、
その層の中に、おそらくあなたは覚えておく必要のない、マイナーな順序規則があります。
ともかく、それらを説明しましょう。

playの実行中に設定される変数は、最も高い優先順位を持っています。これには、登録
された変数や、リモートホストに関して発見された情報の断片であるfactを含みます。

優先度が下がるのが、playbookの中で定義された変数です。'vars_files'として
playbookで定義されたものがその次で、ansible-playbook に --extra-vars (-e) を通じて
渡された変数、'vars'セクションで定義された変数が続きます。これらはすべて、基本的に
同じもの -- play の中ですべてのホストにすることについて定数を定義するのに適した
場所であると解釈されるべきです。

最後に、インベントリ変数が最も低い優先順位を持っています。ホストに関する変数は、
グループに関する変数を上書きします。もし変数が複数のグループで定義され、その一つが
ほかのグループの子グループである場合、子グループの変数は親の変数設定を上書きします。

このため'group_vars/all'ファイルは、別のグループやplaybookの中で上書きする
デフォルト値を定義するのに最適な場所になります。例えば、あなたの組織でデフォルトの
ntpサーバを group_vars/all の中で設定し、地域に基づくグループを基準に上書きして
しまうこともあります。
しかしplaybookのvarsセクションの中で'ntpserver: asdf.example.com' と記述すれば、
その特定の値が間違いなく使用されようとしているものであることは、playbookを読んで
知っています。忍び寄るインベントリ変数に騙されることはありません。

要するに、簡単に覚えるとすれば factはplaybookの定義より強く、playbookの定義は
インベントリ変数より強い、です。


チェックモード ("Dry Run") --check
``````````````````````````````````

.. versionadded:: 1.1

ansible-playbookを--checkを付けて実行すると、リモートのシステムにはなんの変更も
行いません。その代わりに、'チェックモード'のサポート (これはprimaryコアモジュー
ルに含まれていますが、すべてのモジュールでそれを行う必要はありません) を備えて
いるあらゆるモジュールは、行うであろう変更を報告します。チェックモードをサポート
していない他のモジュールは何も行わないので、それらのモジュールが行うであろう
変更も報告はされません。

チェックモードはただのシミュレーションなので、先行するコマンドの結果に依存する
条件を使った手順がある場合は、あまり役に立たないかも知れません。
しかし、"1度に1ノード"の基本的な構成管理のユースケースには最適です。

例::

    ansible-playbook foo.yml --check


--diff で差分を表示する
```````````````````````

.. versionadded:: 1.1

ansible-playbookの--diffオプションは、--check (詳細は上述) と一緒に使うと
素晴らしい効果がありますが、単独でも使用できます。このフラグが渡されると、
リモートシステム上でテンプレート出力ファイルが変更された場合に、
ansible-playbook CLI に、ファイルに大して行われたテキストの変更内容 (または、
--checkと同時に使った場合は、行われるであろう変更内容) の報告が戻ってきます。
差分機能は大量に出力を行うので、このように一度に単一のホストをチェックするのが
最適です::

    ansible-playbook foo.yml --check --diff --limit foo.example.com


辞書＆入れ子(複合)引数
``````````````````````

おさらいですが、ansibelのほとんどのタスクはこの形式です::

    tasks:

      - name: ensure the cobbler package is installed
        yum: name=cobbler state=installed

しかし、場合によっては、ハッシュ (辞書) から直接引数を供給するほうが便利です。
実際に、ごく一部のモジュール (CloudFormations モジュールは1つです) は、実際に
複雑な引数を必要とします。それらはこのように動作します::

    tasks:

      - name: call a module that requires some complex arguments
        foo_module:
           fibonacci_list:
             - 1
             - 1
             - 2
             - 3
           my_pets:
             dogs:
               - fido
               - woof
             fish:
               - limpet
               - nemo
               - {{ other_fish_name }}

上述のように、これらは内部変数として使うこともできます。

local_actionを使う場合、このようにできます::

    - name: call a module that requires some complex arguments
      local_action:
        module: foo_module
        arg1: 1234
        arg2: 'asdf'

これらはもちろん、より冗長ですが、技術的には正しい構文です::

    - name: foo
      template: { src: '/templates/motd.j2', dest: '/etc/motd' }


スタイルのポイント
``````````````````

ansible playbookは色付けされています。これが好きでない場合、環境変数
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
