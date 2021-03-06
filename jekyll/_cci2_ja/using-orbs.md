---
layout: classic-docs
title: "Orb のコンセプト"
short-title: "コンセプト"
description: "Orb のコンセプトの概要に関する入門ガイド"
categories:
  - getting-started
order: 1
---

CircleCI の Orb とは、ジョブ、コマンド、Executor などの構成要素をまとめた共有可能なパッケージです。 CircleCI では承認済み Orbs に加え、CircleCI パートナーによってオーサリングされたサードパーティ製の Orbs も提供しています。 まずは、こうした既存の Orbs がご自身の構成ワークフローに活用できるかどうかを評価することをお勧めします。 承認済み Orb の一覧は、[CircleCI Orb レジストリ](https://circleci.com/orbs/registry/)にて確認してください。

Orb を使用する前に、まず Orb の中核的なコンセプト、Orb の構造と動作についてよく理解しておく必要があります。 こうしたコンセプトについて基本的な知識を身に着けることで、それぞれの環境で Orb を簡単に活用できるようになります。

### 承認済み Orbs と サードパーティ製 Orbs

CircleCI では、プラットフォームでの動作テストを経て承認された Orbs を数多く公開しています。 承認済みの Orb はプラットフォームの一部として扱われ、これ以外の Orb はすべてサードパーティ製 Orbs と見なされます。 メモ: 組織の管理者は、組織の [Settings (設定)] > [Security (セキュリティ)] ページで、サードパーティ製の未承認 Orb の使用をオプトインする必要があります。

すべての Orb はオープンであり、だれでも使用したりソースを確認したりできます。

### 設計手法

Orb を使用する前に、これらの Orb が設計されたときのさまざまな方針や手法について理解しておくとよいでしょう。 Orb は、以下の点を考慮して設計されています。

- Orb では透明性が確保されている - Orb を実行するとき、その Orb のソースは自分だけでなく、他のだれもが表示できます。
- メタデータを使用できる - すべてのキーに説明キーを含めることができ、Orb のトップレベルに説明を追加できます。
- 安定版 Orb は必ずセマンティック バージョニング (semver) される - CircleCI では、「dev:」で始まるバージョン番号を持つ開発版 Orb の作成・使用が許可されています。
- 安定版 Orb は変更不可 - Orb をセマンティック バージョンとしてパブリッシュした後で Orb を変更することはできません。 これにより、コア オーケストレーションでの予期しない破損や動作の変更を防ぐことができます。
- レジストリは 1 つ (インストールあたり) - circleci.com を含む CircleCI のインストールごとに、Orb を保持できるレジストリを 1 つだけ持つことができます。
- Orbs の作成・編集には制限がある - 安定版 Orb をパブリッシュ・プロモートできるのは組織の管理者のみです。 組織のメンバーは、新しく作成された初期の Orb を開発版 Orb としてパブリッシュできます。 また、すべての Orb 名前空間は組織に所有されます。

### Orb の構造

Orb は、以下の要素で構成されます。

- コマンド
- ジョブ
- Executors

#### コマンド

コマンドは、再利用可能なステップの集合体であり、既存のジョブ内から特定のパラメーターを使用して呼び出すことができます。 たとえば、`sayhello` コマンドを呼び出す場合は、以下のように `to` にパラメーターを渡します。

    version: 2.1
    jobs:
      myjob:
        docker:
          - image: "circleci/node:9.6.1"
        steps:
          - myorb/sayhello:
              to: "Lev"
    

#### ジョブ

ジョブは、ステップの集合体と、それが実行される環境の 2 つの部分で構成されます。 ジョブをビルド構成または Orb で定義すると、構成または外部の Orb 構成のジョブ キーの下にあるマップ内でジョブ名を定義できます。

ジョブは config.yml ファイルのワークフロー スタンザで呼び出す必要があります。このとき、必要なパラメーターをサブキーとしてジョブに渡します。

#### Executor

Executor は、ジョブのステップが実行される環境を定義します。 CircleCI 構成でジョブを宣言するとき、ジョブを実行する環境のタイプ (docker、machine、macos、windows など) と共に、その環境について以下のようなパラメーターを定義します。

- 挿入する環境変数
- 使用するシェル
- 使用する resource_class のサイズ

設定ファイル内のジョブの外側で Executor を宣言すると、その宣言をスコープ内のすべてのジョブで使用できるため、1 つの Executor 定義を複数のジョブで再利用できます。

Executor の定義では、以下のキーを使用できます (一部のキーは、ジョブ宣言を使用する際にも使用できます)。

- docker、machine、windows、macos
- environment
- working_directory
- shell
- resource_class

以下に、Executor を使用する簡単な例を示します。

    version: 2.1
    executors:
      my-executor:
        docker:
          - image: circleci/ruby:2.4.0
    
    jobs:
      my-job:
        executor: my-executor
        steps:
    
          - run: echo outside the executor
    

上記の例では、executor キーの単一の値として、`my-executor` という Executor が渡されています。 代わりに、`my-executor` を executor の下で name キーの値として渡すことも可能です。 この方法は、主に、Executor の呼び出しにパラメーターを渡す場合に使用されます。 その場合の例は以下のとおりです。

    version: 2.1
    jobs:
      my-job:
        executor:
          name: my-executor
        steps:
          - run: echo outside the executor
    

### 名前空間

名前空間は、一連の Orbs を編成するために使用されます。 各名前空間はレジストリ内に一意で変更不可の名前を持ちます。また、名前空間内の各 Orb は一意の名前を持ちます。 たとえば、`circleci/rails` Orb と username/rails という名前の Orb は、別々の名前空間にあるため、レジストリ内で共存できます。

**メモ:** 名前空間は組織に所有されます。

デフォルトでは、組織は名前空間を 1 つだけ要求できるように制限されています。 これは、名前空間の占拠や紛らわしさを制限するためのポリシーです。 複数の名前空間が必要な場合は、CircleCI のアカウント チームにお問い合わせください。

### Orbs でのセマンティック バージョニング

Orbs は、3 つの数字による標準的なセマンティック バージョニング システムを使用してパブリッシュされます。

- メジャー
- マイナー
- パッチ

Orb オーサーは、セマンティック バージョニングに従う必要があります。 config.yml 内では、ワイルドカードでバージョン範囲を指定して Orbs を解決することも可能です。 また、volatile という特殊な文字列を使用して、ビルド実行時点の最大のバージョン番号をプルできます。

たとえば、mynamespace/some-orb@8.2.0 が存在すると、8.2.0 の後に mynamespace/some-orb@8.1.24 や mynamespace/some-orb@8.0.56 がパブリッシュされても、volatile は引き続き mynamespace/some-orb@8.2.0 を最大のセマンティック バージョンとして参照します。

以下に Orb バージョン宣言の例を挙げ、その意味を説明します。

- circleci/python@volatile - ビルドがトリガーされた時点でレジストリにある最大の Python Orb バージョンを使用します。 通常、これは最も新しくパブリッシュされた、最も安定性が低い Python Orb です。
- circleci/python@2 - Python Orb バージョン 2.x.y のうち、最新のバージョンを使用します。
- circleci/python@2.4 - Python Orb バージョン 2.4.x のうち、最新のバージョンを使用します。
- circleci/python@3.1.4 - 厳密にバージョン 3.1.4 の Python Orb を使用します。

## Orb のバージョン (開発版と 安定版)

ワークフローで使用できる Orb には、主に開発版 Orb と安定版 Orb の 2 種類があります。 ワークフローのニーズに応じて、これらの Orb のいずれかを選んで使用できます。 以下の各セクションでは、ワークフローに使用するにはどちらの種類が適切か、十分な理解に基づいて判断していただけるよう、これらの 2 種類の Orb の違いについて説明します。

すべての安定版 Orbs は組織オーナーによって安全にパブリッシュできますが、開発版 Orbs はチームのオーナー以外のメンバーでもパブリッシュできます。 安定版 Orb とは異なり、開発版 Orb は変更可能で 90 日後に有効期限が切れるため、アイデアをすばやく繰り返し組み込みたい場合に理想的です。

安定版 Orbs ではワイルドカードによるセマンティック バージョン参照を使用できますが、開発版は mynamespace/myorb@dev:mybranch のように完全修飾名で参照する必要があります。 開発版には便利な省略表記がありません。

Orb の各バージョンは、開発版または安定版としてレジストリに追加されます。 安定版は、1.5.3 のように常にセマンティック バージョンです。一方、開発版には文字列タグを付加でき、`dev:myfirstorb` のように常に「dev:」プレフィックスが付きます。

**メモ: ** 開発版は変更可能で、有効期限があり、90 日後に削除されます。したがって、本番ソフトウェアを開発版 Orb に依存させないこと、また、開発版は Orb 開発を集中的に進めている間にのみ使用することを強くお勧めします。 チームの組織メンバーは、別のメンバーの設定ファイルをコピー & ペーストするのではなく、開発版 Orb を基に Orb のセマンティック バージョンをパブリッシュできます。

### 開発版および安定版 Orb のセキュリティ プロファイル

- 安定版 Orbs をパブリッシュできるのは、組織オーナーのみです。
- 開発版 Orbs は組織の任意のメンバーが名前空間にパブリッシュできます。
- 組織オーナーは、すべての開発版 Orb をセマンティック バージョンの安定版 Orb にプロモートできます。

### 開発版および安定版 Orb の維持特性と可変特性

開発版 Orbs は変更可能で、有効期限があります。 Orb がパブリッシュされた名前空間を所有する組織のメンバーは、だれでも開発版 Orb を上書きできます。

安定版 Orbs は変更不可で、永続的です。 特定のセマンティック バージョンで安定版 Orb をパブリッシュすると、そのバージョンの Orb の内容は変更できません。 安定版 Orb の内容を変更するには、一意のバージョン番号で新しいバージョンをパブリッシュする必要があります。 Orb を安定版としてパブリッシュする際は、circleci CLI で orb publish increment コマンドや orb publish promote コマンドを使用することをお勧めします。

### 開発版および安定版 Orbs のバージョニング セマンティック

開発版 Orbs には、`dev:<< your-string >>` 形式のタグが付きます。 安定版 Orbs は常にセマンティック バージョニング (semver) スキームを使用してパブリッシュされます。

開発版 Orbs に指定できる文字列ラベルには以下の制限があります。

- 空白文字以外の最大 1,023 文字

開発版 Orb タグの例

有効な例

      "dev:mybranch"
      "dev:2018_09_01"
      "dev:1.2.3-rc1"
      "dev:myinitials/mybranch"
      "dev:myVERYIMPORTANTbranch"
    

無効な例

      "dev: 1" (スペースは使用不可)
      "1.2.3-rc1" (先頭に "dev:" が含まれていない)
    

安定版 Orb では `X.Y.Z` 形式を使用します。ここで、`X` は「メジャー」バージョン、`Y` は「マイナー」バージョン、`Z` は「パッチ」バージョンです。 たとえば、2.4.0 は、メジャー バージョン 2、マイナー バージョン 4、パッチ バージョン 0 を意味します。

厳密に強制されているわけではありませんが、安定版 Orbs のバージョニングには、メジャー、マイナー、パッチの標準セマンティック バージョニング規則を使用することをお勧めします。

- メジャー: 互換性がない API の変更を行う場合
- マイナー: 下位互換性を維持しながら機能を追加する場合
- パッチ: 下位互換性を維持しながらバグ修正を行う場合

### Orb 内での Orbs の使用と登録時解決

Orb 内で orbs スタンザを使用することも可能です。

安定版 Orb リリースは変更不可なので、Orb 依存関係の解決は、ビルドの実行時ではなく Orb の登録時にすべて行われます。

たとえば、`biz/baz@volatile` をインポートする orbs スタンザを含んだ Orb `foo/bar` が、バージョン 1.2.3 でパブリッシュされるとします。 `foo/bar@1.2.3` を登録する時点で、`biz/baz@volatile` が最新バージョンとして解決され、その要素がパッケージ バージョンの `foo/bar@1.2.3` に直接インクルードされます。

`biz/baz` が 3.0.0 に更新されても、`foo/bar` が 1.2.3 より上のバージョンでパブリッシュされるまで、`foo/bar@1.2.3` を使用しているユーザーには ``biz/baz@3.0.0 の変更が反映されません。

**メモ:** Orb の要素は、他の Orb の要素を使用して直接構成できます。 たとえば、以下の例のような Orb を使用できます。

    version: 2.1
    orbs:
      some-orb: some-ns/some-orb@volatile
    executors:
      my-executor: some-orb/their-executor
    commands:
      my-command: some-orb/their-command
    jobs:
      my-job: some-orb/their-job
      another-job:
        executor: my-executor
        steps:
          - my-command:
              param1: "hello"
    

### 安定版 Orbs の削除

CircleCI は通常、グローバルに読み取り可能としてパブリッシュされた安定版 Orbs を削除しないように要請しています。構成のソースとしての Orb レジストリの信頼性およびすべての Orb ユーザーからの信頼を損なうおそれがあるためです。

緊急の理由で Orb を削除する必要がある事態が発生した場合は、CircleCI にご連絡ください (メモ セキュリティ上の懸念から削除を行う場合は、CircleCI Security の Web ページを使用して、情報開示の責任を果たす必要があります)。

## 関連項目
{:.no_toc}

- [Orb の概要]({{site.baseurl}}/ja/2.0/orb-intro/):Orb の使用とオーサリングについての概要
- [Orbs リファレンス ガイド]({{site.baseurl}}/ja/2.0/reusing-config/): 再利用可能な Orbs、コマンド、パラメーター、および Executors の詳細
- [CircleCI 構成クックブック]({{site.baseurl}}/ja/2.0/configuration-cookbook/#構成レシピ): CircleCI Orbs のレシピを構成に使用する詳しい方法
