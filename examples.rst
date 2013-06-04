コマンドラインの例と次のステップ
================================

.. イメージ省略

.. highlight:: bash

以下では `/usr/bin/ansible` でアドホックなタスクの実行の仕方の例を示します。
始めましょう。

構成管理やデプロイの為に、この概念を直接移植した `/ust/bin/ansible-playbook` を
使い方を理解したくなるでしょう。
(これらについて、詳しくは :doc:`playbooks` を参照してください。)

.. contents::
   :depth: 2
   :backlinks: top


並列処理とシェルコマンド
````````````````````````

Atlanta にあるすべてのウェブサーバを、一度に10台ずつ再起動させるために、Ansibleの
コマンドラインツールを使ってみましょう。まず、証明書を覚えさせることができるので、
SSHエージェントを準備します::

    $ ssh-agent bash
    $ ssh-add ~/.ssh/id_rsa

ssh-agent を使いたくない場合や鍵の代わりにパスワードを使ったSSHを使いたい場合は、
``--ask-pass`` (``-k``) を使えますが、ssh-agentだけを使う方がずっといいでしょう。

この場合では、*atlanta* グループ内のすべてのサーバに10並列でコマンドを実行します::

    $ ansible atlanta -a "/sbin/reboot" -f 10

バージョン0.7以降では、デフォルトであなたのユーザアカウントで実行されます。
この振る舞いが好ましくない場合は、"-u username" をつけてください
(0.6以前では、rootがデフォルトでした。ほとんど人がカレントユーザをデフォルトにする
ことを好んだため、このように変更しました)。

別のユーザでコマンドを実行したい場合は、このようにします::

    $ ansible atlanta -a "/usr/bin/foo" -u username

sudoを使ってコマンドを実行したい場合は::

    $ ansible atlanta -a "/usr/bin/foo" -u username --sudo [--ask-sudo-pass]

パスワード無しのsudoを使っていない場合は ``--ask-sudo-pass`` (``-K``) を
使いましょう。これで対話的に使用するパスワードの入力を求められます。
パスワード無しsudoの利用は自動化を簡単にできますが、必須ではありません。

``--sudo-user`` (``-U``) を使うと、root以外のユーザになるためにsudoを使うことも
できます::

    $ ansible atlanta -a "/usr/bin/foo" -u username -U otheruser [--ask-sudo-pass]

さて、これらは基本です。パターンとグループをまだ読んでいないなら、
戻って :doc:`patterns` を読んでください。

前述の ``-f 10`` は、10個の同時プロセスの使用を指定します。通常、コマンドは
``-m`` にモジュール名を取りますが、デフォルトのモジュール名は 'command' なので、
ここでは指定する必要がありません。この後の例では他の :doc:`modules` を実行する
ために ``-m`` を使います。

.. note::
   :ref:`command` モジュールは絶対パスを必要とし、またシェル変数をサポート
   していません。シェルを使ってモジュールを実行することは可能で、パイプや
   リダイレクション演算子も使えます。より詳しい差異については :doc:`modules` の
   ページを参照してください。

:ref:`shell` モジュールの使い方はこのようにします::

    $ ansible releigh -m shell -a 'echo $TERM'

(:doc:`playbooks` とは異なり) Ansible *アドホック* CLIでコマンドを実行する場合は、
シェルのクォーティングルールに特に注意を払ってください。そうしないと、シェルは
Ansibleに渡す前に変数を解釈しません。例えば上の例でシングルクォートの代わりに
ダブルクオートを使うと、あなたが居るマシンで変数が評価されます。

.. note:: 訳注
   訳が怪しい。

これまでのところ、単純なコマンドの実行をデモしてきましたが、ほとんどのAnsible
モジュールは単純なスクリプトのようには動作しません。それらはリモートシステムを
あなたが提示したように変更し、そこでそれに必要なコマンドを実行します。
これは一般的に'冪等性'と呼ばれ、Ansibleのコアデザインのゴールでもあります。
しかし、我々は *アドホック* なコマンドを実行することも同様に重要と考えているので、
Ansibleは両方を簡単にサポートしています。

ファイル転送
````````````

これはコマンドラインの `/usr/bin/ansible` のもうひとつのユースケースです。
Ansibleは、多くのファイルを複数のマシンに並列でSCPできます。

複数のサーバに直接ファイルを転送する方法::

    $ ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"

playbookを使う場合は、これと別のもうひと手間を使って、 ``template`` モジュールを
活用できます。 ``file`` モジュールは、ファイルの所有権とパーミッションを変更でき
ます。これらと同じオプションは、 ``copy`` モジュールに直接渡せます::

    $ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600"
    $ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600 owner=mdehaan group=mdehaan"

``file`` モジュールは ``mkdir -p`` のようにディレクトリを作成できます::

    $ ansible webservers -m file -a "dest=/path/to/c mode=644 owner=mdehaan group=mdehaan state=directory"

同様に、ディレクトリの削除(再帰的に)とファイルの削除をします::

    $ ansible webservers -m file -a "dest=/path/to/c state=absent"


パッケージ管理
``````````````

yumやaptのためのモジューつを使うことができます。ここではyumの例をいくつか示します。

パッケージがインストールされている事を確認するが、アップデートはしない::

    $ ansible webservers -m yum -a "name=acme state=installed"

指定したバージョンのパッケージがインストールされている事を確認する::

    $ ansible webservers -m yum -a "name=acme-1.5 state=installed"

