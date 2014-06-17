モジュールの開発
================

.. contents:: トピック

AnsibleモジュールはAnsible APIや `ansible` または `ansible-playbook`
プログラムから使える、再利用可能な魔法の単位です。

コアで開発されているさまざまな種類の一覧は :mod:`modules` を参照してください。

モジュールは任意の言語で記述でき、 `ANSIBLE_LIBRARY` かコマンドライン
オプション ``--module-path`` で指定したパスから見つけられます。

面白いAnsibleモジュールを開発して、あなたのモジュールをコアプロジェクトに
含めるべきかどうかを見られるように、githubプロジェクトにプルリクエストを
送ることを検討すべきです。

.. _module_dev_tutorial:

チュートリアル
``````````````

システム時刻を取得・設定する非常に基本的なモジュールを作ってみましょう。
まず最初に、現在の時刻を出力するだけのモジュールを作ってみましょう。

ここではPythonを使いますが、任意の言語を使えます。ファイルの入出力と、
標準出力への出力だけが必須です。
なのでbash, C++, clojure, Python, Ruby, どの言語を使っても結構です。

いまのPython Ansibleモジュールは（すべてのcoreモジュールが使用する）
いくつかの非常に強力なショートカットを含んでいますが、まずはとても難しい
やり方でモジュールを作ります。こうする理由は、Python以外のどの言語で書かれた
モジュールも全くこのとおりにやらなければならないからです。
簡単な方法は後でお見せします。

では、以下に例を示します。'command'モジュールがこれをやるために使えるので、
システム時刻を設定するモジュールを本当に作る必要は無いでしょう。
ですが、これを作ります。

Ansibleに付属しているモジュールを読むことは、モジュールの書き方を学ぶのに
最適な方法です。ですが、Ansibleソースツリーの中のいくつかのモジュールは
内部実装に寄り過ぎているので `service` や `yum` を見るようにして、
`async_wrapper` などには近寄り過ぎないように注意してください。
さもないと石になってしまいます。
async_wrapperを直接実行した人は未だにいません。

さあ、例に取り掛かりましょう。Pythonを使います。初めての人はこれを `time`
という名前を付けてファイルに保存してください ::

    #!/usr/bin/python

    import datetime
    import json

    date = str(datetime.datetime.now())
    print json.dumps({
        "time" : date
    })

.. _module_testing:

モジュールのテスト
``````````````````

チェックアウトしたAnsibleのソースには便利なテストスクリプトがあります::

    git clone git@github.com:ansible/ansible.git
    source ansible/hacking/env-setup
    chmod +x ansible/hacking/test-module

次のようにだけ書いて、スクリプトを実行しましょう::

    ansible/hacking/test-module -m ./time

このような感じに出力されるでしょう::

    {u'time': u'2012-03-14 22:13:48.539183'}

もしそうでなかったら、モジュールに誤字があるかもしれないので再度確認してから
もう一度実行してみてください。

.. _reading_input:

入力を読み取る
``````````````

現在時刻の設定ができるように、モジュールを変更してみましょう。モジュールに
`time=<string>` という形式のキーと値の組が渡された場合に、それを見て行う
ようにします。

Ansibleは内部的に引数を引数ファイルに保存します。なのでそのファイルを読んで
解析しなければなりません。引数ファイルは単純な文字列なので、どんな形式の
引数も有効です。ここでは key=value のような入力を扱うための基本的な構文解析を
します。

時刻を設定するために達成しようとしている使い方の例はこれです::

    time time="March 14 22:10"

timeパラメータが設定されない場合、時刻はそのままにして現在時刻を返します。

.. note::
   これはモジュールとしては明らかに非現実的な考えです。
   shellモジュールを使うのが最もふさわしいでしょう。
   ですが、これはおそらく適切なチュートリアルなのです。

