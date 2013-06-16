Ansibleのリソース
=================

.. イメージ省略

ユーザがplaybook、module、および記事を寄稿しました。これは小さなキュレーション
のリストですが、成長しています。誰でもこのドキュメントに追加ことを奨励しています。
githubでdocsite/rst/contrib.rstにプル・リクエストを送るだけです！


Ansible Module
``````````````

Ansible moduleはAnsibleに新しいクライアントサイドのロジックを追加する手段です。
これらは任意の言語で記述できます。概ね我々のゴールは、いくつかユースケースや
実装に依存するものはコアの外に残すかも知れないですが、ほとんどのモジュールを
コアに含める（"バッテリー同梱！"）ことです。

- `公式"コア"Ansible モジュール <http://ansible.cc/docs/modules.html>`_ - various
- `Linode <https://github.com/lextoumbourou/ansible-linode>`_ - Lex Toumbourou
- `zypper (bashモジュールのサンプル) <https://github.com/jpmens/ansible-zypp>`_ -
  jp\_mens
- `プロビジョニング関連追加モジュール <https://github.com/ansible-provisioning>`_ -
  jhoekx and dagwieers
- `動的DNS更新 <https://github.com/jpmens/ansible-m-dnsupdate>`_ - jp\_mens

すべてのPythonモジュールはボイラープレートコードの量を劇的に削減するために、
共通の"AnsibleModule"クラスの利用が必須です。
上記のすべてのモジュールがこの機能を利用できるわけではありません。詳細については
公式のドキュメントを参照してください。


選り抜きのplaybook
``````````````````

`プレイブック <http://ansible.cc/docs/playbook.html>`_ はAnsibleの構成管理言語
です。たいていのアプリケーションのためにスクラッチから自分で書くのが簡単ですが、
他の人がやっていることを参考にするにも便利です。

- `Hadoop <https://github.com/jkleint/ansible-hadoop>`_ - jkleint
- `LAMP <https://github.com/fourkitchens/server-playbooks>`_ -
  `Four Kitchens <http://fourkitchens.com>`_
- `LEMP <https://github.com/francisbesset/ansible-playbooks>`_ - francisbesset
- `Ganglia (demo) <https://github.com/mpdehaan/ansible-examples>`_ - mpdehaan
- `Nginx <http://www.capsunlock.net/2012/04/ansible-nginx-playbook.html>`_ - cocoy
- `OpenStack <http://github.com/lorin/openstack-ansible>`_ - lorin
- `Systems Configuration <https://github.com/cegeddin/ansible-contrib>`_ - cegeddin
- `Fedora Infrastructure <http://infrastructure.fedoraproject.org/cgit/ansible.git/tree/>`_ -
  `Fedora <http://fedoraproject.org>`_


コールバックとプラグイン
````````````````````````

Ansibleプロジェクトはリポジトリ全体で、新しいコネクションタイプ、logging/event
コールバック、およびインベントリデータのストレージを使ってAnsibleの拡張に専念して
います。CobblerやEC2とやり取りをしたり、ログ出力の仕方を調整したり、さらには
効果音を追加しようともしています。

-  `Ansible-Plugins <https://github.com/ansible/ansible/tree/devel/plugins>`_


スクリプトとその他
``````````````````

Ansibleは単なるプログラムではなく、APIでもあります。
これは、"Runner"とPlaybook APIの上手な組み合わせや、その他ソフトウェアの
興味深いものとの組み合わせの例です。

-  `Ansible Vagrant plugin <https://github.com/dsander/vagrant-ansible>`_ -
   dsander
-  `Ansible+Vagrant Tutorial <https://github.com/mattupstate/vagrant-ansible-tutorial>`_ -
   mattupstate -
-  `virt-install <http://fedorapeople.org/cgit/skvidal/public_git/scripts.git/tree/ansible/start-prov-boot.py>`_ -
   skvidal
-  `rebooting hosts <http://fedorapeople.org/cgit/skvidal/public_git/scripts.git/tree/ansible/host-reboot>`_ -
   skvidal
-  `uptime (API demo) <https://github.com/ansible/ansible/blob/devel/examples/scripts/uptime.py>`_ -
   mpdehaan
-  `vim snippet generator <https://github.com/bleader/ansible_snippet_generator>`_ -
   bleader


ブログ、ビデオ＆紹介記事
````````````````````````

-  `HighScalability.com <http://highscalability.com/blog/2012/4/18/ansible-a-simple-model-driven-configuration-management-and-c.html>`_ - mpdehaan
-  `ColoAndCloud.com interview <http://www.coloandcloud.com/editorial/an-interview-with-ansible-author-michael-dehaan/>`_ - mpdehaan
-  `dzone <http://server.dzone.com/articles/ansible-cm-deployment-and-ad>`_ - Mitch Pronschinske
-  `Configuration Management With Ansible <http://jpmens.net/2012/06/06/configuration-management-with-ansible/>`_ - jp\_mens
-  `Shell Scripts As Ansible Modules <http://jpmens.net/2012/07/05/shell-scripts-as-ansible-modules/>`_ - jp\_mens
-  `Ansible Facts <http://jpmens.net/2012/07/15/ansible-it-s-a-fact/>`_ - jp\_mens
-  `Infrastructure as Data <http://www.capsunlock.net/2012/04/ansible-infrastructure-as-data-not-infrastructure-as-code.html>`_ - cocoy
-  `Ansible Pull Mode <http://www.capsunlock.net/2012/05/using-ansible-pull-and-user-data-to-setup-ec2-or-openstack-servers.html>`_ - cocoy
-  `Exploring Configuration Management With Ansible <http://palominodb.com/blog/2012/08/01/exploring-configuration-management-ansible>`_ - Palamino DB
-  `You Should Consider Using SSH Based Configuration Management <http://www.lshift.net/blog/2012/07/30/you-should-consider-using-ssh-based-configuration-management>`_ - LShift Ltd
-  `Deploying Flask/uWSGI, Nginx, and Supervisorctl <http://mattupstate.github.com/python/devops/2012/08/07/flask-wsgi-application-deployment-with-ubuntu-ansible-nginx-supervisor-and-uwsgi.html>`_ - mattupstate
-  `Infracoders Presentation <http://www.danielhall.me/2012/10/ansible-talk-infra-coders/>`_ - Daniel Hall
-  `Ansible - an introduction <https://speakerdeck.com/jpmens/ansible-an-introduction>`_ - jp\_mens
-  `Using Ansible to setup complex networking - <http://exarv.nl/2013/02/using-ansible-to-setup-complex-networking/>`_ - Robert Verspuy
-  `Video presentation to Montreal Linux <http://www.youtube.com/embed/up3ofvQNm8c>`_ - Alexandre Bourget
-  `Provisioning CentOS EC2 Instances with Ansible <http://jpmens.net/2012/11/21/provisioning-centos-ec2-instances-with-ansible/>`_ - jp\_mens


免責事項
````````

ここで紹介されているモジュールやプレイブックは最新のAnsibleの機能を使っていない
かも知れません。もしAnsibleの特定のバージョンの機能に疑問がある場合には、いつでも
`ansible.cc <http://ansible.cc>`_ に相談したり、
特に `Best Practices <http://ansible.cc/docs/bestpractices.html>`_ を参照すれば
いくつかのヒントやコツが役に立つかも知れません。

Ansible の著作権は (C) 2012, `Michael DeHaan <http://twitter.com/laserllama>`_
およびその他により、GPLv3ライセンスの下で利用可能です。ここにある各コンテンツの
ライセンスについては、個々の貢献者によって指定されます。
