======================
 Ansible ドキュメント
======================

.. source: http://ansible.cc/docs/released/1.1/

プレイブック、構成管理、デプロイメント、そして組織化に飛び込む前に、Ansibleの
インストール方法やいくつかの基本的な情報を学びましょう。 ``/usr/bin/ansible`` を
使って、ノード間で並列にアドホックなコマンドを実行する方法を練習します。
Ansibleコアで利用可能なモジュールなども分かります (あなた自身の手で書くことも
できますが、それはまた後でお見せします) 。

- Getting Started
- Inventory & Patterns
- Command Line Examples And Next Steps
- Ansible Modules


概要
====

.. image:: http://ansible.cc/img/ansible_arch.png


プレイブック
============

プレイブックはAnsibleの組織化のための言語です。基本的なレベルでは、リモート
マシンの構成やデプロイを管理することができます。より高度なレベルでは、
ローリングアップデートを含めた逐次型の多層ロールアウトができ、その過程で
監視サーバやロードバランサとやり取りしながら、他のホストにアクションを
委譲できます。あなたは小さく始め、徐々に必要な機能を選択していくことができます。
プレイブックは可読性を持つようにデザインされていて、基本的なテキスト言語で
開発されています。プレイブックやそこに含まれるファイルをまとめる方法は複数ある
ので、我々はそれに対していくつかの提案を提供し、Ansibleを最大限に活用しましょう。

- Playbooks
- Advanced Playbooks
- Best Practices
- YAML Syntax
- Example Playbooks


開発者向け情報
==============

任意の言語で、独自のモジュールをビルドする方法を学びます。AnsibleのPython API を
探索し、あなたの環境で他のソリューションと統合するためのPythonプラグインを
書きましょう。

- API & Integrations
- Module Development


その他いろいろ
==============

`Ansibleのイケてる小技を勉強・シェアしよう on Coderwall`_ -- Github か Twitter で
サインインして投票したり、自分のモノを追加しましょう。

その他のリンク:

- Ansible Resources
- Glossary
