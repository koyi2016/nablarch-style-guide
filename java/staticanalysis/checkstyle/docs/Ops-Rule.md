# Checkstyle運用ガイド

実際のプロジェクトでCheckstyleを運用していく際のガイドです。

ここでは、CheckStyle「違反」という言葉を「設定ファイル記載のCheckyStyleルールに違反している」という意味で使用します。※その際、ルールの重要度(error, warning等)は問いません。

## 基本的な考え方

### 原則1: Checkstyle違反は全て解決します

プロジェクトで設定したCheckstyleルールの違反は全て解決するようにします。
違反を暗黙的に許容するような運用にすると、恒常的にルール違反が表示され続け、
Checkstyle違反を修正するモチベーションを失います。

常に違反0件である状態をキープすることを目標とします。

### 原則2: Checkstyle違反を解消するために品質を下げてはいけません

開発者はCheckstyle違反を見つけたら対応をしなければなりませんが、
その際違反を回避するための修正をしたことによって、かえってソースコード品質を落とすようなことがあってはいけません。

このような場合は、有識者やプロジェクトのアーキテクトに相談して
適切な対処を仰ぐようにしましょう。

個人の判断で、Checkstyle違反を無理矢理回避しようとしてはいけません。
Checkstyleはコード品質を上げるため（下げないため）のものであり、
これを回避するためにコード品質を下げるのは本末転倒です。


### 原則3: 正当な理由があってCheckstyle違反となる場合、チェック対象から外します

正当な理由がある場合は、Checkstyle違反となっているソースコードをチェック対象から除外するようにします。

プロジェクトのアーキテクトは開発者からの申請を受けて妥当であると判断した場合、
該当箇所をチェック対象から除外するようにしてください。

この運用をすることによって、違反が放置されたり、開発者が無理に違反を回避したりすることを防げます。

## 例外登録ルール

### Checkstyle違反除外方法

Checkstyle違反となっている箇所をチェック対象から外すにはいくつかやり方があります。
(詳細は[Checkstyle -Filters](http://checkstyle.sourceforge.net/config_filters.html)を参照)

ここでは以下の2つを解説します。

- 除外設定ファイルによる除外設定([SuppressionFilter](http://checkstyle.sourceforge.net/config_filters.html#SuppressionFilter))
- 行コメントによる除外設定([SuppressWithNearbyCommentFilter](http://checkstyle.sourceforge.net/config_filters.html#SuppressWithNearbyCommentFilter))


### 除外設定ファイルによる除外設定

除外設定ファイルを用いた除外設定では、除外対象となるソースコードを列挙します。
除外対象となるソースコードがひとつの設定ファイルに集約されているため、
アーキテクトが一元管理しやすいというメリットがあります。

開発者から除外申請をRedmine等の課題管理システムで受け付けて管理するようにします。
アーキテクトは除外申請が妥当であり除外することを決定した場合、設定ファイルに除外設定を追加します。

設定ファイルには次の項目を記載します。

- 対象ファイル（正規表現）
- 対象チェック項目（正規表現）

除外設定ファイルの記述例を以下に示します。

``` xml
<suppressions>

  <!-- 自動生成コードはチェック対象外 -->
  <suppress checks=".*" files=".*Entity.java$" />

  <!-- #12345 により除外設定 -->
  <suppress checks=".*" files="Foo.java" />
  
</suppressions>
```

除外設定をする場合は次のようにしておくと、後で経緯を追跡できます。

- 課題管理システムの課題管理番号をXMLコメントに記載する
- バージョン管理システムのコミットコメントに記載する

#### 設定ファイルの記載方法

Checkstyleの設定ファイルに除外設定ファイルのパスを記載します。
2つのファイルを同じディレクトリに配置する例を以下に示します。

``` xml
  <module name="SuppressionFilter">
    <property name="file" value="${config_loc}/suppressions.xml"/>
    <property name="optional" value="false"/>
  </module>
```

`${config_loc}`はEclipse Checkstyle PluginおよびIntelliJ Checkstyle-IDEAプラグイン使用時に「Checkstyle設定ファイルが置かれているディレクトリ」と解釈されます。

Mavenから起動する場合は、[Checkstyleの実行をMavenで行う方法のガイド](./Maven-settings.md)を参照してください。

#### 注意事項

この方法ではファイル単位での除外となるので、メソッド単位のような細かい除外はできません。
このため、除外対象をできるだけ増やさないようにする必要があります。
やむを得ないもののみを除外対象として認める、現実的に守ることが難しいルールは廃止する等の工夫が必要です。

なお、行番号指定による除外も可能ですが、行番号は修正によってズレてしまうので推奨しません。


### 行コメントによる除外設定

ソースコード上のCheckstyle違反箇所に、特定のコメントを付与することで、
該当行をCheckstyle対象外にできます。


以下の例では変数名にアンダースコアを使用しており通常はCheckstyle違反となりますが、
行コメントにより除外設定しているためCheckstyle対象外となります。

``` java
   private int counter_;  // SUPPRESS CHECKSTYLE #12345
```

開発者から除外申請をRedmine等の課題管理システムで受け付けて管理するようにします。
アーキテクトは除外申請が妥当除外することを決定した場合、
開発者は行コメントに課題管理システムの課題管理番号を記載するようにします。

Checkstyle設定ファイルの記載例を以下に示します。
以下の例では、「// SUPPRESS CHECKSTYLE #12345により除外設定」というようなコメントを記載することで
該当行がチェック対象外となります。


``` xml
    <!-- コメントによるCheckstyle除外設定 -->
    <module name="SuppressWithNearbyCommentFilter">
      <!--
        特定の書式のコメントを記載することで該当行をCheckstyle対象外にできる。
        コメント例： // SUPPRESS CHECKSTYLE #12345により除外設定
       -->
      <property name="commentFormat" value="SUPPRESS CHECKSTYLE #\d+"/>
    </module>
```

この方法は、ソースコードのどの箇所が除外対象となっているのかがわかりやすいというメリットがあります。
また、違反箇所をピンポイントで除外できるため、除外対象箇所を最小限にできます。

一方、ソースコードに除外設定を直接記載するため、管理がしにくいというデメリットがあります。
例えば、ソースコードをコピーした場合に除外設定もコピーされるため、
意図しないコードが除外設定となってしまう可能性があります。

除外設定一箇所につき必ず1つ対応する課題チケットを発行して課題管理番号が重複しないようにする
といった工夫が必要となります（grepして課題管理番号が重複していたら誤りがあると見なす）。


## コードフォーマッターとの組み合わせ

Checkstyle違反のうち、コードフォーマッターを実行することで自動的に解消できるものがあります（インデントや空白等）。
コードフォーマッターの設定とCheckstyleの設定の両者で整合性が取れるように設定をしておきます。
バージョン管理システムへのコミット時にフォーマッターを自動で実行するよう設定しておくことで
自動的に違反を減らすことができます。