パッケージが最新のバージョンであることを確認する::

    $ ansible webservers -m yum -a "name=acme state=latest"

パッケージがインストールされていないことを確認する::

    $ ansible webservers -m yum -a "name=acme state=removed"

現在のところ、Ansibleにはyumとaptによるパッケージ管理のモジュールだけがあります。
いまは、コマンドラインモジュールを使って他のパッケージをインストールするか、他の
パッケージマネージャのためのモジュールをコントリビュート(これが望ましい！)できます。
情報/詳細についてはメーリングリストに立ち寄ってみてください。


ユーザとグループ
````````````````

'user' モジュールはユーザの作成や既存のユーザアカウントの編集、および削除を簡単に
行えます::

    $ ansible all -m user -a "name=foo password=<crypted password here>"

    $ ansible all -m user -a "name=foo state=absent"

グループやグループのメンバーシップの操作方法を含む、利用可能なすべてのオプションの
詳細については :doc:`modules` セクションを参照してください。


ソース管理からのデプロイ
````````````````````````

webアプリケーションをgitから直接デプロイする方法::

    $ ansible webservers -m git -a "repo=git://foo.example.org/repo.git dest=/srv/myapp version=HEAD"

Ansibleモジュールは変更ハンドラに通知することができるので、コードが変更された際に
Perl/Python/PHP/Rubyをgitから直接デプロイした後にapacheを再起動するような、特定の
タスクを実行するようにAnsibleに伝えることができます。


サービスの管理
``````````````

すべてのwebサーバでサービスが起動していることを確認する::

    $ ansible webservers -m service -a "name=httpd state=started"

代わりに、すべてのwebサーバでサービスを再起動する::

    $ ansible webservers -m service -a "name=httpd state=restarted"

サービスが停止していることを確認する::

    $ ansible webservers -m service -a "name=httpd state=stoped"


バックグラウンド操作の時間制限
``````````````````````````````

実行時間の長い操作をバックグラウンドで実行させ、後でその状態をチェックできます。
同一のジョブIDをすべてのホストの同じタスクに付与するので、追跡し損なうことは
ありません。ホストをキックして放っておきたい場合は、次のようにします::

    $ ansible all -B 3600 -a "/usr/bin/long_running_operation --do-stuff"

後でジョブの状態を確認したいなら、こうできます::

    $ ansible all -m async_status -a "jid=123456789"

ポーリングは組み込まれているので、このようにします::

    $ ansible all -B 1800 -P 60 -a "/usr/bin/long_running_operation --do-stuff"

上の例は、"最大30分 (``-B``: 30*60=1800) 実行"、60秒毎にポーリングを意味します。

任意のマシンでポーリングが実行されるよりも前にすべてのジョブを開始するように
できるので、ポーリングモードはスマートです。すべてのジョブがすぐに開始される
ようにしたい場合は、 ``--forks`` を充分に高い値にしてください。
制限時間(秒)を使い果たした後 (``-B``)、リモートノード上のプロセスが終了します。
通常、バックグラウンド化するのは実行時間の長いシェルコマンドやソフトウェアの
アップデートだけでしょう。copyモジュールをバックグラウンド化しても、ファイル転送は
バックグラウンド化されません。
:doc:`playbooks` はポーリングもサポートしていて、そのための簡単な構文があります。


選択したホストを制限する
````````````````````````

.. versionadded:: 0.7

管理するために選択したホストを、'--limit' パラメータや 'batch' (または'range')
セレクタを使って制限を加えることができます。

前述したとおり、パターンで一つ以上のグループの選択されたホストを連結できます::

    $ ansible webservers:dbservers -m command -a "/bin/foo xyz"

これは "or" 条件です。さらに選択対象を制約した場合は --limit を使います。これは
``ansible-playbook`` でも動作します::

    $ ansible webservers:dbservers -m command -a "/bin/foo xyz" --limit region

バージョン0.9以降であれば、他のホストパターンや制限するための値と同様に、
";"、":" および "," で区切ることができます。

今度は範囲選択について説明しましょう。'datacenter' グループに1000台のサーバがあるが、
一度に１つをターゲットとしたい、と仮定します。これも簡単です::

    $ ansible webservers[0-99] -m command -a "/bin/foo xyz"
    $ ansible webservers[100-199] -m command -a "/bin/foo xyz"

これはwebserversグループにあるホストエントリから、最初の100台を選択し、それから
次の100台を選択します (それらの名前やIPアドレスが何であるかは関係ありません)。

これらの方法のどちらも同時に使えますし、--limit パラメータに範囲を渡すこともできます。


構成とデフォルト
````````````````

.. versionadded:: 0.7

Ansibleには設定を調整したり、様々なコマンドラインフラグをいちいち渡さなくても
済むようにできる、追加の構成ファイルがあります。Ansibleは次の順序で構成ファイルを
検索し、最初に見付けたファイルを使います。

1. 環境変数 ``ANSIBLE_CONFIG`` で指定されたファイル
2. カレントワーキングディレクトリ内の ``ansible.cfg`` (バージョン0.8以上)
3. ``~/.ansible.cfg``
4. ``/etc/ansible/ansible.cfg``

これらをソースから実行する場合、サンプルの構成ファイルは examples/ ディレクトリに
あります。RPMは構成ファイルを自動的に /etc/ansible/ansibe.cfg にインストールします。

.. seealso::

   :doc:`modules`
       利用できるモジュール一覧
   :doc:`playbooks`
       構成管理やデプロイにAnsibleを利用する
