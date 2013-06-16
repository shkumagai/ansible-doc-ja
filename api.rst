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
pluginsディレクトリには、すでにこれらのいくつかが含まれています -- 以下で詳述する EC2/Eucalyptus や OpenStack用のオプションを含みます。

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

引数'--host <hostname>'を付けて実行した場合、
