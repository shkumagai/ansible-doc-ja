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

入力を読み取る
``````````````

モジュールが提供する'fact'
``````````````````````````

共通モジュールボイラープレート
``````````````````````````````

checkモード
```````````

陥りやすい落とし穴
``````````````````

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
