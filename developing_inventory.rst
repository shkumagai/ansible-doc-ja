動的インベントリソースの開発
============================

.. contents:: トピックス
   :local:

:doc:`intro_dynamic_inventory` で説明したように、Ansibleはクラウドのソースを含め
動的ソースからインベントリ情報を取得できます。

新しいものはどうやって書けば？

簡単です！適切な引数を与えたら適切なJSONを返すことができるスクリプト、
またはプログラムを作るだけです。
これはどんな言語でもできます。

.. _inventory_script_conventions:

スクリプトの規約
````````````````

外部ノードスクリプトを ``--list`` 引数だけで実行した場合、
スクリプトは管理するすべてのグループのJSONハッシュ/辞書を返す必要があります。
それぞれのグループの値は、このように、それぞれホスト/IP、潜在的な子グループ、
潜在的なグループ変数のリストを含むハッシュ/辞書、または単純なホスト/IPアドレスの
リストでなければいけません::

    {
        "databases"   : {
            "hosts"   : [ "host1.example.com", "host2.example.com" ],
            "vars"    : {
                "a"   : true
            }
        },
        "webservers"  : [ "host2.example.com", "host3.example.com" ],
        "atlanta"     : {
            "hosts"   : [ "host1.example.com", "host4.example.com", "host5.example.com" ],
            "vars"    : {
                "b"   : false
            },
            "children": [ "marietta", "5points" ],
        },
        "marietta"    : [ "host6.example.com" ],
        "5points"     : [ "host7.example.com" ]
    }

.. versionadded: 1.0

バージョン1.0以前では、上記のwebservers、marietta、および5pointsグループのように、
それぞれホスト名/IPアドレスのリストだけを持てます。

引数 ``--host <hostname>`` （<hostname>のところは前述のホスト）を付けて実行した場合、
テンプレートやplaybookが使えるように、スクリプトは空のハッシュ/辞書JSON、または
変数のハッシュ/辞書を返さなければなりません。変数の返却はオプションなので、
そうしたくない場合は空のハッシュ/辞書を返すようにします::

    {
        "favcolor"   : "red",
        "ntpserver"  : "wolf.example.com",
        "monitoring" : "pack.example.com"
    }

.. _inventory_script_tuning:

外部インベントリスクリプトのチューニング
````````````````````````````````````````

.. versionadded:: 1.3

上で説明したストックインベントリスクリプトシステムはAnsibleの全てのバージョンで
機能しますが、特にそれがリモートホストに対する高コストなAPI呼び出しを含む場合には、
全てのホストに対して ``--host`` を呼び出すのはハイコストです。
Ansible 1.3以降では、インベントリスクリプトが "_meta" というトップレベル要素を返す場合、
一回のインベントリスクリプト呼び出しで、ホスト変数の全てを返すことができます。
このmeta要素に"hostvars"の値を含む場合、インベントリスクリプトはホスト毎に
``--host`` を付けて呼び出されることはありません。
この結果、多数のホストに対するパフォーマンスが大幅に向上し、またインベントリスクリプトの
実装はクライアント側のキャッシュが用意になります。

トップレベルのJSON辞書に追加されるデータは次のようになります::

   {
      # 前述したインベントリスクリプトの結果はここ
      # ...

      "_meta" : {
         "hostvars" : {
            "moocow.example.com"     : { "asdf" : 1234 },
            "llama.example.com"      : { "asdf" : 5678 },
         }
      }

   }

.. seealso::

   :doc:`developing_api`
       Python API to Playbooks and Ad Hoc Task Execution
   :doc:`developing_modules`
       How to develop modules
   :doc:`developing_plugins`
       How to develop plugins
   `Ansible Tower <http://ansible.com/ansible-tower>`_
       REST API endpoint and GUI for Ansible, syncs with dynamic inventory
   `Development Mailing List <http://groups.google.com/group/ansible-devel>`_
       Mailing list for development topics
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