コードをみてみましょう。説明する順にしたがってコメントを読んでください。
これは教育の例である事を意図しているので、非常に冗長であることに注意して
ください。これよりもずっと短く書けます::

    #!/usr/bin/python

    # import some python modules that we'll use.  These are all
    # available in Python's core

    import datetime
    import sys
    import json
    import os
    import shlex

    # read the argument string from the arguments file
    args_file = sys.argv[1]
    args_data = file(args_file).read()

    # for this module, we're going to do key=value style arguments
    # this is up to each module to decide what it wants, but all
    # core modules besides 'command' and 'shell' take key=value
    # so this is highly recommended

    arguments = shlex.split(args_data)
    for arg in arguments:

        # ignore any arguments without an equals in it
        if "=" in arg:

            (key, value) = arg.split("=")

            # if setting the time, the key 'time'
            # will contain the value we want to set the time to

            if key == "time":

                # now we'll affect the change.  Many modules
                # will strive to be 'idempotent', meaning they
                # will only make changes when the desired state
                # expressed to the module does not match
                # the current state.  Look at 'service'
                # or 'yum' in the main git tree for an example
                # of how that might look.

                rc = os.system("date -s \"%s\"" % value)

                # always handle all possible errors
                #
                # when returning a failure, include 'failed'
                # in the return data, and explain the failure
                # in 'msg'.  Both of these conventions are
                # required however additional keys and values
                # can be added.

                if rc != 0:
                    print json.dumps({
                        "failed" : True,
                        "msg"    : "failed setting the time"
                    })
                    sys.exit(1)

                # when things do not fail, we do not
                # have any restrictions on what kinds of
                # data are returned, but it's always a
                # good idea to include whether or not
                # a change was made, as that will allow
                # notifiers to be used in playbooks.

                date = str(datetime.datetime.now())
                print json.dumps({
                    "time" : date,
                    "changed" : True
                })
                sys.exit(0)

    # if no parameters are sent, the module may or
    # may not error out, this one will just
    # return the time

    date = str(datetime.datetime.now())
    print json.dumps({
        "time" : date
    })

モジュールをテストしてみましょう::

    ansible/hacking/test-module -m ./test -a test=\"March 14 12:23\"

これはこのような結果を返します::

    {"changed": true, "time": "2012-03-14 12:23:00.000307"}

.. _module_provied_facts:

モジュールが提供する 'Facts'
````````````````````````````

Ansibleに付属している'setup'モジュールはplaybookやtemplateで使用できる、
システムに関する多くの変数を提供します。
しかしシステムモジュールを変更しなくてもあなだ独自のfactsを追加できます。
これを行うには、このようにモジュールが他の戻り値と一緒に `ansible_facts`
キーを持つだけです::

    {
        "changed" : True,
        "rc" : 5,
        "ansible_facts" : {
            "leptons" : 5000
            "colors" : {
                "red"   : "FF0000",
                "white" : "FFFFFF"
            }
        }
    }

これら'facts'は、playbookの中でモジュールの後（前ではできません）に呼ばれる
全てのステートメントで使用できます。
我々は常にAnsibleと同様にコアfactsの選択の改善にもオープンにしていますが、
'site_fact'というモジュールを作成してそれぞれのplaybookの先頭で呼び出すようにする、
というのは良いアイデアかも知れません。

.. _common_module_boilerplate:

共通モジュールの鋳型
````````````````````

既に言ったとおり、モジュールをPythonで書く場合には非常に強力なショートカットが
使えます。
モジュールはひとつのファイルとして転送されますが引数ファイルはもう必要になるので、
コードの点で短いというだけでなく実行時間の点からも実際に高速です。

ここで言及するよりも、学ぶための一番の方法はAnsibleに付属している
`モジュールのソースコード <https://github.com/ansible/ansible/tree/devel/library>`_
をいくつか読むことです。

'group'と'user'モジュールは重要、かつこれがどのようなものかを紹介するのに合理的です。

キーパーツのインクルードは常にモジュールファイルの末尾で行い::

    from ansible.module_utils.basic import *
    main()

