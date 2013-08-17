playbook
============

.. イメージ省略

.. contents::
   :depth: 2
   :backlinks: top


イントロダクション
``````````````````

playbookはタスク実行モードに比べ、全く異なるansibleの使い方で、非常に
強力です。簡単にいえば、playbookは複雑なアプリケーションのデプロイに特化した
ものとも、既存のいずれのものとも違って、本当にシンプルな構成管理と複数マシンへの
デプロイシステムのための基盤です。

playbookは構成を宣言できますが、任意に手動の命令プロセスの手順を調整すること
もでき、ある命令が行われたマシンのセットの中で、異なる手順をあれこれ試してみる
ことさえも可能です。同期または非同期にタスクを起動できます。

アドホックなタスクには /usr/bin/ansible プログラムをメインに実行してよいですが、
playbookはソース管理の中に保存して、構成をpushしたり、リモートシステムの
構成が仕様に従っていることを保証するために利用するのに適しています。

さぁ、取り掛かってそれらがどう動くのかを見てみましょう。進むに従って、別のタブに
`github の examples ディレクトリ <https://github.com/ansible/ansible/tree/devel/examples/playbooks>`_
を開きたくなる場合もあるでしょうが、そうすると実際の見え方に理論を当てはめること
もできます。

`ansible-examples リポジトリ <https://github.com/ansible/ansible-examples>`_ に
これらの技術の多くを説明するための、playbookのフルセットがいくつかあります。


playbook言語の例
````````````````````

playbookはYAML形式と最小限の文法で表現します。各playbookはリストの中の
一つ以上の 'plays' で構成されています。

playのゴールはansibleがタスクと呼ぶもので表現される、明確に定義されたいくつか
のロールに、ホストのグループをマップすることです。基本的なレベルでは、タスクとは
以前の章で学んだはずの、ansibleモジュールへの呼び出し以上の何ものでもありません。

複数の 'plays' でplaybookを構成することで、複数マシンへデプロイして、
webserversグループのすべてのマシンで特定の手順を実行して、その後database server
グループで手順を実行して、さらにwebserversグループに戻ってコマンドを実行して、
等々の調整が可能です。

まず最初に、これは一つのplayが含まれたplaybookです::

    ---
    - hosts: webservers
      vars:
        http_port: 80
        max_clients: 200
      user: root
      tasks:
      - name: ensure apache is at the latest version
        action: yum pkg=httpd state=latest
      - name: write the apache config file
        action: template src=/srv/httpd.j2 dest=/etc/httpd.conf
        notify:
        - restart apache
      - name: ensure apache is running
        action: service name=httpd state=started
      handlers:
        - name: restart apache
          action: service name=httpd state=restarted

以下では、playbook言語の様々な機能が何であるかを分類していきます。

基本
````

ホストとユーザ
++++++++++++++

playbookの中の各playのために、インフラの中のどのマシンを対象とするか、
どのリモートユーザがその手順 (タスクと呼ばれます) を完了させるかを選択します。

`hosts` の行は :ref:`patterns` のドキュメントに記載されているように、コロンで
区切られた一つ以上のグループまたはホストパターンです。
`user` はユーザアカウントの名前だけです::

    ---
    - hosts: webservers
      user: root

sudoからの実行のサポートも用意されています::

    ---
    - hosts: webservers
      user: yourname
      sudo: yes

play全体の代わりに特定のタスクに対してのみsudoすることもできます::

    ---
    - hosts: webservers
      user: yourname
      tasks:
        - service: name=nginx state=started
          sudo: yes

自分のアカウントでログインしてから、別のroot以外の別のユーザにsudoもできます::

    ---
    - hosts: webservers
      user: yourname
      sudo: yes
      sudo_user: postgres

sudoのパスワードを指定する場合は、 `ansible-playbook` を ``--ask-sudo-pass``
(`-K`) 付きで実行してください。sudoのplaybookを実行して、playbookが
ハングしたように見える場合は、sudoのプロンプトで詰まっているかも知れません。
単純に `Control-C` でkillして、 `-K` を付けてもう一度実行してください。

