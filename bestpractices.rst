ベストプラクティス
==================

.. イメージ省略

ここにはansibleを活用するためのヒントがあります。

`ansible-examples リポジトリ <https://github.com/ansible/ansible-examples>`_
ではこれらベストプラクティスを例示するプレイブックの例を見つけられます。

.. contents::
   :depth: 2
   :backlinks: top

コンテンツ編成
++++++++++++++

以下のセクションでは、コンテンツを構成するために使える方法のうちのひとつを
示しています。ansibleの使い方は、自分のニーズに適合しているべきですから、
このアプローチを自分の使い方に合うように、自由に変更や整理してください。


ディレクトリ構成
````````````````

ディレクトリのトップレベルに、このようなファイルやディレクトリを含めます::

    production            # 本番サーバ用インベントリファイル
    stage                 # ステージングサーバ用インベントリファイル

    group_vars/
       group1             # 特定のグループの変数を、ここで代入
       group2             # ""
    host_vars/
       hostname1          # システムに特定の変数が必要な場合は、ここに置く
       hostname2          # ""

    site.yml              # マスタープレイブック
    webservers.yml        # webserver層のプレイブック
    dbservers.yml         # dbserver層のプレイブック

    common/               # この階層は "role" を表す
        tasks/            #
            main.yml      #  <-- タスクファイルは正当であればより小さなファイルをインクルードできる
        handlers/         #
            main.yml      #  <-- ハンドラファイル
        templates/        #  <-- テンプレートで使用するファイル
            ntp.conf.j2   #  <------- テンプレートファイル名は .j2 で終わる
        files/            #
            bar.txt       #  <-- コピーで使用するファイル

    webtier/              # 上記の"common"と同様に、web層のroleを表す
    monitoring/           # ""
    fooapp/               # ""


