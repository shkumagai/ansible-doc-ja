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
