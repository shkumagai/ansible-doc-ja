YAML 構文
=========

このページではansible playbook(我々の構成管理言語)を表現する、
正しいYAML構文の基本的な概要を示します。

私たちがYAMLを使うのは、XMLやJSONといった他の一般的なデータ形式にくらべて、
人間にとってより読み書きが容易だからです。
さらにほとんどのプログラミング言語で、YAMLを扱うライブラリが使えます。

これが実際にどのように使われるのかを確認するには、合わせて
:doc:`playbooks` を読むとよいでしょう。


YAMLの基礎
----------

`ansible` の場合、ほとんどのYAMLファイルはリストで始まります。
リスト内の各アイテムは key/value のペアのリストで、一般的に"ハッシュ"
または"辞書"と呼ばれます。
そのため、YAML内でのリストや辞書の書き方を知る必要があります。

YAMLにはもう一つ、ちょっと奇妙なクセがあります。
すべてのYAMLファイルは (`ansible` に関連するとしないとに関係なく)
``---`` で始まる必要があります。
これは単にYAMLの書式で、文書の始まりを意味します。

リストのすべてのメンバーは、同じインデントレベルから始まる行であり、
``-`` (ダッシュ) 文字で始まります::

    ---
    # A list of tasty fruits
    - Apple
    - Orange
    - Strawberry
    - Mango

辞書は、単純に ``key:`` と ``value`` の形式で表現します::

    ---
    # An employee record
    name: Example Developer
    job: Developer
    skill: Elite

どうしても必要なら、辞書は省略形式でも表現できます::

    ---
    # An employee record
    {name: Example Developer, job: Developer, skill: Elite}

.. _truthiness:

ansibleはまずこれらを使用しませんが、いくつかの形でブール(真/偽)値も指定
できます::

    ---
    create_key: yes
    needs_agent: no
    knows_oop: True
    likes_emacs: TRUE
    uses_cvs: false

これまでのYAMLの例で学んだことを組み合わせてみましょう。
これは本当にAnsibleとは何の関係もありませんが、YAMLフォーマットの感覚が
つかめるでしょう::

    ---
    # An employee record
    name: Example Developer
    job: Developer
    skill: Elite
    employed: True
    foods:
        - Apple
        - Orange
        - Strawberry
        - Mango
    languages:
        ruby: Elite
        python: Elite
        dotnet: Lame

`Ansible` playbook を書き始めるに当たって知る必要があることは、これだけです。

落とし穴
--------

YAMLは通常、非常に友好的ですが、以下はYAML構文エラーになります:

    foo: somebody said I should put a colon here: so I did

コロンを使ったハッシュの値を引用符で、このように括りたくなるでしょう:

    foo: "somebody said I should put a colon here: so I did"

すると、コロンは保持されます。

さらに、Ansibleは "{{ var }}" を変数として使います。
コロンの後が"{"で始まる場合、Ansibleはそれを辞書と解釈しようとするので、
このように括らなければいけません::

    foo: "{{ variable }}"


.. seealso::

   :doc:`playbooks`
       Learn what playbooks can do and how to write/run them.
   `YAMLLint <http://yamllint.com/>`_
       YAML Lint (online) helps you debug YAML syntax if you are having problems
   `Github examples directory <https://github.com/ansible/ansible/tree/devel/examples/playbooks>`_
       Complete playbook files from the github project source
   `Mailing List <http://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