.. important::

   root 以外のユーザのために `sudo_user` を使う場合、モジュール引数は一時的に
   /tmpの下のランダムな一時ファイルに書き込まれます。これらはコマンドが実行され
   た後、すぐに削除されます。これは 'bob' から 'timmy' のようなユーザからの
   sudoの時にのみ起こるもので、'bob' から 'root' になる場合や、直接 'bob' や
   'root' でログインした場合には起こりません。データが一時的に読み取り可能
   (書き込み可能ではない) である事を懸念するなら、 `sudo_user` を設定して
   暗号化されていないパスワードの転送を避けてください。これ以外のケースでは
   '/tmp'は使用されず、playと関わることもありません。
   ansibleもパスワードのパラメータを記録しないように注意を払っています。


変数セクション
++++++++++++++

`vars` セクションは、このようにplayのなかで使うことができる変数と値のリストを
含みます::

    ---
    - hosts: webservsers
      user: root
      vars:
         http_port: 80
         van_halen_port: 5150
         other: 'magic'

.. note::
   変数を個別のファイルに保持して、play中の `vars_file` 宣言とインラインの
   `vars` と一緒に、それらをインクルードすることもできます。詳しくは
   `高度なPlaybookの章 <http://ansible.cc/docs/playbooks2.html#variable-file-separation>`_
   を参照してください。

これらの変数はplaybookの後半でこのように使えます::

    $varname or ${varname} or {{ varname }}

文字列を大文字化するなど複雑なことをしたい場合には、Jinja2 テンプレートエンジンが
使うように {{ varname }} がベストです。出力が文字列である場合には、いつもこの
フォームを使う習慣を身に付けておくことをおすすめします。

ですが、他の単純な値を参照する場合には $x や ${x} を使っても大丈夫です。
これは、データ構造が別のデータ構造の値を持つ場合には一般的です。

もっとJinja2について学ぶため、必要に応じて `Jinja2 docs <http://jinja.pocoo.org/docs/>`_
を参照できますが、Jinja2のループや条件分岐はAnsibleの'templates'のみのもので、
playbook では ansible がループや条件分岐のために 'when' や 'with' キーワードを
持っていることを覚えておいてください。

システムについての変数がある場合、それは'facts'と呼ばれ、これらの変数はplaybookに
戻され、ただ明示的に設定された変数のように、各システムで使えます。ansibleは、
'ansible'のプレフィックスが付いたこれらをいくつか提供していて、モジュールの
ドキュメントの 'setup' の下に記載されています。さらに、factsはもしそれらが
インストールされていれば、ohaiやfacterによって収集できます。facterの変数には
プレフィックス ``facter_`` が付き、ohaiの変数にはプレフィックス ``ohai_`` が
付きます。これらは単に、追加の依存関係を足し、このような他のシステムからユーザを
簡単に移植するためのものです。

例はどうでしょう。 /etc/motd ファイルにホスト名を書き込みたい場合には、こう言うことが
できます::

    - name: write the motd
      action: template src=/srv/templates/motd.j2 dest=/etc/motd

そして、/srv/templates/motd.j2 の中は::

    You are logged into {{ facter_hostname }}

しかし、前倒しでplaybook内のタスクを示すだけのつもりが、ちょっと先走り過ぎて
しまいました。さぁ、タスクについて話しましょう。


タスクリスト
++++++++++++

各playはタスクのリストを含みます。タスクはホストパターンにマッチするマシンに
対して、次のタスクに移るまえに、一度に一つずつ、順番に実行されます。
playの中では、すべてのホストが同じタスクディレクティブを取得しようとすること
を理解するのが重要です。タスクに選択したホストをマッピングするのがplayの目的
です。

上から下まで、playbookを実行している時、失敗したタスクを持つホストは
playbook全体のローテーションからは外されます。何か失敗した場合は、単純に
playbookファイルを修正し、再実行してください。

それぞれのタスクのゴールは、とても具体的な引数を使ってモジュールを実行すること
です。変数は、上述したように、モジュールの変数として使えます。

モジュールは'冪等'、つまりあなたがそれらを実行したら、希望する状態にシステムを
変えるように変更を加えます。これにより、非常に安全に同じplaybookを何度も
再実行できます。playbookは物事を変更するひつようがない限り、何も変更する
ことはありません。

`command` と `shell` モジュールは通常、同じコマンドを再実行しますが、コマンドが
'chmod'や'setsetool'等のようなものであれば、全く問題ありません。
これらのモジュールも冪等にするために利用可能な'creates'フラグもありますが。

