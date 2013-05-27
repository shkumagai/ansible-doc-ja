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

慎重に扱うべきデータを指示させる
````````````````````````````````

コマンドライン上から変数を渡す
``````````````````````````````

条件付き実行
````````````

条件付き実行 (簡易版)
`````````````````````

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