インベントリの整理方法 ステージ vs 本番
```````````````````````````````````````

この例では、 *production* ファイルはすべての本番ホストのインベントリを含みます。
もちろん外部のデータソースからインベントリを引くこともできますが、これは単純に
基本的な例です。ホスト(役割)の目的および地理またはデータセンターの場所に基いて
グループを定義します::

    # file: production

    [atlanta-webservers]
    www-atl-1.example.com
    www-atl-2.example.com

    [boston-webservers]
    www-bos-1.example.com
    www-bos-2.example.com

    [atlanta-dbservers]
    db-atl-1.example.com
    db-atl-2.example.com

    [boston-dbservers]
    db-bos-1.example.com

    # すべての地域の webservers
    [webservers:children]
    atlanta-webservers
    boston-webservers

    # すべての地域の dbserver
    [dbservers:children]
    atlanta-dbservers
    boston-dbservers

    # atlanta 地域のすべてのホスト
    [atlanta:children]
    atlanta-webservers
    atlanta-dbservers

    # boston 地域のすべてのホスト
    [boston:children]
    boston-webservers
    boston-dbservers


グループ変数とホスト変数
````````````````````````

さて、グループは編成を行うのに適していますが、グループが適しているのはそれが
すべてではありません。グループに変数を代入することもできるんです！例えば、
atlantaは自身のNTPサーバを持っているので、ntp.confを設定するときはそれを使う
べきです。では、それらを設定してみましょう::

    ---
    # file: group_vars/atlanta
    ntp: ntp-atlanta.example.com
    backup: backup-atlanta.example.com


変数は、どこかの地理情報だけのものではありません。webserverは、dbserverにとって
は意味を成さないをいくつか持っているかも知れません::

    ---
    # file: group_vars/webservers
    apacheMaxRequestsPerChild: 3000
    apacheMaxClients: 900

もし何らかのデフォルトの値や、普遍的にtrueの値がある場合、それを group_vars/all
と呼ばれるファイルに記述します::

    ---
    # file: group_vars/all
    ntp: ntp-boston.example.com
    backup: backup-boston.example.com

host_varsファイルには、システム内の特定のハードウェア用の変数を定義できますが、
これは必要でなければ使うことはありません::

    ---
    # file: host_vars/db-bos-1.example.com
    foo_agent_port: 86
    bar_agent_port: 99


トップレベルのプレイブックは役割で分割する
``````````````````````````````````````````

site.ymlでは、インフラ全体を定義するplaybookが含まれています。 `非常に`
短いことに注意してください。これは他のplaybookをインクルードしているだけ
だからです。playbookはplayのリスト以外の何者でもない、ということを覚えて
下さい::

    ---
    # file: site.yml
    - include: webservers.yml
    - include: dbservers.yml

(同じトップレベルにある) webservers.ymlなどのファイルでは、単純にwebservers
グループによって実行されるroleに、webserversグループの設定をマッピングします。
これも信じられないほど短いことに気づきます。例えば::

    ---
    # file: webservers.yml
    - hosts: webservers
      tasks:
        - include: common/tasks/main.yml tags=common
        - include: webtier/tasks/main.yml tags=webtier
      handlers:
        - include: common/handlers/main.yml
        - include: webtier/handlers/main.yml


roleに対するタスクとハンドラをまとめる
``````````````````````````````````````

このファイルは、ホストに対して彼らが果たすroleをマッピングしているだけです。
さて、以下はタスクファイルの例ですが、これがどのように機能するかを説明します。
ここではcommon roleはNTPをセットアップしますが、必要ならそれ以外のことも
可能です::

    ---
    # file: common/tasks/main.yml

    - name: be sure ntp is installed
      yum: pkg=ntp state=installed
      tags: ntp

    - name: be sure ntp is configured
      template: src=common/templates/ntp.conf.j2 dest=/etc/ntp.conf
      notify:
        - restart ntpd
      tags: ntp

    - name: be sure ntpd is running and enabled
      service: name=ntpd state=running enabled=yes
      tags: ntp

これはハンドラファイルの例です。確認ですが、ハンドラは特定のタスクが変更を
報告した場合にだけ発火し、各playの最後に実行されます::

    ---
    # file: common/handlers/main.yml
    - name: restart ntpd
      service: name=ntpd state=restarted


この構成でできること(例)
````````````````````````

これは基本的な組織構造です。

さて、このレイアウトでいったいどんなユースケースが可能でしょうか？たくさん！
もしインフラ全体を再構成したいなら、これだけです::

    ansible-playbook -i production site.yml

全てのNTPを再構成するのはどうでしょう？ 簡単です::

    ansible-playbook -i production site.yml --tags ntp

webserverだけを再構成するのは？::

    ansible-playbook -i production webservers.yml

Bostonのwebserverだけなら？::

    ansible-playbook -i production webservers.yml --limit boston

最初の10台だけと、その次の10台では？::

    ansible-playbook -i production webservers.yml --limit boston[0-10]
    ansible-playbook -i production webservers.yml --limit boston[10-20]

そしてもちろん、単純にアドホックなものも可能です::

    ansible -i production -m ping
    ansible -i production -m command -a '/sbin/reboot' --limit boston

それから、いくつか便利なコマンドがあります (1.1以上が必要です)::

    # confirm what task names would be run if I ran this command and said "just ntp tasks"
    ansible-playbook -i production webservers.yml --tags ntp --list-tasks

    # confirm what hostnames might be communicated with if I said "limit to boston"
    ansible-playbook -i production webservers.yml --limit boston --list-hosts


デプロイ vs 設定のオーガナイズ
``````````````````````````````

上記のセットアップは、典型的なOS設定のトポロジーを作ります。多階層のデプロイを
行う場合、アプリケーションを展開するための階層間を跨ぐplaybookが追加で必要に
なります。その場合、'site.yml'は'deploy_exampledotcom.yml'のようなplaybookに
よって拡張できるが、それでも通常の概念は適用できます。

ansibleは同じツールを使ってデプロイと設定が可能なので、グループを再利用したり、
アプリケーションのデプロイとは別のプレイブックで、OSの設定を保持するのに
適しています。


ステージ vs 本番
++++++++++++++++

また、前述したように、ステージ (またはテスト) 環境と本番環境を個別に保つための
よい方法は、ステージと本番のインベントリファイルを分けて使うことです。この方法
では対象とする方を -i を付けて選びます。全てを一つのファイルに入れておくと、
予期しないことが起きる原因になります！


ローリングアップデート
++++++++++++++++++++++

'serial'キーワードを理解しましょう。webserverのファームをアップデートする場合、
一度の処理でアップデートするマシンの数を制御するために使いたいと思うはずです。


常に状態を表示する
++++++++++++++++++

'state'パラメータは多くのモジュールのオプションです。'state=present' か
'state=absent' のどちらでも、特に追加の状態をサポートしているモジュールなどは、
状態を明確にするために常にplaybookの中にパラメータを残しておくのが最善です。


role毎のグループ
++++++++++++++++

システムは複数のグループに属することができます。 ref:`pattern` を参照してください。
*webservers* や *dbservers* のようなものにちなんで名付けたグループが例のなかで
繰り返されるのは、それがとても協力な概念だからです。

これによって、playbookはroleに基づいてマシンを対象にしたり、同様にグループ変数の
仕組みを利用して、role固有の変数を割り当てられます。


オペレーティングシステムとディストリビューション毎の差異
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

2つの異なるオペレーティングシステム間で異なるパラメータを扱う場合、これをうまく
扱うための最良の方法は、group_byモジュールを使うことです。

これはインベントリファイルにグループが定義されていなくても、特定の条件に一致する
ホストのグループを動的に作ります::

   ---

   # talk to all hosts just so we can learn about them

   - hosts: all
     tasks:
        - group_by: key=${ansible_distribution}

   # now just on the CentOS hosts...

   - hosts: CentOS
     gather_facts: False
     tasks:
        - # tasks that only happen on CentOS go here

グループ固有の設定が必要な場合も、これが使えます。例えば::

    ---
    # file: group_vars/all
    asdf: 10

    ---
    # file: group_vars/CentOS
    asdf: 42

上記の例では、CentOSマシンは'asdf'の値として'42'を取得し、そうでないマシンは
10を取得します。


ansibleモジュールをplaybookにバンドルする
+++++++++++++++++++++++++++++++++++++++++

.. versionadded:: 0.5

playbookが、YAMLファイルからの相対パスで "./library" ディレクトリを持つ場合、
このディレクトリは、自動的にansibleモジュールのパスに入るモジュールを追加する
ために使えます。
これはplaybookとモジュールを一緒に維持するのに素晴らしい方法です。


空白とコメント
++++++++++++++

物事を区切るための十分な空白の使用、およびコメント('#'から始まります)の使用が
推奨されています。


常にタスクに名前を付ける
++++++++++++++++++++++++

特定のタスクで、行われていることの代わりにその理由の説明を提供することが
推奨されていますが、'name' を付けないでおくことも可能です。
playbookが実行されているときに、この名前が表示されます。


シンプルに保つ
++++++++++++++

それが単純にできる場合は、単純にやりましょう。ansibleのすべての機能を一度に
使おうとしないでください。自分に役に立つものを使いましょう。
たとえば、外部のインベントリファイルも使いながら、'vars'、'vars_files'、
'vars_prompt' および '--extra-vars'を一度に使う必要は、おそらくないでしょう。


バージョン管理
++++++++++++++

バージョン管理を使いましょう。playbookとインベントリファイルをgit(または他の
バージョン管理システム)に保存し、変更したらコミットをしてください。
これはあなたがいつ、どんな理由でルールを変更したかを記述する履歴追跡を持って
インフラを自動化する方法です。


.. seealso::

   :doc:`YAMLSyntax`
       Learn about YAML syntax
   :doc:`playbooks`
       Review the basic playbook features
   :doc:`modules`
       Learn about available modules
   :doc:`moduledev`
       Learn how to extend Ansible by writing your own modules
   :doc:`patterns`
       Learn about how to select hosts
   `Github examples directory <https://github.com/ansible/ansible/tree/devel/examples/playbooks>`_
       Complete playbook files from the github project source
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
