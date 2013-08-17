Ansibleモジュール
=================

.. イメージ省略

.. contents::
   :depth: 3
   :backlinks: top


イントロダクション
``````````````````

Ansibleは、直接または :ref:`playbook` を経由してリモートホストで実行できる
複数のモジュール('モジュールライブラリ'と呼ばれます)が同梱されています。
ユーザが独自のモジュールを記述することもできます。
これらのモジュールはサービス、パッケージまたはファイル(実際の何か)のような
システムリソースの制御やシステムコマンドの実行を扱えます。

コマンドラインから、3つの異なるモジュールを実行する方法を確認してみましょう::

    ansible webservers -m service -a "name=httpd state=running"
    ansible webservers -m ping
    ansible webservers -m command -a "/sbin/reboot -t now"

それぞれのモジュールは引数の取得をサポートします。ほぼ全てのモジュールが、
スペース区切りで ``key=value`` の引数を取ります。いくつかのモジュールは引数を
取らず、command/shellモジュールは実行したいコマンド文字列を引数に取ります。

playbookからの場合、Ansibleモジュールはよく似た方法で実行されます::

    - name: reboot the servers
      action: command /sbin/reboot -t now

バージョン0.8以降では、次の短い構文をサポートします::

    - name: reboot the servers
      command: /sbin/reboot -t now

すべてのモジュールは、技術的にはJSON形式のデータを返しますが、コマンドラインで
使う場合もplaybookで使う場合も、そのことについて多くを知っている必要はありません。
自分でモジュールを記述する場合は、これに注意すれば、モジュールを特定の言語で
記述する必要はありません--あなたは選択できます。

モジュールは `冪等` 、つまり変更する必要がない限り、変更を回避することを意味
します。Ansible playbookを使う場合、追加のタスクを実行するために'ハンドラ'に
通知する形式で、'変更イベント'をトリガできます。

各モジュールのドキュメントには、manコマンドと同様に、ansible-docを使って
コマンドラインからアクセスできます::

    ansible-doc command

    man ansinle.template

さあ、箱からとりだして、Ansibleモジュールライブラリでなにが使えるか見て
みましょう:


.. include:: modules/_list.rst


独自のモジュールを記述する
``````````````````````````

:doc:`moduledev` を参照してください。


.. seealso::

   :doc:`contrib`
       User contributed playbooks, modules, and articles
   :doc:`examples`
       Examples of using modules in /usr/bin/ansible
   :doc:`playbooks`
       Examples of using modules with /usr/bin/ansible-playbook
   :doc:`moduledev`
       How to write your own modules
   :doc:`api`
       Examples of using modules with the Python API
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
