==========
 始めよう
==========

.. contents:: contents:
   :depth: 1


必要条件
========

Ansibleの要件は本当に最小限です。

AnsibleはPython2.6以上向けに書かれています。もし"Enterprise Linux"の類で
Python2.5を使っているなら、2.6の追加方法もお見せします。LinuxやOS Xのより
新しいバージョンではすでに2.6以上が入っています。

Python2.6+に加えて、次のPythonモジュールが必要です(pip、またはOSのパッケージ
マネージャでちょっと違う名前のものをインストールします):

- paramiko
- PyYAML
- jinja2

RHELやCentOS5を使っている場合、デフォルトはPython2.4ですが、Python2.6は
簡単にインストールできます。 `EPELを使うと`_ これらの依存パッケージは
次のようにインストールします::

    $ yum install python26 python26-PyYAML python26-paramiko python26-jinja2

管理されるノードでは、Python2.4以降だけが必要ですが、Python2.6未満を使って
いる場合はこれも必要です:

- python-simplejson

.. note::

   Ansibleの"生"の (コマンドを簡単かつ汚いやり方で実行する) モジュールや
   スクリプトモジュールはこれを必要としません。そのため、技術的には"生"の
   モジュールを使えばpython-simplejsonをインストールするためにAnsibleを
   使うことができ、その後、その他すべてを使えるようになります (少し先に
   話が飛びますが) 。

.. note::

   Python3はPython2とは少々異なる言語で、(Ansibleを含め)ほとんどのPython
   モジュールは切り替え終わっていません。しかしいくつかのLinuxディストリ
   ビューション (GentooやArch) は、Python2.Xインタプリタがデフォルトで
   インストールされないことがあります。それらのシステムでは自分で
   インストールして、インベントリの変数 ``'ansible_python_interpreter'`` に
   自分のpython2.Xへのポインタの設定が必要です。Redhat Enterprise Linux、
   CentOS、FedoraそしてUbuntu等のディストリビューションはPython2.Xインタプリタが
   デフォルトでインストールされているので、これには該当しません。
   これはまた、ほぼすべてのUnixシステムにも当てはまります。
   Python2.Xをインストールするためにこれらリモートシステムを起動する必要が
   ある場合は、"生"のモジュールを使ってリモートで起動できます。




Ansibleの入手
=============

最新の機能をすべて使うことに興味があるなら、チェックアウトしたGitの開発ブランチ
を最新に保ちたいと思うでしょう。これはまたプロジェクトへ貢献し返すのも、最も
簡単になります。

ソースからインストールする手順は以下のとおりです。

Ansibleのリリースサイクルは１ヶ月程度です。この短いリリースサイクルによって、
どのようなバグも一般的にはstableブランチへのバックポートを維持するのではなく
次のリリースで修正されます。

もしあなたがGithubアカウントを持っているなら、Githubのプロジェクトをフォロー
したいと思うでしょう。ここは我々がバグや機能のアイデアを共有するための
issueトラッカーを維持する場所でもあります。


チェックアウトしたソースから実行する
------------------------------------

チェックアウトしたソースからAnsibleを実行するのは簡単なことで、root権限は
必要ありません::

    $ git clone git://github.com/ansible/ansible.git
    $ cd ./ansible
    $ source ./hacking/env-setup

オプションで ``/etc/ansible/hosts`` 以外のインベントリファイルを指定できます ::

    $ echo "127.0.0.1" > ~/ansible_hosts
    $ export ANSIBLE_HOSTS=~/ansible_hosts

インベントリファイルについては、マニュアルの後の部分で読むことができます。

さぁ、試してみましょう ::

    $ ansible all -m ping --ask-pass


Make install
------------

まだAnsibleがパッケージ化されていないディストリビューションで作業する場合に、
``make install`` を使ってAnsibleをインストールできます。
これは ``python-distutils`` を通して行われます ::

    $ git clone git://github.com/ansible/ansible.git
    $ cd ./ansible
    $ sudo make install


