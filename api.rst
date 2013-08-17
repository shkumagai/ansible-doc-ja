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

引数'--host <hostname>'（<hostname>のところは前述のホスト）を付けて実行した場合、
テンプレートやplaybookが使えるように、スクリプトは空のハッシュ/辞書JSON、または
変数のハッシュ/辞書を返さなければなりません。変数の返却はオプションなので、そう
したくない場合は空のハッシュ/辞書を返すようにします::

    {
        "favcolor"   : "red",
        "ntpserver"  : "wolf.example.com",
        "monitoring" : "pack.example.com"
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

Amazon Web Service EC2 を使っている場合、インベントリファイルを維持するのは最善の
アプローチではないかも知れません。なぜなら、
`EC2外部インベントリ <https://raw.github.com/ansible/ansible/devel/plugins/inventory/ec2.py>`_
スクリプトが使えます。

次のいずれかの方法でこのスクリプトを使えます。最も簡単な方法は、Ansibleのコマンド
ラインオプション ``-i`` を使い、スクリプトのパスを指定することです。

    ansible -i ec2.py -u ubuntu us-east-1d -m ping

2つ目の方法は `/etc/ansible/hosts` にスクリプトをコピーし、 `chmod +x` します。
`ec2.ini <https://raw.github.com/ansible/ansible/devel/plugins/inventory/ec2.ini>`_
ファイルも `/etc/ansible/ec2.ini` にコピーする必要があります。
それから、Ansibleを通常どおりに実行できます。

APIにうまくAWSを呼び出しさせるためには、Boto (AWSのPythonインターフェース)を設定
する必要があります。
`様々な方法 <http://docs.pythonbot.org/en/latest/boto_config_tut.html>`_ が
ありますが、最も簡単なのは単に環境変数を２つエクスポートするだけです:

    export AWS_ACCESS_KEY_ID='AK123'
    export AWS_SECRET_ACCESS_KEY='abc123'

設定が正しいことを確認するために、自分自身でスクリプトをテストすることができます

    cd plugins/inventory
    ./ec2.py --list

しばらくすると、すべてのリージョンを横断したEC2全体のインベントリを含むJSONが表示
されるはずです。

各リージョンはそれぞれのAPIを呼び出す必要があるので、もしリージョンの小さなセット
だけを使っている場合 ``ec2.ini`` を自由に編集して関心のあるリージョンだけを記入
してください。 ``ec2.ini`` にはキャッシュ制御や宛先変数を含む、他の設定オプション
があります。

根本的には、インベントリファイルは単に某かの名前から宛先へのマッピングです。
デフォルトの ``ec2.ini`` 設定は、EC2外部（例えばあなたのラップトップ）から
Ansibleを実行するために構成されています。もしEC2上からAnsibleを実行している場合、
内部DNSとIPアドレスのほうが、パブリックDNSよりも理に適っているかも知れません。
この場合、 ``ec2.ini`` の ``destination_variable`` をインスタンスのプライベート
DNS名になるように変更できます。インスタンスにアクセスする唯一の手段がプライベート
IPアドレスを介してのみのVPCで、内部のプライベートサブネットでAnsibleを実行して
いる場合、これは特に重要です。VPCインスタンスの場合 ``ec2.ini`` の中の
`vpc_destination_variable` が、あなたのユースケースに対して最も理に適った
`boto.ec2.instance 変数 <http://docs.pythonboto.org/en/latest/ref/ec2.html#module-boto.ec2.instance>`_
を使う手段を提供します。

EC2外部インベントリはいくつかのグループからインスタンスへのマッピングを提供します:

インスタンスID
  インスタンスIDが一意であるため、これらは１つのグループです。
  例
  ``i-00112233``
  ``i-a1b1c1d1``

リージョン
  AWSリージョンの中のすべてのインスタンスのグループ。
  例
  ``us-east-1``
  ``us-west-2``

アベイラビリティゾーン
  アベイラビリティゾーン内のすべてのインスタンスのグループ。
  例
  ``us-east-1a``
  ``us-east-1b``

セキュリティグループ
  インスタンスは１つまたは複数のセキュリティグループに属しています。グループは
  各セキュリティグループ毎に、英数字以外のすべての文字とダッシュ (-) をアンダー
  スコア (_) に置き換えて作られます。各グループには ``security_group_`` の
  プレフィックスが付きます。
  例
  ``security_group_default``
  ``security_group_webservers``
  ``security_group_Pete_s_Fancy_Group``

タグ
  各インスタンスは、それに関連付けられたタグと呼ばれる、様々なキー/値のペアを
  持てます。なんでも大丈夫ですが、最も一般的なタグのキーは'Name'です。それぞれの
  キー/値のペアは ``tag_KEY_VALUE`` の書式で、特殊な文字はアンダースコアに変換
  された、インスタンスに固有のグループです。
  例
  ``tag_Name_Web``
  ``tag_Name_redis-master-001``
  ``tag_aws_cloudformation_logical-id_WebServerGroup``

Ansibleが指定されたサーバとやり取りをする際、EC2インベントリスクリプトは、再度
``--host HOST`` オプション付きで呼び出されます。これはインスタンスIDを取得する
ためにインデックスキャッシュのホストを検索し、その特定のインスタンスに関する
情報を取得するためにAWSへのAPI呼び出しを行います。その後、あなたのplaybook変数
として利用可能な、インスタンスに関する情報になります。各変数にはプレフィックス
``ec2_`` が付きます。利用可能な変数の一部は、次のとおりです:

- ec2_architecture
- ec2_description
- ec2_dns_name
- ec2_id
- ec2_image_id
- ec2_instance_type
- ec2_ip_address
- ec2_kernel
- ec2_key_name
- ec2_launch_time
- ec2_monitored
- ec2_ownerId
- ec2_placement
- ec2_platform
- ec2_previous_state
- ec2_private_dns_name
- ec2_private_ip_address
- ec2_public_dns_name
- ec2_ramdisk
- ec2_region
- ec2_root_device_name
- ec2_root_device_type
- ec2_security_group_ids
- ec2_security_group_names
- ec2_spot_instance_request_id
- ec2_state
- ec2_state_code
- ec2_state_reason
- ec2_status
- ec2_subnet_id
- ec2_tag_Name
- ec2_tenancy
- ec2_virtualization_type
- ec2_vpc_id

``ec2_security_group_ids`` と ``ec2_security_group_names`` は、どちらもカンマで
区切られたすべてのセキュリティグループのリストです。各EC2タグは ``ec2_tag_KEY``
形式の変数です。

インスタンスで利用可能な変数の、完全なリストを表示するにはスクリプトそれ自身を
実行します::

    cd plugins/inventory
    ./ec2.py --host ec2-12-12-12-12.compute-1.amazonaws.com


例: OpenStack インベントリスクリプト
````````````````````````````````````

EC2モジュールと同等の内容をここで詳しく説明はしませんが、pluginsディレクトリに
は、OpenStack Compute の外部インベントリのソースもあります。OpenStack の
Grizzly リリース以降が必要です。
使い方についてはモジュールのソースのインラインコメントを参照してください。


Callbackプラグイン
------------------

Ansibleは、外部のイベントに対応するコードを通じて設定を行えます。これはログの強化
や、外部ソフトウェアシステムのシグナル、さらに（本当に）効果音を鳴らせます。
いくつかの例がpluginsディレクトリに含まれています。


接続タイププラグイン
--------------------

デフォルトでは、Ansibleは 'paramiko' SSH、ネイティブSSH(単に'ssh'と呼ばれます)、
および'local'接続タイプが同梱されています。リリース0.8では'fireball'と呼ばれる
速度を高めた接続タイプを追加しました。これらはすべてplaybookや/usr/bin/ansible
の中でリモートマシンとどうやり取りする方法を決めるために使えます。
これらの接続タイプの基本は、'getting started'セクションの中でカバーしています。
他のトランスポート（SNMP？メッセージバス？それとも伝書鳩？）をサポートするため
にAnsibleの拡張をするのは、既存のモジュールのいずれかのフォーマットをコピーして
接続プラグインのディレクトリにドロップするのと同じくらい簡単です。


参照プラグイン
--------------

"with_fileglob"や"with_items"のような言語構造は、参照プラグインを使って実装されて
います。他のプラグインタイプのように自分で書けます。


Varsプラグイン
--------------

playbookは'vars'プラグインを介して、'host_vars'や'group_vars'の機能を構築します。
これらはインベントリ、playbookおよびコマンドラインから何も渡されていないAnsibleの
実行に追加の変数データを入れることができます。変数はインベントリからも返すことが
できるので、ほとんどの場合はvars_pluginsを書いたり、理解する必要はないことに注意
してください。


フィルタープラグイン
--------------------

Jinja2テンプレートで（デフォルトで、to_yamlやto_json等のフィルタが用意されて
いますが）もっとJinja2フィルタを利用した場合は、filterプラグインを書いて拡張
できます。


分散プラグイン
--------------

.. versionadded: 0.8

プラグインはPythonのsite_packages (Ansibleに同梱されているもの) と、設定された
pluginsディレクトリ -- デフォルトでは /usr/share/ansible/plugins -- のそれぞれ
のプラグインタイプのサブディレクトリの両方から読み込まれます::

    * action_plugins
    * lookup_plugins
    * callback_plugins
    * connection_plugins
    * filter_plugins
    * vars_plugins

このパスを変更するには、Ansibleの設定ファイルを編集します。

また、プラグインはplaybookのトップディレクトリからの相対パスで、上に示したものと
同じ名前のサブディレクトリに含めることができます。

.. seealso::

   :doc:`modules`
       List of built-in modules
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