すべてのタスクは `name` があり、実行中のplaybookからの出力含まれます。
これは人間のために出力されるので、それぞれのタスクステップにちょうどよい説明が
あると便利です。名前が提供されていない場合は、'action' に送られた文字列が
出力に使われます。

これは、ほとんどのモジュールと同様ですが、serviceモジュールがkey=valueの引数を
とる基本的な例です::

    tasks:
      - name: make sure apache is running
        action: service name=httpd state=running

`command` と `shell` モジュールは引数のリストだけを取るモジュールで、key=valueの
引数は使いません。あなたが期待するように動作させるためにはこうします。単純に::

    tasks:
      - name: disable selinux
        action: command /sbin/setenforce 0

command と shellモジュールは戻り値をケアするので、もし正常終了の値がゼロでない
コマンドがある場合には、こうすることもできます::

    tasks:
      - name: run this command and ignore the result
        action: shell /usr/bin/somecommand || /bin/true

またはこうです::

    tasks:
      - name: run this command and ignore the result
        action: shell /usr/bin/somecommand
        ignore_errors: True

action行が気持ちよく書くにはあまりにも長すぎる場合には、スペースの部分で区切って
任意の継続行をインデントすることができます::

    tasks:
      - name: Copy ansible inventory file to client
        action: copy src/etc/ansible/hosts dest=/etc/ansible/hosts
                owner=root group=root mode=0644

変数はaction行で使えます。'vars' セクションで 'vhost' という変数を定義したと
仮定すると、このようにできます::

    tasks:
      - name: create a virtual host file for {{ vhost }}
        action: template src=somefile.j2 dest=/etc/httpd/conf.d/{{ vhost }}

それら同様の変数はテンプレートの中でも使えますが、それは追々触れます。

通常、'include'ディレクティブを使ってタスクを分割する方が理にかなっているでしょう
が、いま非常に基本的なplaybookでは、すべてのタスクはplayの中に直接記述して
います。そのことについては少し跡で触れます。


アクションの省略記法
````````````````````

.. versionadded:: 0.8

このように "action:" と明確な単語を列挙するのではなく::

    action: template src=templates/foo.j2 dest=/etc/foo.conf

こう言うことも可能です::

    template: src=templates/foo.j2 dest=/etc/foo.conf

モジュールの名前は単純にコロンと、モジュールの引数が続きます。この方がずっと
直感的だと思います。我々のドキュメントは、多くのユーザがまだ古いバージョンを
使用している可能性があるので、まだ新しい形式に変換されていないだけです。
どちらの書式もずっと使うことができます。


変更によって実行する操作
````````````````````````

既に言及している通り、モジュールは'冪等'となるように書かれていて、リモート
システム上で変更を行なった際には連携ができます。playbookはこれを認識し、
変化に対応するために使える基本的なイベントシステムを持っています。

これらの'通知'アクションはplaybookの中の、各タスクブロックの終わりにトリガされ、
また複数の異なるタスクによって通知されても、トリガされるのは一度だけです。

例えば、複数のリソースはconfigファイルを変更されているので、apacheを再起動する
必要がありますが、不必要な再起動を避けるためapacheには一度だけ知らされます。

ここではファイルの内容が変更された時に２つのサービスをリスタートする例を
示しますが、ファイルの変更があった時だけです::

    - name: template configuration file
      action: template src=template.j2 dest=/etc/foo.conf
      notify:
         - restart memcached
         - restart apache

'notify' セクションにリストされているのはハンドラと呼ばれるタスクです。

ハンドラはタスクのリストで、実際には通常のタスクと違いはなく、名前で参照されます。
ハンドラはnotifyに通知するものです。ハンドラを何も通知しない場合、実行されません。
どれだけ多くハンドラに通知されたかには関係なく、特定のplayの中のすべてのタスクが
完了した後に、一度だけ実行されます。

これはハンドラセクションの例です::

    handlers:
        - name: restart memcached
          action: service name=memcached state=restarted
        - name: restart apache
          action: service name=apache state=restarted

ハンドラはサービスのリスタートや再起動のトリガに最もよく使われます。
ほとんどの場合、おそらく必要とすることはないでしょう。

.. note::
   ハンドラの通知は書かれた順番で実行されます。