pipの場合
---------

あなたはPythonデベロッパーですか？

Ansibleはpipでもインストールできますが、その場合、オプションのモードで利用する
他の依存ライブラリをインストールするか確認します ::

    $ sudo easy_install pip
    $ sudo pip install ansible

``virtualenv`` を使う読者は ``virtualenv`` の下にAnsibleをインストールすること
もできます。Ansibleをインストールするために直接 ``easy_install`` を使わないで
ください。


RPMの場合
---------

最後のAnsibleリリースのRPSが、EPEL6と現在サポートされているFedoraディストリ
ビューションで利用できます。Ansible自体はPython2.4以降が含まれている以前の
オペレーティング・システムを管理できます。

RHELやCentOSを使用していて、まだ設定していない場合は `EPELを設定します`_ ::

    # install the epel-release RPM if needed on CentOS, RHEL, or Scientific Linux
    $ sudo yum install ansible

あなたが配布およびインストールするRPMをビルドするために ``make rpm`` コマンドを
使うこともできます。 ``rpm-build`` 、 ``make`` および ``python2.x-devel`` が
インストールされていることを確認してください ::

    $ git clone git://github.com/ansible/ansible.git
    $ cd ./ansible
    $ make rpm
    $ sudo rpm -Uvh ~/rpmbuild/ansible-*.noarch.rpm


RHELおよびCentOS5のためのPython2.6 EPELの説明
=============================================

これらのディストリビューションは、デフォルトでPython2.6が入っていませんが、
簡単にインストール可能です。

.. note:: 訳注

   このセクションのレベル間違ってないか？


MacPortsの場合
--------------

OS X向けポーティングはMacPortsで利用可能で、MacPortsからAnsibleの安定バージョンを
インストールする (これは推奨する方法です) ために、これを実行します::

    $ sudo port install ansible

Gitでチェックアウトしたソースから、MacPortsシステムを通じて最新のビルドを
インストールしたい場合は、以下のとおりに実行します::

    $ git clone git://github.com/ansible/ansible.git
    $ cd ./ansible/packaging/macports
    $ sudo port install

MacPortsでのPortfileの使用に関する詳しい情報は、< http://www.macports.org > の
ドキュメントを参照してください。


UbuntuとDebian
--------------

Ubuntu向けビルドは `このPPA`_ のものが利用可能です。

Debian/Ubuntu 向けパッケージのレシピも、チェックアウトしたソースからビルドできます
::

    $ make debian

Gentoo、Archおよびその他
------------------------

Gentoo eBuildはportageに含まれており、まもなくバージョン1.0.0になります ::

    $ emerge ansible

ArchのPKGBUILDは AUR_ にあるものが利用可能です。ArchにPython3がインストール
されている場合は、 ``python`` を ``python2`` にシンボリックリンクしたくなるかも
しれません ::

    $ sudo ln -sf /usr/bin/python2 /usr/bin/python

``python`` が ``python3`` を指しているホストでのために ``'ansible_python_interpreter'``
インベントリ変数の設定も必要です。そうすれば管理されるノードで正しくpythonを
見つけることができます。


タグ付きリリース
----------------

リリースのtarボールは、ansible.cc のページにあるものが利用可能です。

- `Ansible/downloads`_

これらのリリースは、gitリポジトリでもリリースバージョンでタグ付けされています。


ParamikoとネイティブSSHの選択
=============================

デフォルトでは、AnsibleはSSH経由で管理対象ノードとやり取りを行うためにParamikoを
使用しています。Paramikoは高速で非常に透過的に動作し、設定の必要がなく、ほとんど
のユーザに適しています。しかし、一部の人達が使いたいような先進的な機能をサポート
していません。

*バージョン 0.5 から*

