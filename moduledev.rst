モジュール開発
==============

.. イメージ省略

Ansibleモジュールは、Ansible API または `ansible` もしくは `ansible-playbook`
プログラムによって使用できる、再利用可能な魔法の単位です。

モジュールはどんな言語でも記述でき、 `ANSIBLE_LIBRARY` や ``--module-path``
コマンドラインオプションで指定されたパスにあります。

.. contents::
   :depth: 2
   :backlinks: top


チュートリアル
``````````````

システム時刻を取得、および設定するためのモジュールを作りましょう。まず最初に、
現在の時刻を出力するためのモジュールを作りましょう。

ここではPythonを使おうとしていますが、任意の言語が使えます。必須なのは
ファイルI/Oと標準出力への出力だけです。なので、bash、C++、clojure、Python、
Ruby、などあなたが好きなもので結構です。

現在Ansible Python モジュールには(すべてのコアモジュールが使用している)非常に
強力なショートカットが含まれていますが、最初は面倒な方法でモジュールを作りたい
と思います。そうする理由は、Python "以外" の言語で書かれるモジュールは、
まさにそれをしなければならないからです。簡単な方法は、後で紹介します。

さて、ここからが例です。既に'command'モジュールがこれを実行するためにあるので、
システム時刻を作る必要はないでしょうが、作ります。

Ansibleに付属のモジュール(上記リンク)を読むことは、モジュールの作り方を学ぶのに
最適な方法です。但し、Ansibleのソースツリー内のモジュールのいくつかは内部向け
であることを覚えておいてください。 `service` や `yum` を見て、 `async_wrapper`
等にはあまり近くで見入らないようにしてください。さもないと石になります。
async_wrapperを直接実行するものはありません。

OK、例を挙げていきましょう。我々はPythonを使います。まず最初に、これを `time`
という名前でファイルに保存します::

    #!/usr/bin/python

    import datetime
    import json

    date = str(datetime.datetime.now())
    print json.dumps({
        "time" : date
    })


テストモジュール
````````````````

Ansibleのチェックアウトしたソースには、テストに役立つスクリプトがあります::

    git clone git@github.com:ansible/ansible.git
    chmod +x ansible/hacking/test-module

これだけ書いて、スクリプトを実行してみましょう::

    ansible/hacking/test-module -m ./time

このような出力が見れるはずです::

    {u'time': u'2012-03-14 22:13:48.539183'}

もしそうでない場合、モジュールの中でtypoしているかも知れないので、再度確認を
してから試してみてください。


入力を読み取る
``````````````

現在の時刻を設定できるように、モジュールを変更してみましょう。キーと値のペアが
`time=<string>` の形でモジュールに渡された場合に、それを見るようにします。

Ansibleは内部的に、引数を引数ファイルに保存します。そのため、我々はファイルを
読み込んでパースしなければなりません。引数ファイルは単なる文字列なので、どんな
引数の形式でも有効です。ここでは入力を key=value として扱うための、いくつか
基本的なパースを行います。

我々が時間を設定するために、達成しようとしている使い方の例です::

   time time="March 14 22:10"

時間のパラメータがなにも設定されていない場合、時間の設定はそのままにして現在の
時刻を返します。

.. note::
   これはモジュールにとしては明らかに非現実的なアイデアです。あなたは単に
   shellモジュールを使うだけになりそうですが、まともなチュートリアルに
   なります。

コードを見てみましょう。我々がこれからやろうとしていることを表わしている
コメントを読んでください。教育用の例を意図しているため、とても冗長になっている
ことに注意してください。これよりもずっと短くコードを書くことはできます::

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
        if arg.find("=") != -1:

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

モジュールをテストしましょう::

    ansible/hacking/test-module -m ./time -a time=\"March 14 12:23\"

このように返ってくるはずです::

    {"changed": True, "time": "2012-03-14 12:23:00.000307"}


モジュールが提供する'fact'
``````````````````````````

Ansibleに付属の'setup'モジュールはplaybookやテンプレートで使用可能な、システムに
関する多くの変数を提供します。しかしシステムモジュールを変更しなくても、
独自のfactを追加することもできます。これを行うには、モジュールが他の戻り値と
一緒に `ansible_facts` キーを返すようにします::

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

