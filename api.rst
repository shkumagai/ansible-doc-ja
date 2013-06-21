API & インテグレーション
========================

.. イメージ省略

APIの観点からAnsibleを使う場合、いくつか面白い方法があります。ノードを管理する
ためにAnsibleのPython APIが利用でき、様々なPythonのイベントに応答するように
Ansibleを拡張でき、外部リソースからインベントリデータを組み込めます。
AnsibleはそのAPIそのもので書かれているので、全体としてかなりの力を持っています。

.. contents:: `Table of contents`
   :depth: 2
   :backlinks: top


Python API
----------

Python API は非常に強力であり、Ansible CLIやansible-playbookを実装している方法
です。

これは非常に単純にです::

    import ansible.runner

    runner = ansible.runner.Runner(
       module_name='ping',
       module_args='',
       pattern='web*',
       forks=10
    )
    datastructure = runner.run()

runメソッドは、接続できたかそうでないかでグループ化して、ホストごとの結果を
返します。'ansible-modules'ドキュメントに書かれているように、戻り値の型は
モジュール固有のものです::

    {
        "dark" : {
           "web1.example.com" : "failure message"
        }
        "contacted" : {
           "web2.example.com" : 1
        }
    }

Ansibleは必要ならどんな型の値でも返せるので、Ansibleを強力なアプリケーションと
スクリプトを迅速に構築するためのフレームワークとして利用できます。


APIの詳細な例
`````````````

次のスクリプトはすべてのホストの稼働時間を出力します::

    #!/usr/bin/python

    import ansible.runner
    import sys

    # construct the ansible runner and execute on all hosts
    results = ansible.runner.Runner(
        pattern='*', forks=10,
        module_name='command', module_args='/usr/bin/uptime',
    ).run()

    if results is None:
       print "No hosts found"
       sys.exit(1)

    print "UP ***********"
    for (hostname, result) in results['contacted'].items():
        if not 'failed' in result:
            print "%s >>> %s" % (hostname, result['stdout'])

    print "FAILED *******"
    for (hostname, result) in results['contacted'].items():
        if 'failed' in result:
            print "%s >>> %s" % (hostname, result['msg'])

    print "DOWN *********"
    for (hostname, result) in results['dark'].items():
        print "%s >>> %s" % (hostname, result)

高度なプログラマは、コマンドラインツール ``ansible`` や ``ansible-playbook`` を
実装するための Runner() API（と利用可能なすべてのオプション）を使うために、
Ansible自身のソースを読みたいと思うかも知れません。


オンラインで利用可能なプラグイン
--------------------------------

APIドキュメントの残りの部分には、
`ansible-plugins <http://github.com/ansible/ansible/blob/devel/plugins>`_ の中で
利用可能なコンポーネントが含まれています。
何か面白い機能を開発したら、我々にGithubプル・リクエストを送ってください。


外部インベントリスクリプト
--------------------------

構成管理システムのユーザは、しばしばインベントリを異なるシステムで保管したいと
思うでしょう。よくある例としては、LDAPや `Cobbler <http://cobbler.github.com/>`_
または一部の高価なエンタープライズ向けCMDBソフトウェアです。Ansibleは外部
インベントリシステムを介して、簡単にこれらのオプションをすべてサポートします。
pluginsディレクトリには、すでにこれらのいくつかが含まれています -- 以下で詳述する
EC2/Eucalyptus や OpenStack用のオプションを含みます。

外部インベントリスクリプトはどんな言語でも書けます。あなたがもしPuppetの用語に
詳しいのであれば、この概念は、管理されているホストも定義するというわずかな違いが
ありますが、基本的には'external nodes'と同じです。


スクリプトの規約
````````````````

外部ノードスクリプトを '--list' 引数だけで実行した場合、スクリプトは管理する
すべてのグループのJSONハッシュ/辞書を返す必要があります。
それぞれのグループの値は、このように、それぞれホスト/IP、潜在的な子グループ、
潜在的なグループ変数のリストを含むハッシュ/辞書、または単純なホスト/IPアドレスの
リストでなければいけません::

    {
        'databases'   : {
            'hosts'   : [ 'host1.example.com', 'host2.example.com' ],
            'vars'    : {
                'a'   : true
            }
        },
        'webservers'  : [ 'host2.example.com', 'host3.example.com' ],
        'atlanta'     : {
            'hosts'   : [ 'host1.example.com', 'host4.example.com', 'host5.example.com' ],
            'vars'    : {
                'b'   : false
            },
            'children': [ 'marietta', '5points' ],
        },
        'marietta'    : [ 'host6.example.com' ],
        '5points'     : [ 'host7.example.com' ]
    }

.. versionadded: 1.0

バージョン1.0以前では、上記のwebservers、marietta、および5pointsグループのように、
それぞれホスト名/IPアドレスのリストだけを持てます。