role は後で説明します。ハンドラが 'pre_tasks'、'roles'、'tasks' そして
'post_tasks' の間で自動的に処理される事は注目に値します。ですが、もしすべての
ハンドラコマンドを直ちにフラッシュしたい場合、1.2以降であればこのようにできます::

    tasks:
       - shell: some tasks go here
       - meta: flush_handlers
       - shell: some other tasks

上記の例では、'meta'ステートメントに到達した時に、キューされている任意のハンドラは
先に処理されます。これは少しニッチなケースですが、時々便利なことがあります。


タスクインクルードファイルと再利用の勧め
````````````````````````````````````````

タスクのリストをplayやplaybookの間で再利用したいとします。これを行うために
ファイルのインクルードが使えます。タスクリストのインクルードの利用は、システムが
満たそうとしているロールを定義するのに最適な方法です。playbookのplayの目的
は複数の役割にシステムのグループをマッピングすること、ということを覚えておいて
ください。これがどのようなものか見てみましょう...

タスクインクルードファイルは単純にフラットなタスクのリストなので::

    ---
    # possibly saved as tasks/foo.yml
    - name: placeholder foo
      action: command /bin/foo
    - name: placeholder bar
      action: command /bin/bar

includeディレクティブはこのようになり、playbookの中で通常のタスクと混在
させることができます::

    tasks:
      - include: tasks/foo.yml

インクルードファイルに変数を渡すこともできます。これを 'parameterized include'
と呼んでいます。

例えば、もし複数のwordpresインスタンスをデプロイするなら、自分のwordpressタスクを
一つのwordpress.ymlファイルに含め、このように使うことができます::

    tasks:
      - include: wordpress.yml user=timmy
      - include: wordpress.yml user=alice
      - include: wordpress.yml user=bob

渡された変数は、インクルードされたファイルの中で使用できます。このように参照できます::

    {{ user }}

(明示的に渡したパラメータに加えて、varsセクションのすべての変数も同じようにここで
利用可能です。)

1.0以降では、代替の構文を使ってインクルードファイルに変数を渡すこともでき、これは
構造化された変数もサポートしています::

    tasks:

      - include: wordpress.yml
        vars:
            user: timmy
            some_list_variable:
              - alpha
              - beta
              - gamma

playbookは他のplaybookをインクルードすることもできますが、それについては
後のセクションで説明します。

.. note::
   1.0の時点で、タスクインクルード構文は任意の深さで使用できます。以前は、単一の
   レベルに限定されていたので、タスクインクルードは、タスクインクルードを含む他の
   ファイルをインクルードできませんでした。

インクルードは'handlers'セクションでも使えるので、例えばapacheの再起動の方法を
定義したければ、一度それを行うだけですべてのplaybookで使えます。
このような handler.yml をつくればよいでしょう::

    ---
    # this might be in a file like handlers/handlers.yml
    - name: restart apache
      action: service name=apache state=restarted

そして、メインのplaybookファイルで、playの一番下で、それをこのように
インクルードします::

    handlers:
      - include: handlers/handlers.yml

インクルードは通常のインクルードではないタスクやハンドラと混在させることが
できます。

インクルードはplaybookを別のplaybookにインポートするために使うことも
できます。これによって、他のplaybookで構成されたトップレベルのplaybookを
定義することができます。

例::

    - name: this is a play at the top level of a file
      hosts: all
      user: root
      tasks:
      - name: say hi
        tags: foo
        action: shell echo "hi..."

    - include: load_balancers.yml
    - include: webservers.yml
    - include: dbservers.yml

playbook内の別のplaybookを含む場合、変数の置換ができないことに注意して
ください。

.. note::

   'vars_files' でできるような、インクルードファイルの場所への条件付きパスは
   使えません。それを行う必要があると分かった場合は、playbookをもっとクラス/
   ロール指向に構築しなおすことを検討してください。どのインクルードファイルを
   使うのかを決めるために'fact'を使うことはできません。playに含まれるすべての
   ホストは同じタスクを得ようとします。
   (*when* は、ホストが条件によってタスクをスキップする機能を提供します。)


ロール
``````

.. versionadded: 1.2

vars_files、タスクそしてハンドラについて学んだ今、あなたのplaybookを整理するのに
最良の方法は何でしょうか？端的に言うとroleを使うことです！roleは既知のファイル
構造に基いて、あるvars_files、タスク、ハンドラを自動的に読み込む方法です。
roleによってコンテンツをグループ化すると、他のユーザとroleを簡単に共有できます。