これらのfactは、playbook内でそのモジュールの（前ではなく）後に呼び出された
すべての文で使えるようになります。'site_facts'というモジュールを作って、それを
常に各playbookの先頭で呼び出すのが良いアイデアかも知れませんが、同時に我々は
Ansibleのコアfactの選択の改善についてはいつでも歓迎します。


共通モジュールボイラープレート
``````````````````````````````

既に述べたように、Pythonでモジュールを作成する場合、非常に強力なショートカットが
使えます。モジュールは依然、一つのファイルとして転送されますが、引数ファイルが
不要になったので、コードの観点からみて短いだけではなく、実行時間の観点から見ても
実際に `より高速` です。

ここで言及するよりは、Ansibleに付属している
`モジュールのソース <https://github.com/ansible/ansible/tree/devel/library>`_ を
読むのが最高の勉強です。

'group'と'user'モジュールが、まあまあ簡単ではなくてどのように見えるかの見本です。

キーパーツのインクルードは、常にモジュールファイルの終わりで行い::

    # include magic from lib/ansible/module_common.py
    #<<INCLUDE_ANSIBLE_MODULE_COMMON>>
    main()

そしてモジュールクラスを、このようにインスタンス化します::

    module = AnsibleModule(
        argument_spec = dict(
            state     = dict(default='present', choices=['present', 'absent']),
            name      = dict(required=True),
            enabled   = dict(required=True, choices=BOOLEANS),
            something = dict(aliases=['whatever'])
        )
    )

AnsibleModuleクラスは、戻り値の扱い、引数のパースおよび入力値のチェックが可能な
多くの共通コードを提供します。

成功時の戻り値はこのように作られています::

    module.exit_json(changed=True, something_else=12345)

そして失敗時も同じように('msg'はエラーを説明するのに必要なパラメータです)
簡単です::

    module.fail_json(msg="Something fatal happened")

モジュールクラスの中にはmodule.md5(path)のように、他にも便利な機能があります。
実装の詳細については、チェックアウトしたソースの lib/ansible/module_common.py
を参照してください。

繰り返しますが、このやり方で開発されたモジュールは、チェックアウトしたgitの
ソースの中のhacking/test-moduleスクリプトで、十分にテストされています。魔法が
関係しているせいで、これはAnsibleの外で機能できる唯一の方法です。

もし我々の勧めでAnsibleのコアコードにモジュールを提出する場合は、AnsibleModuleの
使用が必須になります。


checkモード
```````````

.. versionadded:: 1.1

モジュールは、必要に応じてチェックモードをサポートできます。ユーザがチェック
モードでAnsibleを実行している場合、モジュールは変更が発生するかどうかを予測
する必要があります。

チェックモードをサポートするためには、AnsibleModuleオブジェクトをインスタンス化
する際に、 ``support_check_mode=True`` を渡さなければいけません。
チェックモードが有効の場合は、AnsibleModule.check_mode属性はTrueと評価します。
例えば::

    module = AnsibleModule(
        argument_spec = dict(...),
        supports_check_mode=True
    )

    if module.check_mode:
        # Check if any changes would be made by don't actually make those changes
        module.exit_json(changed=check_if_system_state_would_be_changed())

モジュール開発者として、あなたは、ユーザがチェックモードを有効にしている場合は、
システムの状態が全く変更されないことを保証する責任がある、ということを覚えて
おいて下さい。

あなたのモジュールがチェックモードをサポートしない場合、ユーザがAnsibleを
チェックモードで実行した時は、あなたのモジュールは単純にスキップされます。


陥りやすい落とし穴
``````````````````

モジュールの中では、これも行うべきではありません::

    print "some status message"

出力は正しいJSONであることが想定されているからです。

慣習/勧告
`````````

省略記法 vs JSON
````````````````

自分のモジュールのドキュメントを書く
````````````````````````````````````

例
++++


ビルド & テスト
+++++++++++++++


あなたのモジュールをコアに
``````````````````````````


.. seealso::

   :doc:`modules`
       Learn about available modules
   `Ansible Resources <https://github.com/ansible/ansible/tree/devel/contrib>`_
       User contributed playbooks, modules, and articles
   `Github modules directory <https://github.com/ansible/ansible/tree/devel/library>`_
       Browse source of core modules
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
