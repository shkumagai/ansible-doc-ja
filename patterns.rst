.. _patterns:

インベントリとパターン
======================

.. イメージ省略

Ansibleはあなたのインフラの複数のシステムに対して、同時に機能します。
これは、デフォルトでは /etc/Ansible/hosts にあるAnsibleのインベントリ
ファイルにリストされたシステムの一部を選択することによって行われます。

.. contents:: `Table of contents`
   :depth: 2
   :backlinks: top

.. _inventoryformat:

ホストとグループ
++++++++++++++++

/etc/Ansible/hosts の書式はINI形式で、このようになります::

    mail.example.com

    [webservers]
    foo.example.com
    bar.example.com

    [dbservers]
    one.example.com
    two.example.com
    three.example.com

ブラケットの中はグループ名です。グループ名は付ける必要はありませんが、付けて
あると便利です。

標準ではないSSHポートを使っているホストがある場合は、ホスト名の後ろにコロンを
付けて書いておけます。任意のSSH設定ファイルに書かれているポートは読まれないので
デフォルトポートで実行されていないには、これらを設定することが重要です::

    badwolf.example.com:5309

静的IPアドレスを持っているが、自分のホストファイルに書かれていない別名を
設定したい場合や、トンネルを介して接続するような場合、このようにできます::

    jumper ansible_ssh_port=5555 ansible_ssh_host=192.168.1.50

上の例で、Ansibleを実行するとホストの別名 "jumper" (実際のホスト名ではないかも
しれない) に対して、アドレス192.168.1.50のポート5555で接続しようとします。

もっと多くのホストを追加するには？ 0.6以降では、多くのホストが次のように似た
パターンなら、各ホスト名を羅列するよりこのように書けます::

    [webservers]
    www[01:50].example.com

1.0以降なら、これにアルファベットの範囲指定も使えます::

    [webservers]
    db-[a:f].example.com

数値パターンの場合、前に付けたゼロは必要に応じて含めることも取り除くことも
できます。範囲指定の場合は含まれます。

1.1以降では、各ホスト毎に接続タイプとユーザを選択することもできます::

    [targets]

    localhost              Ansible_connection=local
    other1.example.com     Ansible_connection=ssh        Ansible_ssh_user=mpdehaan
    other2.example.com     Ansible_connection=ssh        Ansible_ssh_user=mdehaan

インベントリファイルをシンプルに保ちたい場合、これらすべての変数はインベントリ
ファイルの外の 'host_vars' に設定することもできます。


ターゲットを選択する
++++++++++++++++++++

:doc:`examples` のセクションでコマンドラインの使い方は練習しますが、基本的には
次のようになります::

    ansible <pattern_goes_here> -m <module_name> -a <arguments>

例えば::

    ansible webservers -m service -a "name=httpd state=restarted"

:doc:`playbooks` では、これらのパターンはより大きな目的のために使えます。

ともかく、Ansibleを使うには、まず最初にインベントリファイル内のホスト名を
Ansibleに伝える方法を知っておく必要があります。
これは特定のホストまたはホストのグループを指定することによって行われます。

次のパターンは、インベントリファイル内のすべてのホストをターゲットとします::

    all
    *

基本的には、'all'は'*'の別名です。これは特定のホストまたはホスト群にも対処できます::

    one.example.com
    one.example.com:two.example.com
    192.168.1.50
    192.168.1.*

次のパターンは、前述のインベントリファイル内でブラケットヘッダで示されている
一つ以上のグループに対応します::

    webservers
    webservers:dbservers

例えば、phoenixグループに含まれていないすべてのウェブサーバのように、グループを
除外することができます::

    webservers:!phoenix

また、二つのグループの共通部分を指定することもできます::

    webservers:&staging

組み合わせることもできます::

    webservers:dbservers:!phoenix:&staging

変数を使うこともできます::

    webservers:!$excluded:&required

各ホスト名、IP、およびグループはワイルドカードで参照することもできます::

    *.example.com
    *.com

ワイルドカードパターンとグループを同時に混ぜても大丈夫です::

    one*.com:dbservers

簡単ですよね。選択したホストに対して行うことについては :doc:`examples` と、
それから :doc:`playbooks` を参照してください。


ホスト変数
++++++++++

後でplaybookの中で利用されるホストに変数を代入するのは簡単です::

    [atlanta]
    host1 http_port=80 maxRequestsPerChild=808
    host2 http_port=303 maxRequestsPerChild=909


グループ変数
++++++++++++

変数は、グループ全体に一括で適用することもできます::

    [atlanta]
    host1
    host2

    [atlanta:vars]
    ntp_server=ntp.atlanta.example.com
    proxy=proxy.atlanta.example.com


グループのグループとグループ変数
++++++++++++++++++++++++++++++++

グループのグループを作成して、グループに変数を代入することもできます。
これらの変数は /usr/bin/ansible-playbook で使えますが、/usr/bin/ansible では
使えません::

    [atlanta]
    host1
    host2

    [releigh]
    host2
    host3

    [southeast:children]
    atlanta
    releigh

    [sousheast:vars]
    some_server=foo.southeast.example.com
    halon_system_timeout=30
    self_destruct_countdown=60
    escape_pods=2

    [usa:children]
    southeast
    northeast
    southwest
    southeast

.. note:: 訳注
   southeast が重複している。

リストやハッシュのデータを格納する必要がある場合や、インベントリファイルとは
別にホストやグループ固有の変数を保持したい場合は、次のセクションを参照して
ください。


ホストの分割とグループ固有データ
++++++++++++++++++++++++++++++++

.. versionadded:: 0.6

INIファイルに直接格納する変数に加えて、ホストやグループ変数はインベントリ
ファイルに対応する個別のファイルに格納できます。これらの変数ファイルは
YAML形式になっています::

インベントリファイルのパスを仮定します::

    /etc/ansible/hosts

もしホスト名に 'foosball'、グループに 'raleigh' と 'webservers' が指定されている
なら、次の場所にあるYAMLファイル内の変数がホストに利用できます::

    /etc/ansible/group_vars/raleigh
    /etc/ansible/group_vars/webservers
    /etc/ansible/host_vars/foosball

例えばデータセンターでグループ化されたホストがあるとすると、それぞれのデータ
センターではいくつかの異なるサーバを使用しています。'raleigh' グループ用の
グループファイル '/etc/ansible/group_vars/raleigh' の中のデータは次のように
なります::

    ---
    ntp_server: acme.example.org
    database_server: storage.example.org

これはオプショナルの機能なので、これらのファイルは存在しなくても大丈夫です。

Tip: インベントリファイルや変数をgitリポジトリ (または他のバージョン管理) で
保持するのが、インベントリやホスト変数の変更を追跡するためには優れた方法です。

.. versionadded:: 0.5
   システム上に複数のPythonインタプリタがある場合や、Python バージョン2 の
   インタプリタが /usr/bin/python で見つからない場合は、'ansible_python_interpreter'
   というインベントリ変数に使用したいPythonインタプリタのパスを設定してください。


.. seealso::

   :doc:`examples`
       基本的なコマンドの例
   :doc:`playbooks`
       Ansibleの構成管理言語を学ぶ
   `Mailing List <http://groups.google.com/group/Ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