あなたが (Kerberos対応のSSHやジャンプホストのような) 先進的な機能を活用したい
なら、任意のAnsibleコマンドに ``'--connection=ssh'`` を渡すか、環境変数
``ANSIBLE_TRANSPORT`` に'ssh'を設定します。これでAnsibleは代わりにopensshを
使うようになります。

``ANSIBLE_SSH_ARGS`` が設定されていない場合、Ansibleはデフォルトでいくつか
気の利いたControlMasterオプションを使おうとします。この環境変数を上書きすること
は自由ですが、転送のパフォーマンスを確保するためにはControlMasterオプションを
渡す必要があります。ControlMasterを使うと、どちらの転送もほぼ同じ速度です。
ControlMasterを使わないと、バイナリのssh転送はかなり遅くなります。

このどれもがあなたにとって有意でなければ、デフォルトのparamikoオプションで
恐らく大丈夫でしょう。


はじめてのコマンド
==================

さて、Ansibleがインストールできたので、今度は試す番です:

``/etc/ansible/hosts`` を編集 (または作成) し、 ``authrized_keys`` にあなたの
SSH鍵を持っている一つまたはそれ以上のリモートホストを追加します::

    192.168.1.50
    aserver.example.org
    bserver.example.org

何度もパスワードを入力しないで済むように、SSHエージェントを立ちあげます::

    $ ssh-agent bash
    $ ssh-add ~/.ssh/id_rsa

(セットアップによっては、代わりにpemファイルを指定するためにAnsibleの
'--private-key' オプションを使ってもよいでしょう)

すべてのノードにpingを投げます::

    $ ansible all -m ping

AnsibleはちょうどSSHと同じように、現在のユーザ名を使ってマシンへのリモート
接続を試みます。リモートのユーザ名を上書きするには、単に ``-u`` の
パラメータを使います。

sudoモードでアクセスしたい場合は、それを行うためのフラグもあります::

    # as bruce
    $ ansible all -m ping -u bruce
    # as bruce, sudoing to root
    $ ansible all -m ping -u bruce --sudo
    # as bruce, sudoing to batman
    $ ansible all -m ping -u bruce --sudo --sudo-user batman

(The sudo implementation is changeable in ansbile’s configuration file
if you happen to want to use a sudo replacement.
Flags passed dot sudo can also be set.)

.. admonition:: todo

   訳せず。


今度は通常のコマンドをすべてのノードで実行します::

    $ ansible all -a "/bin/echo hello"

おめでとうございます。今まさに、Ansibleを使ってノードに連絡しました。今度は
現実世界の `コマンドラインの例と次のステップ`_ のいくつかを読んだり、別の
モジュールで何ができるかを探る番です。Ansibleプレイブック言語も同様です。
Ansibleは単にコマンドを実行するだけではなく、構成管理やデプロイの機能も
備えています。調べることはたくさんありますが、あなたはもう完全に動作する
インフラを持っています！


.. admonition:: See Also

   `コマンドラインの例と次のステップ`_
     基本的なコマンドの例

   `プレイブック`_
     Ansibleの構成管理言語を学ぶ

   `メーリングリスト`_
     質問？ヘルプ？アイデア？Google Groupsへお立ち寄りください

   `irc.freenode.net`_
     #ansible IRC Chatチャンネル

.. ------------------------------------------------------------
.. external link

.. _epel: http://fedoraproject.org/wiki/EPEL
.. _`EPELを使うと`: epel_
.. _`EPELを設定します`: epel_

.. _ppa: https://launchpad.net/~rquillo/+archive/ansible
.. _`このPPA`:

.. _aur: https://aur.archlinux.org/packages.php?ID=58621

.. _rel: http://ansible.cc/releases
.. _`Ansible/downloads`: rel_


.. _`コマンドラインの例と次のステップ`: ./command_line_examples_and_next_steps.rst
.. _`プレイブック`: ./playbook.rst

.. _`メーリングリスト`: http://groups.google.com/group/ansible-project
.. _`irc.freenode.net`: http://irc.freenode.net/
