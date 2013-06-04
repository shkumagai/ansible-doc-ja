

Ansible ドキュメント
````````````````````

.. イメージ省略

このページは `Ansible <http://ansible.cc>`_ の使い方のドキュメントです。
クラスの受講にも興味があるかも知れませんね。

.. イメージ省略


playbook、構成管理、デプロイメント、そして組織化に飛び込む前に、Ansibleの
インストール方法やいくつかの基本的な情報を学びましょう。 ``/usr/bin/ansible`` を
使って、ノード間で並列にアドホックなコマンドを実行する方法を練習します。
Ansibleコアで利用可能なモジュールなども分かります (あなた自身の手で書くことも
できますが、それはまた後でお見せします) 。

.. toctree::
   :maxdepth: 1

   gettingstarted
   patterns
   examples
   modules


概要
````

.. image:: http://ansible.cc/img/ansible_arch.png
   :alt: ansible architecture diagram
   :width: 566px
   :height: 439px

playbook
````````

playbookはAnsibleの組織化のための言語です。基本的なレベルでは、リモート
マシンの構成やデプロイを管理することができます。より高度なレベルでは、
ローリングアップデートを含めた逐次型の多層ロールアウトができ、その過程で
監視サーバやロードバランサとやり取りしながら、他のホストにアクションを
委譲できます。あなたは小さく始め、徐々に必要な機能を選択していくことができます。
playbookは可読性を持つようにデザインされていて、基本的なテキスト言語で
開発されています。playbookやそこに含まれるファイルをまとめる方法は複数ある
ので、我々はそれに対していくつかの提案を提供し、Ansibleを最大限に活用しましょう。

.. toctree::
   :maxdepth: 1

   playbooks
   playbooks2
   bestpractices
   YAMLSyntax
   Example Playbooks <https://github.com/ansible/ansible-examples>

開発者向け情報
``````````````

任意の言語で、独自のモジュールをビルドする方法を学びます。AnsibleのPython API を
探索し、あなたの環境で他のソリューションと統合するためのPythonプラグインを
書きましょう。

.. toctree::
   :maxdepth: 1

   api
   moduledev

その他いろいろ
``````````````

`Ansibleのイケてる小技をCoderwallで勉強・シェアしよう <https://coderwall.com/p/t/ansible>`_ -
Github か Twitter でサインインして投票したり、自分のモノを追加しましょう。

`Ansible ユーザやAnsibleについての引用 <http://www.ansibleworks.com/users>`_ 。

その他のリンク:

.. toctree::
   :maxdepth: 1

   contrib
   glossary