引数'--host <hostname>'（<hostname>のところは前述のホスト）を付けて実行した場合、
テンプレートやplaybookが使えるように、スクリプトは空のハッシュ/辞書JSON、または
変数のハッシュ/辞書を返さなければなりません。変数の返却はオプションなので、そう
したくない場合は空のハッシュ/辞書を返すようにします::

    {
        'favcolor'   : 'red',
        'ntpserver'  : 'wolf.example.com',
        'monitoring' : 'pack.example.com'
    }


例: Cobblerによる外部インベントリスクリプト
```````````````````````````````````````````

多くのAnsibleユーザは `Cobbler <http://cobbler.github.com/>`_ でもあると思います。
Cobblerは複数の構成管理システムのデータを（複数同時に）表すことができる汎用レイヤ
を持ち、一部のアドミニストレータからは'軽量CMDB'と呼ばれています。
この特別なスクリプトはCobblerのXMLRPC APIを使ってCobblerと通信します。

CobblerとAnsibleのインベントリを結び付けるには、
`このスクリプト <https://raw.github.com/ansible/ansible/devel/plugins/inventory/cobbler.py>`_
を /etc/ansible/hosts にコピーし、ファイルを `chmod +x` します。これ以降は
Ansibleを使用するときには、cobblerdが実行されている必要があります。

`./etc/ansible/hosts` を直接実行してファイルをテストします。
いくつかのJSONデータの出力が表示されるはずですが、それだけではまだそれは何も
持っていない可能性があります。

さぁ、何が起こっているかを見てみましょう。cobblerは、以下のようなシナリオします::

    cobbler profile add --name=webserver --distro=CentOS6-x86_64
    cobbler profile edit --name=webserver --mgmt-classes="webserver" --ksmeta="a=2 b=3"
    cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta" --ksmeta="c=4"
    cobbler system edit --name=bar --dns-name="bar.example.com" --mgmt-classes="atlanta" --ksmeta="c=5"

上記の例では、システム'foo.example.com'はAnsibleから直接呼ぶことができますが、
グループ名'webserver'や'atlanta'を使って呼ぶこともできます。AnsibleはSSHを使って
いるので、'foo.example.com'をfooに短縮しようとするるだけで、単なる'foo'ではあり
ません。同様に、"ansible foo"としようとしてもそのシステムは見つからないでしょう
が、システムのDNSの名前は'foo'から始まっているので、"absible 'foo.*'"であれば
見つかるでしょう。

スクリプトはホストとグループの情報を提供するだけではありません。さらに特典として、
'setup'モジュールを実行（playbookを使っていると自動的に行われます）すると、変数
'a'、'b'および'c'はすべて自動的にテンプレートに投入されます::

    # file: /srv/motd.j2
    Welcome, I am templated with a value of a={{ a }}, b={{ b }}, and c={{ c }}

これらは、ちょうどこのように実行できます::

    ansible webserver -m setup
    ansible webserver -m template -a "src=/tmp/motd.j2 dest=/etc/motd"

.. note::
   'webserver'という名前は、設定ファイルの変数としてCobblerから取得したものです。
   依然、Ansibleでは通常のように独自の変数を渡すこともできますが、同じ名前を持つ
   外部のインベントリスクリプトからの変数は、すべて上書きします。

なので上記のテンプレート (motd.j2) では、システム'foo'の /etc/motd に書き込まれた
以下のデータをもたらすでしょう::

    Welcome, I am templated with a value of a=2, b=3, and c=4

そしてシステム'bar' (bar.example.com) では::

    Welcome, I am templated with a value of a=2, b=3, and c=5

さらに、技術的にはこれを行う大きな理由はありませんが、これでも動作します::

    ansible webserver -m shell -a "echo {{ a }}"

つまり、言い換えれば、それらの変数は引数/アクションでも使うことができます。
conf.dファイルに適切または似たような名前を付けるためにこれを使うことができます。
ご存知でしたか？

なので、Cobbler統合サポート -- 例のようにcobblerスクリプトを使うことは、変数
情報と同じように、任意のデータソースからインベントリを引いて、簡単にAnsibleに
適用できなければなりません。もし面白いものを作ったらメーリングリストで共有して
ください。そうすれば他の人が使うためのソースコードツリーでそれを維持できます。


例: AWS EC2 外部インベントリスクリプト
``````````````````````````````````````

例: OpenStack インベントリスクリプト
````````````````````````````````````

Callbackプラグイン
------------------

接続タイププラグイン
--------------------

参照プラグイン
--------------

Varsプラグイン
--------------

フィルタープラグイン
--------------------

分散プラグイン
--------------

.. versionadded: 0.8


.. seealso::

   :doc:`modules`
       List of built-in modules
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