このようにしてモジュールのクラスをインスタンス化します::

    module = AnsibleModule(
        argument_spec = dict(
            state     = dict(default='present', choices=['present', 'absent']),
            name      = dict(required=True),
            enabled   = dict(required=True, choices=BOOLEANS),
            something = dict(aliases=['whatever'])
        )
    )

AnsibleModuleは戻り値の扱い、引数の解析、そして入力チェックのための多くの共通コードを
提供します。

成功時の戻り値はこのようになります::

    module.exit_json(changed=True, something_else=12345)

そして失敗時も同様に単純です（'msg'はエラーを説明するために必要なパラメータです）::

    module.fail_json(msg="Something fatal happened")

モジュールクラスには他にもmodule.md5(path)のような便利な関数があります。
実装の詳細については、チェックアウトしたソースのlib/ansible/module_common.py
を参照してください。

繰り返しますがこの方法で開発されたモジュールはgitのソースチェックアウトに含まれる
hacking/test-moduleスクリプトを使ってよくテストされています。
魔法を使っているので、これはAnsibleの外側でスクリプトが機能する唯一の方法です。

Ansibleのコアコードにモジュールを提供（推奨）する場合は、AnsibleModuleクラスの使用は必須です。

.. _developing_for_check_mode:

チェックモード
``````````````
.. versionadded:: 1.1

モジュールは任意にチェックモードをサポートできます。ユーザがAnsibleをチェックモードで
実行すると、モジュールは変更が起こるかどうかを予測するようにします。

自分のモジュールでチェックモードをサポートするには、AnsibleModuleオブジェクトを
インスタンス化するときに ``supports_check_mode=True`` を渡さなければいけません。
チェックモードが有効な場合、AnsibleModule.check_mode属性はTrueに評価されます。
例えば ::

    module = AnsibleModule(
        argument_spec = dict(...),
        supports_check_mode=True
    )

    if module.check_mode:
        # Check if any changes would be made by don't actually make those changes
        module.exit_json(changed=check_if_system_state_would_be_changed())

モジュール開発者は、ユーザがチェックモードを有効にした時には、システムの状態が変更されて
いないことを保証する責任があることを覚えておいてください。

自分のモジュールがチェックモードをサポートしない場合は、ユーザがAnsibleをチェックモードで
実行した場合には、単純にスキップされます。


.. _module_dev_pitfalls:

よくある落とし穴
````````````````

モジュールの中ではこれも行うべきではありません ::

    print "some status message"

なぜなら出力は有効なJSONであることが想定されているからです。
Except that's not quite true, but we'll get to that later.

システムが標準出力を標準エラーにマージしてJSONの解析を防ぐため、モジュールは標準エラーに
何も出力すべきではありません。標準エラーをキャプチャして、それを標準出力上のJSON内で
変数として返すのが適切ですし、実際にそれがcommandモジュールの実装方法です。

モジュールが標準エラーを返すか、有効なJSONを返すことに失敗した場合、実際の出力はAnsibleの
中で表示されますが、コマンドは成功しないでしょう。

モジュールを開発する際は常にhacking/test-moduleを使用してください。そうすれば、このような
種類の事象について警告してくれます。

.. note:: 訳注
   このセクションの訳はいまいちしっくり来ない。なので後で見直しする。


.. _module_dev_conventions:

規約と推奨
``````````


.. _module_dev_shorthand:

簡潔表現 vs JSON
````````````````


.. _module_documenting:

ドキュメントを書く
``````````````````


.. _module_doc_example:

例
++++


.. _module_dev_testing:

ビルドとテスト
++++++++++++++


.. _module_contribution:

モジュールをCoreに入れよう
``````````````````````````



.. seealso::

   :doc:`modules`
       Learn about available modules
   :doc:`developing_plugins`
       Learn about developing plugins
   :doc:`developing_api`
       Learn about the Python API for playbook and task execution
   `Github modules directory <https://github.com/ansible/ansible/tree/devel/library>`_
       Browse source of core modules
   `Mailing List <http://groups.google.com/group/ansible-devel>`_
       Development mailing list
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
