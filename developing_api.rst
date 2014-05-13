Python API
==========

.. contents:: トピックス

APIの視点からAnsibleを使うには、いくつかの方法があります。
Ansible Python APIを使ってノードを制御したり、Pythonのイベントに応答するように
Ansibleを拡張したり、プラグインを書いたり、外部のデータソースから
インベントリデータを差し込むことができます。
このドキュメントではRunnerおよびPlaybook APIの基礎レベルをカバーしています。

もしPython以外の何かからプラグラム的にAnsibleを使ったり、非同期にイベントを
トリガしたり、アクセス制御やログ要求をする方法を探しているなら、高レベルの
REST APIでこれらを提供している :doc:`tower` のドキュメントをご覧ください。

Ansibleは自身のAPIによって記述されているので、すべてにおいて相当な力を
発揮します。この節ではPython APIについて論じます。

.. _python_api:

Python API
----------

Python API は非常に強力な、Ansible CLIやansible-playbookが実装されている
やり方です。

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
モジュール固有のものです。::

    {
        "dark" : {
           "web1.example.com" : "failure message"
        }
        "contacted" : {
           "web2.example.com" : 1
        }
    }

Ansibleは必要に応じてどんな型の値でも返せるので、Ansibleは協力な
アプリケーションとスクリプトを迅速に構築するためのフレームワークとして
利用できます。

.. _detailed_api_example:

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