roleは上で再度説明したように、単に'include'ディレクティブを中心とした自動化で、
実際に参照先ファイルの検索パス処理を改善する以外に、追加の魔法はほぼありません。
しかし、効果は絶大です。

プロジェクト構造の例::

    site.yml
    webservers.yml
    fooservers.yml
    roles/
       common/
         files/
         templates/
         tasks/
         handlers/
         vars/
       webservers/
         files/
         templates/
         tasks/
         handlers/
         vars/

playbookの中はこのような感じです::

    ---
    - hosts: webservers
      roles:
         - common
         - webservers

これはそれぞれのrole 'x' に対して以下の動作を指示します:

- roles/x/tasks/main.yml が存在する場合、そこに記載されているタスクがplayに
  追加されます
- roles/x/handlers/main.yml が存在する場合、そこに記載されているハンドラがplayに
  追加されます
- roles/x/vars/main.yml が存在する場合、そこに記載されている変数がplayに
  追加されます
- 任意のcopyタスクは、相対的または絶対的パスを用いることなく roles/x/files/ の
  中のファイルを参照できます
- 任意のscriptタスクは、相対的または絶対的パスを用いることなく roles/x/files/ の
  中野ファイルを参照できます
- 任意のtemplateタスクは、相対的または絶対的パスを用いることなく roles/x/templates/
  の中のファイルを参照できます

ファイルがなにも存在しない場合は、単に無視します。なので、例えばroleに'vars/'サブ
ディレクトリが無くても大丈夫です。

注意してほしいことは、playbookの中でroleを使わずに"緩く"タスクやvars_files、
ハンドラを書き連ねることも依然としてできますが、roleは良い整理機能なので強くお勧め
します。playbook内に緩いモノがある場合、roleが最初に評価されます。

roleをパラメータ化する必要があるなら、変数を追加して、このようにすることも
できます::

    ---
    - hosts: webservers
      roles:
        - common
        - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
      roles:
        - common
        - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
        - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }

また、それほど頻繁に行うべきものではありませんが、条件付きでroleを適用することも
できます::

    ---
    - hosts: webservers
      roles:
        - { role: some_role, when: "ansible_os_family == 'RedHat'" }

これは、roleの中のすべてのタスクに条件を適用して動作します。条件文はドキュメントの
後半でカバーされています。

playにまだ'tasks'セクションがある場合、それらのタスクはroleが適用された後に
実行されます。

あるタスクをroleの前と後に定義したい場合、このようにすることができます::

    ---
    - hosts: webservers
      pre_tasks:
        - shell: echo 'hello'
      roles:
        - { role: some_role }
      tasks:
        - shell: echo 'still busy'
      post_tasks:
        - shell: echo 'goodbye'


playbookの実行
``````````````````

playbookの構文について学んできましたが、どのように実行しますか？
それは簡単です。並列処理レベル10でplaybookを実行してみましょう::

    ansible-playbook playbook.yml -f 10


ヒントとコツ
````````````

実行されたノードやどのように実行されたかのサマリは、playbookの実行結果の
一番下をみてください。一般的な失敗や、致命的な通信試行の"到達不能"などが個別に
カウントされています。

失敗したモジュールだけでなく、成功したものからも詳細なを出力を表示したい場合は、
'--verbose' フラグを使用します。これはansible 0.5以降で利用可能です。

また0.5以降では、cowsayパッケージがインストールされている場合には、ansible
playbookの出力が大幅にアップグレードします。お試しください！

バージョン0.7以降では、playbookを実行擦る前に、ホストが影響を受けるかどうか
を、このようにして確認できます::

    ansible-playbook playbook.yml --list-hosts


.. seealso::

   :doc:`YAMLSyntax`
       YAML 構文について学ぶ
   :doc:`playbooks`
       基本的なplaybook言語の機能のおさらい
   :doc:`playbooks2`
       高度なplaybook機能について学ぶ
   :doc:`bestpractices`
       実際のplaybookの管理についての様々なヒント
   :doc:`modules`
       利用可能なモジュールについて学ぶ
   :doc:`moduledev`
       自分のモジュールを書いてansibleを拡張する方法を学ぶ
   :doc:`patterns`
       ホストを選択する方法を学ぶ
   `Github examples directory <https://github.com/ansible/ansible/tree/devel/examples/playbooks>`_
       Complete playbook files from the github project source
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
