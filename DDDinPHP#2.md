# 2 Architectural Styles

In order to be able to build complex applications, one of the key requirements is having an architectural design that fits the applications needs.
A good advantage of Domain-Driven Design is that it is not tied to any particular architecture style.
Instead, we are free to choose the architecture that best fits the needs of every Bounded Context inside the Core Domain, offering a diverse set of architectural choices for every specific domain problem.

複雑なアプリケーションを構築できるようにするための重要な要件の1つは、アプリケーションのニーズに合ったアーキテクチャ設計を用意することです。
ドメイン駆動設計の優れた利点は、特定のアーキテクチャスタイルに縛られていないことです。
代わりに、コアドメイン内のすべての境界コンテキストのニーズに最適なアーキテクチャを自由に選択して、特定のドメインの問題ごとにさまざまなアーキテクチャの選択肢を提供します。


For example, an Order Processing System can use Event Sourcing to track all the different order operations, a Product Catalog can use CQRS to expose the product details to the different clients and a Content Management System can use plain Hexagonal Architecture to expose requirements such as blogs, static pages, and so on.

たとえば、注文処理システムはイベントソーシングを使用してさまざまな注文操作をすべて追跡でき、製品カタログはCQRSを使用して製品の詳細をさまざまなクライアントに公開でき、コンテンツ管理システムはプレーンなヘキサゴナルアーキテクチャを使用してブログなどの要件を公開できます。 、静的ページなど。


This chapter presents an introduction to every relevant architecture style in the land of PHP, through an evolution from traditional ‘old-school' PHP code to a more sophisticated architecture.
Please note that although there are many other existing architecture styles, like Data Fabric or SOA, we found them a bit complex to introduce from the PHP perspective.

この章では、従来の「昔ながらの」PHPコードからより洗練されたアーキテクチャへの進化を通じて、PHPの世界に関連するすべてのアーキテクチャスタイルを紹介します。
Data FabricやSOAなど、他にも多くの既存のアーキテクチャスタイルがありますが、PHPの観点から導入するのは少し複雑であることに注意してください。



## 2.1 The Good Old Times

Before the release of PHP version 5, the language did not embrace the Object-Oriented paradigm.
Back in these days, the usual way to write applications was by using procedures and global state.
Concepts like Separation of Concerns, MVC and such were very alien among the PHP community.
The example below, is an application written in this ‘traditional way', where applications were composed of many front controllers mixed with HTML code.
During this time Infrastructure, Presentation or UI and Domain layer code was tangled all together.

PHPバージョン5がリリースされる前は、この言語はオブジェクト指向パラダイムを採用していませんでした。
昔は、アプリケーションを作成する通常の方法は、プロシージャとグローバル状態を使用することでした。
関心の分離、MVCなどの概念は、PHPコミュニティの間では非常に異質でした。
以下の例は、この「従来の方法」で記述されたアプリケーションであり、アプリケーションはHTMLコードと混合された多くのフロントコントローラーで構成されていました。
この間、インフラストラクチャ、プレゼンテーション、またはUIとドメインレイヤーのコードがすべて絡み合っていました。


【古いコード①】
```php
include __DIR__ . '/bootstrap.php';

 // DBへの接続オブジェクトを作成する
$db = new PDO('mysql:host=localhost;dbname=my_database', 'a_username', '4_p4ssw0\
rd', [
      PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',
]);

 // エラーメッセージを初期化
$errormsg = null;

if (isset($_POST['submit'] && isValid($_POST['post'])) {
  $post = getFrom($_POST['post']);
  // DB操作のトランザクションを開始
  $db->beginTransaction();

  try {
    $stm = $db->prepare('INSERT INTO posts (title, content) VALUES (?, ?)');
    $stm->exec([
      $post['title'],
      $post['content']
    ]);
  
    // DB操作のトランザクションをコミット
    $db->commit();
  } catch (Exception $e) {
    // DB操作のトランザクションをロールバック
    $db->rollback();
    // エラーメッセージ
    $errormsg = 'Post could not be created! :(';
  }
}

 // SQL文を作成
$stm = $db->prepare('SELECT id, title, content FROM posts');
 // SQL文を実行しデータを取得する
$posts = $stm->fetchAll(PDO::FETCH_ASSOC);

?>
<html>
  <head></head>
  <body>
    <?php if (null !== $errormsg): ?>
    <div class="alert error"><?php echo $errormsg; ?></div>
    <?php else: ?>
    <div class="alert success">Bravo! Post was created successfully!</div>
    <?php endif; ?>
    <table>
      <thead><tr><th>ID</th><th>TITLE</th><th>ACTIONS</th></tr></thead>
      <tbody>
      <!-- foreachで回す -->
      <?php foreach ($posts as $post): ?>
        <tr>
          <td><?php echo $post['ID']; ?></td>
          <td><?php echo $post['TITLE']; ?></td>
          <td><?php editPostUrl($post['ID']); ?></td>
        </tr>
      <?php endforeach; ?>
      </tbody>
    </table>
  </body>
</html>
```

This style of coding is often referred to as the Big Ball of Mud¹.
An improvement seen in this style however, was to encapsulate the header and the footer of the web page in their own separate files, which were included in the others.
This avoided duplication and favoured reuse.

このスタイルのコーディングは、BigBallofMud¹と呼ばれることがよくあります。
ただし、このスタイルで見られた改善点は、Webページのヘッダーとフッターを独自の個別のファイルにカプセル化することでした。これらのファイルは他のファイルに含まれていました。
これにより、重複が回避され、再利用が促進されました。


¹Extracted from the c2.com wiki: 
A BIG BALL OF MUD is haphazardly structured, sprawling, sloppy, DuctTape and bailing wire, SpaghettiCode jungle.

大きな泥だんごは、無計画に構造化され、広大で、ずさんな、ダクトテープとベイルワイヤー、スパゲッティコードジャングルです。


【古いコード②】
```php
include __DIR__ . '/bootstrap.php';

$db = new PDO('mysql:host=localhost;dbname=my_database', 'a_username', '4_p4ssw0\
rd', [
PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',
]);
$errormsg = null;
if (isset($_POST['submit'] && isValid($_POST['post'])) {
$post = getFrom($_POST['post']);
$db->beginTransaction();
try {
$stm = $db->prepare('INSERT INTO posts (title, content) VALUES (?, ?)');
$stm->exec([
$post['title'],
$post['content']
]);
$db->commit();
} catch (Exception $e) {
$db->rollback();
$errormsg = 'Post could not be created! :(';
}
}
$stm = $db->prepare('SELECT id, title, content FROM posts');
$posts = $stm->fetchAll(PDO::FETCH_ASSOC);
?>
<?php include __DIR__ . '/header.php'; ?>
<?php if (null !== $errormsg): ?>
<div class="alert error"><?php echo $errormsg; ?></div>
<?php else: ?>
<div class="alert success">Bravo! Post was created successfully!</div>
<?php endif; ?>
<table>
<thead><tr><th>ID</th><th>TITLE</th><th>ACTIONS</th></tr></thead>
<tbody>
<?php foreach ($posts as $post): ?>
<tr>
<td><?php echo $post['ID']; ?></td>
<td><?php echo $post['TITLE']; ?></td>
<td><?php editPostUrl($post['ID']); ?></td>
</tr>
<?php endforeach; ?>
</tbody>
</table>
<?php include __DIR__ . '/footer.php'; ?>
```

Nowadays, and although it is highly discouraged, there are still applications that use this procedural way of coding.
The main disadvantage of this style of architecture is that there is no real separation of concerns - maintenance and cost of change increases drastically in relation to other well-known and proven architectures.

今日では、非常に推奨されていませんが、この手続き型のコーディング方法を使用するアプリケーションがまだあります。
このスタイルのアーキテクチャの主な欠点は、関心の分離が実際にないことです。他の有名で実績のあるアーキテクチャと比較して、メンテナンスと変更のコストが大幅に増加します。



## 2.2 Layered Architecture
From the code maintainability and reuse perspectives, the best way to make this code a bit easier to maintain would be splitting up concepts - creating layers for each different concern.
In our previous example, it is easy to shape some different layers like the one encapsulating the data access and manipulation, another one to handle infrastructure concerns and a final one for encapsulating the orchestration of the previous two.
An essential rule of the Layered architecture is that each layer may be tightly coupled with the layers beneath it, as shown in the following picture

コードの保守性と再利用の観点から、このコードの保守を少し簡単にする最善の方法は、概念を分割することです。つまり、さまざまな懸念事項ごとにレイヤーを作成します。
前の例では、データのアクセスと操作をカプセル化するレイヤー、インフラストラクチャの問題を処理するレイヤー、前の2つのオーケストレーションをカプセル化するレイヤーなど、いくつかの異なるレイヤーを簡単に形成できます。
階層化アーキテクチャの基本的なルールは、次の図に示すように、各層がその下の層と緊密に結合される可能性があることです。


==(Layered architectureの図)==


What the layered architecture really seeks is the separation of the different components of an application.
For instance, in terms of the previous example, a blog post representation must be completely independent of a blog post as a conceptual entity.
A blog post as a conceptual entity can instead be associated with one or more representations, as opposed to being tightly coupled to a specific representation.
This is commonly named Separation of Concerns.
Another architecture paradigm and pattern that seeks the same purpose is the Model-View-Controller pattern.
It was initially thought and widely-used for building desktop GUI applications, and now it is mainly used in web applications thanks to popular web frameworks like Symfony, Zend Framework or Codeigniter.

階層化アーキテクチャが本当に求めているのは、アプリケーションのさまざまなコンポーネントの分離です。
たとえば、前の例では、ブログ投稿の表現は、概念エンティティとしてのブログ投稿から完全に独立している必要があります。
概念エンティティとしてのブログ投稿は、特定の表現に緊密に結合されるのではなく、代わりに1つ以上の表現に関連付けることができます。
これは一般に関心の分離と呼ばれます。
同じ目的を追求する別のアーキテクチャパラダイムとパターンは、Model-View-Controllerパターンです。
当初はデスクトップGUIアプリケーションの構築に考えられ、広く使用されていましたが、Symfony、Zend Framework、Codeigniterなどの人気のあるWebフレームワークのおかげで、現在は主にWebアプリケーションで使用されています。



### 2.2.1 Model-View-Controller

Model-View-Controller is an architectural pattern and paradigm that divides the application into three main layers:

Model-View-Controllerは、アプリケーションを3つの主要なレイヤーに分割するアーキテクチャパターンとパラダイムです。


• The Model: Captures and centralizes all the domain model behaviour.
This layer manages all the data, logic and business rules independently of the data representation.
It can be said that the Model layer is the heart and soul of every MVC application.

• The Controller: Orchestrates interactions between the other layers.
Triggers actions on the model in order to update its state and refreshes the representations associated to the model.
Additionally, the Controller can also send messages to the View layer in order to change the specific Model representation.

• The View: A layer whose main purpose is to expose the differing representations of the Model layer and to give a way to trigger changes on the Model's state.


•モデル：すべてのドメインモデルの動作をキャプチャして一元化します。
このレイヤーは、データ表現とは関係なく、すべてのデータ、ロジック、およびビジネスルールを管理します。
モデルレイヤーは、すべてのMVCアプリケーションの核心であると言えます。

•コントローラー：他のレイヤー間の相互作用を調整します。
モデルの状態を更新するためにモデルのアクションをトリガーし、モデルに関連付けられた表現を更新します。
さらに、コントローラーは、特定のモデル表現を変更するために、ビューレイヤーにメッセージを送信することもできます。

•ビュー：モデルレイヤーのさまざまな表現を公開し、モデルの状態の変更をトリガーする方法を提供することを主な目的とするレイヤー。


### ==(The MVC patternの図)==


### 2.2.2 Example of Layered Architecture


2.2.2.1 The Model

Following the previous example, we mentioned that different concerns should be split up.
In order to do so, all layers should be identified in our original tangled code.
Through this process we need to pay special attention to the code conforming to the Model layer, which will be the beating heart of the application.

前の例に続いて、さまざまな懸念事項を分割する必要があることを説明しました。
そのためには、すべてのレイヤーを元のもつれたコードで識別する必要があります。
このプロセスを通じて、アプリケーションの心臓部となるモデル層に準拠するコードに特別な注意を払う必要があります。


/// ブログ投稿モデル
class Post
{
  use ProtectsInvariants;
  private $title;
  private $content;
  
  public static function writeNewFrom($title, $content)
  {
    // PHPはこんな書き方をするのかな？
    return new static($title, $content);
  }
  
  private function __construct($title, $content)
  {
    $this->setTitle($title);
    $this->setContent($content);
  }

  // メンバ変数 titleに値をセットする
  private function setTitle($title)
  {
    if (empty($title)) {
      throw new \RuntimeException('Title cannot be empty');
    }

    $this->title = $title;
  }
  
  // メンバ変数 contentに値をセットする
  private function setContent($content)
  {
    if (empty($content)) {
      throw new \RuntimeException('Content cannot be empty');
    }
    $this->content = $content;
  }
}


class PostRepository
{
  // DB接続オブジェクトを格納するメンバ変数
  private $db;
  
  public function __construct()
  {
    // DB接続オブジェクトを作成
    $this->db = new PDO(
      'mysql:host=localhost;dbname=my_database',
      'a_username',
      '4_p4ssw0rd',
      [
      PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',
      ]
    );
  }

  // DBに追加
  public function add(Post $post)
  {
    // トランザクション開始
    $this->db->beginTransaction();
    try {
      $stm = $this->db->prepare(
      'INSERT INTO posts (title, content) VALUES (?, ?)'
      );
      $stm->execute([
        $post->title(),
        $post->content(),
      ]);
      $this->db->commit();
    } catch (Exception $e) {
      // DB登録に失敗した場合はロールバック
      $this->db->rollback();
      throw new UnableToCreatePostException($e);
    }
  }
}



The Model layer is now defined by a Post class and a PostRepository class.
The Post class represents a blog post and the PostRepository class represents the whole collection of blog posts available.

Modelレイヤーは、PostクラスとPostRepositoryクラスによって定義されるようになりました。
Postクラスはブログ投稿を表し、PostRepositoryクラスは利用可能なブログ投稿のコレクション全体を表します。

 **松本疑問** 
 PostRepositoryクラスは利用可能なブログ投稿のコレクション全体を表しているか？


Additionally, another layer inside the Model is needed, a layer that coordinates and orchestrates the domain model behaviour: the Application Layer.

さらに、モデル内の別のレイヤー、つまりドメインモデルの動作を調整および調整するレイヤーであるアプリケーションレイヤーが必要です。


class PostService
{
  public function createPost($title, $content)
  {
    $post = Post::writeNewFrom($title, $content);
    (new PostRepository())->add($post);
    return $post;
  }
}


The PostService is what is known as an Application Service and its purpose is to orchestrate and organize the domain behaviour.
In other words, the Application services are the ones that make things happen and they are the direct clients of a Domain Model.
No other type of object should be able to directly talk to the internal layers of the Model layer.

PostServiceは、アプリケーションサービスと呼ばれるものであり、その目的は、ドメインの動作を調整および整理することです。
言い換えれば、アプリケーションサービスは物事を実現するサービスであり、ドメインモデルの直接のクライアントです。
他のタイプのオブジェクトは、モデルレイヤーの内部レイヤーと直接通信できないようにする必要があります。

 **松本解説** 
   『他のタイプのオブジェクト』は、
   モデルレイヤーの内部レイヤーと直接通信できないようにする必要があります。
   →→ 
   モデルレイヤーの内部レイヤー：
      ここで言う Postクラス、PostRepositoryクラス
   この内部レイヤーを操作できるのは、PostServiceのみで、
   他のタイプのオブジェクトは Postクラス、PostRepositoryクラスを
   直接操作してはならない。



2.2.2.2 The View

The View is a layer that can both receive and send messages from the Model layer and/or from the Controller layer.
Its main purpose is to represent the Model to the user at the UI level, and refresh the representation in the UI each time the Model is updated.

ビューは、モデルレイヤーおよび/またはコントローラーレイヤーからメッセージを送受信できるレイヤーです。
その主な目的は、UIレベルでユーザーにモデルを表現し、モデルが更新されるたびにUIの表現を更新することです。


Generally speaking, the View layer receives an object, often a Data Transfer Object (DTO) instead of instances of the Model layer, gathering all the needed information to be successfully represented.

一般的に、ビューレイヤーはオブジェクト（多くの場合、モデルレイヤーのインスタンスではなくデータ転送オブジェクト（DTO））を受け取り、正常に表現するために必要なすべての情報を収集します。


For PHP there are several template engines that can help a great deal in separating the Model representation from the Model itself and from the Controller.
The most popular one by far is called Twig².
Lets see how the View layer would look like with Twig

PHPの場合、モデル表現をモデル自体およびコントローラーから分離するのに大いに役立つテンプレートエンジンがいくつかあります。
最も人気のあるものはTwig²と呼ばれています。
Twigでビューレイヤーがどのように見えるかを見てみましょう


DTOs instead of Model instances?
モデルインスタンスの代わりにDTO？

This is an old and active topic.
Why create a DTO instead of giving an instance of the Model to the View layer? The main reason and the short answer is, again, Separation of Concerns.
Letting the view inspect and use a Model instance leads to tight coupling between the View layer and the Model layer.
In fact, a change in the Model layer can potentially break all the views that make use of the changed Model instances.

これは古くて活発なトピックです。
モデルのインスタンスをビューレイヤーに渡す代わりにDTOを作成するのはなぜですか？
主な理由と簡単な答えは、ここでも関心の分離です。
ビューにModelインスタンスを検査して使用させると、ViewレイヤーとModelレイヤーが緊密に結合されます。
実際、モデルレイヤーを変更すると、変更されたモデルインスタンスを利用するすべてのビューが破損する可能性があります。



{% extends "base.html.twig" %}

{% block content %}
  {% if errormsg is defined %}
  <div class="alert error">{{ errormsg }} </div>
  {% else %}
  <div class="alert success">Bravo! Post was created successfully!</div>
  {% endif %}
  <table>
    <thead><tr><th>ID</th><th>TITLE</th><th>ACTIONS</th></tr></thead>
    <tbody>
    {% for post in posts %}
      <tr>
        <td>{{ post.id }}</td>
        <td>{{ post.title }}</td>
        <td><a href="{{ editPostUrl(post.id) }}">Edit Post</a></td>
      </tr>
    {% endforeach %}
    </tbody>
  </table>
{% endblock %}



Most of the time, when the Model triggers a state change, it also notifies the related Views so that the UI can get refreshed.
In a typical web scenario the synchronization between the Model and its representations can be a bit tricky because of the client-server nature.
In this kind of environments some JavaScript defined interactions are usually needed to maintain that synchronization.
For this reason, JavaScript MVC frameworks like the ones below have become widely popular in recent years:

ほとんどの場合、モデルが状態変更をトリガーすると、UIを更新できるように、関連するビューにも通知します。
典型的なWebシナリオでは、モデルとその表現の間の同期は、クライアントサーバーの性質のために少し注意が必要な場合があります。
この種の環境では、通常、その同期を維持するためにJavaScriptで定義された対話が必要です。
このため、以下のようなJavaScriptMVCフレームワークが近年広く普及しています。


• AngularJS³
• EmberJS⁴
• Marionette⁵
• ReactJS⁶

³https://angularjs.org/
⁴http://emberjs.com/
⁵http://marionettejs.com/
⁶https://facebook.github.io/react/



2.2.2.3 The Controller

The Controller layer is responsible for organizing and orchestrating the View and the Model.
It receives messages from the View layer and triggers Model behaviour in order to perform the desired action.

コントローラーレイヤーは、ビューとモデルの整理と調整を担当します。
ビューレイヤーからメッセージを受信し、目的のアクションを実行するためにモデルの動作をトリガーします。

Furthermore, it sends messages to the View in order to display Model representations.
Both operations are performed thanks to the Application Layer, responsible for orchestrating, organizing and encapsulating domain behaviour.

さらに、モデル表現を表示するためにビューにメッセージを送信します。
両方の操作は、ドメインの動作の調整、編成、およびカプセル化を担当するアプリケーション層のおかげで実行されます。

In terms of a web application in PHP, the Controller usually comprehends a set of classes, which in order to fulfill their purpose “speak HTTP”.
That is, they receive an HTTP request and return an HTTP response.

PHPのWebアプリケーションに関しては、コントローラーは通常、「HTTPを話す」という目的を果たすために一連のクラスを理解します。
つまり、HTTP要求を受信し、HTTP応答を返します。



class PostsController
{
  public function updateAction(Request $request)
  {
    if ($request->request->has('submit')
    && Validator::validate($request->request->post)
    ) {
      // PostServiceオブジェクトを生成
      $postService = new PostService();

      try {
        // 記事を作成
        $postService->createPost(
          $request->request->get('title'),
          $request->request->get('content')
        );
        // 成功メッセージを表示
        $this->addFlash(
          'notice',
          'Post has been created successfully!'
        );
      } catch (Exception $e) {
        // エラーメッセージを表示
        $this->addFlash(
          'error',
          'Unable to create the post!'
        );
      }
    }
    // 描画する
    return $this->render('posts/update-result.html.twig');
  }
}



## 2.3 Hexagonal Architecture: Inverting Dependencies
Following the essential rule of Layered Architecture, there is a risk to end up implementing domain interfaces that need to make use of infrastructural concerns within the domain model layer.

階層化アーキテクチャの基本的なルールに従って、ドメインモデルレイヤー内のインフラストラクチャの懸念を利用する必要があるドメインインターフェイスを実装することになるリスクがあります。


As an example, the PostRepository from the previous example should be placed in the Domain Model if we were following MVC.
However, placing infrastructural details right in the middle of our domain violates separation of concerns.
This can result in issues, it is hard to avoid violating the essential rules of Layered Architecture, leading to a style of code which can become hard to test if the Domain Layer is aware of technical implementations.

例として、MVCをフォローしている場合は、前の例のPostRepositoryをドメインモデルに配置する必要があります。
ただし、インフラストラクチャの詳細をドメインの真ん中に配置すると、関心の分離に違反します。
これにより問題が発生する可能性があり、階層化アーキテクチャの基本的なルールに違反することを回避することは困難であり、ドメイン層が技術的な実装を認識している場合にテストが困難になる可能性のあるコードのスタイルにつながります。



### 2.3.1 The Dependency Inversion Principle (DIP)

How can we fix this? 
As the Domain Model layer depends on concrete infrastructure implementations, the Dependency Inversion Principle⁷ could be applied by relocating the Infrastructure layer on top of the other three layers.

どうすればこれを修正できますか？
ドメインモデルレイヤーは具体的なインフラストラクチャの実装に依存しているため、他の3つのレイヤーの上にインフラストラクチャレイヤーを再配置することで、依存性逆転の原則⁷を適用できます。
⁷http://www.objectmentor.com/resources/articles/dip.pdf

   →→ 
    レンタルサーバーのページ
    https://objectmentor.booked.net/
     にリダイレクトされる。
     つまり、もうページは存在していない、ということ。


=========▼▼（ここから）▼▼=========  

The Dependency Inversion Principle
依存性逆転の原則

High level modules should not depend upon low level modules.
Both should depend upon abstractions.
Abstractions should not depend upon details.
Details should depend upon abstractions.

高レベルのモジュールは、低レベルのモジュールに依存しないでください。
どちらも抽象化に依存する必要があります。
抽象化は詳細に依存するべきではありません。
詳細は抽象化に依存する必要があります。

Robert C. Martin
=========▲▲（ここまで）▲▲=========


By using the Dependency Inversion Principle, the architecture schema changes and the Infrastructure layer which can be referred to as low level modules now depend on the UI, the Application Layer and the Domain Layer, which are the high level modules.
The dependency has been inverted.

依存性逆転の原則を使用することにより、アーキテクチャスキーマが変更され、低レベルモジュールと呼ばれるインフラストラクチャ層が、高レベルモジュールであるUI、アプリケーション層、およびドメイン層に依存するようになりました。
依存関係が反転しました。

But then, what is Hexagonal Architecture?, 
and how does it fit within of all this? 
しかし、それでは、ヘキサゴナルアーキテクチャとは何ですか？
そしてそれはこれらすべての中にどのように適合しますか？

Hexagonal Architecture (also known as Ports and Adapters) was defined by Alistair Cockburn and represents the application as an hexagon where each side represents a Port with one or more Adapters.
A Port is a connector with a pluggable Adapter which transforms an outside input to something the inside application can understand.
In terms of the DIP, the Port would be a high level module and an Adapter would be a low level module.

ヘキサゴナルアーキテクチャ（ポートおよびアダプタとも呼ばれます）は、Alistair Cockburnによって定義され、アプリケーションを六角形として表します。各側面は、1つまたは複数のアダプタを備えたポートを表します。
ポートは、外部入力を内部アプリケーションが理解できるものに変換するプラグ可能なアダプタを備えたコネクタです。
DIPに関しては、ポートは高レベルのモジュールであり、アダプターは低レベルのモジュールです。

Furthermore, if the application needs to emit some message to the outside it will also use a Port with an Adapter to send it and transform it to something that the outside can understand.
For this reason, Hexagonal Architecture brings up the concept of symmetry in the application and it is also the main reason why the schema of the architecture changes.

さらに、アプリケーションが外部にメッセージを送信する必要がある場合は、アダプタ付きのポートを使用してメッセージを送信し、外部が理解できるものに変換します。
このため、ヘキサゴナルアーキテクチャはアプリケーションに対称性の概念をもたらし、アーキテクチャのスキーマが変更される主な理由でもあります。

It is often represented as a hexagon, because it does no longer make sense to talk about a “top” layer nor a “bottom” layer.
Instead, Hexagonal Architecture talks mainly in terms of the “outside” and the “inside”.

「上」の層や「下」の層について話すことはもはや意味がないため、六角形として表されることがよくあります。
代わりに、ヘキサゴナルアーキテクチャは主に「外側」と「内側」の観点から話します。


There are great videos on YouTube by Matthias Noback talking about Hexagonal Architecture.
You may take a look at one of those⁸.

六角形のアーキテクチャについて話しているMatthiasNobackによるすばらしいビデオがYouTubeにあります。
あなたはそれらの1つを見るかもしれません⁸。

⁸https://www.youtube.com/watch?v=K1EJBmwg9EQ



### 2.3.2 Applying Hexagonal Architecture

Continuing with the blog example application, 
ブログのサンプルアプリケーションで話を続けると、

the first concept we need is a Port where the outside world can talk to the application.
最初に必要な概念は、外の世界がアプリケーションと通信できるポートです。

For this case, we'll use an HTTP Port and its corresponding Adapter.
この場合、HTTPポートとそれに対応するアダプタを使用します。

The outside will use the Port to send messages to the application.
外部はポートを使用してアプリケーションにメッセージを送信します。

The blog example was using a database to store the whole collection of blog posts, so in order to allow the application to retrieve blog posts from the database, a Port is needed:

ブログの例では、データベースを使用してブログ投稿のコレクション全体を保存していたため、アプリケーションがデータベースからブログ投稿を取得できるようにするには、ポートが必要です。

  （松本）ポートとアダプターというのがわからない。
      アプリケーションがブログ投稿をデータベースから取得する？？
      
      アプリケーションというのは、アプリケーション層のことか？


/// レポジトリというのは、
/// アプリケーション(層)に対してデータを提供するためのポートのようなもの
interface PostRepository
{
  // ブログ取得メソッドの契約
  public function byId(PostId $id);
  // ブログ追加メソッドの契約
  public function add(Post $post);
}


interface PostRepository
{
  public function byId(PostId $id): Post;
  public function add(Post $post): Post;
}


This interface states the Port by which the application will retrieve information about blog posts, and it will be located in the Domain Layer.
このインターフェイスは、アプリケーションがブログ投稿に関する情報を取得するためのポートを示し、ドメインレイヤーに配置されます。

Now, an Adapter for this Port is needed.
The Adapter is in charge of defining the way in which the blog posts will be retrieved using a specific technology.

ここで、このポート用のアダプターが必要です。
アダプターは、特定のテクノロジーを使用してブログ投稿を取得する方法の定義を担当します。

  （松本解釈）
  ポート： 抽象クラスの契約, ドメイン層
  アダプター： 抽象クラスを具体的に実装したもの


class PDOPostRepository implements PostRepository
{
  private $db;
  public function __construct(PDO $db)
  {
    $this->db = $db;
  }

  // ブログ取得メソッドの実装
  public function byId(PostId $id)
  {
    $stm = $this->db->prepare(
    'SELECT * FROM posts WHERE id = ?'
    );
    $stm->execute([$id->id()]);
    return recreateFrom($stm->fetch());
  }

  // ブログ追加メソッドの実装
  public function add(Post $post)
  {
    $stm = $this->db->prepare(
      'INSERT INTO posts (title, content) VALUES (?, ?)'
    );
    
    $stm->execute([
      $post->title(),
      $post->content(),
    ]);
  }
}



【購入したPDF】

class PDOPostRepository implements PostRepository
{
  private \PDO $db;

  public function __construct(\PDO $db)
  {
    $this->db = $db;
  }

  // ブログ取得メソッドの実装
  public function byId(PostId $id): Post
  {
    $stmt = $this->db->prepare(
      'SELECT * FROM posts WHERE id = ?'
    );

    $stmt->execute([$id->id()]);

    $fetch = $stmt->fetch();
    return \mimic\hydrate(Post::class, [
    'id' => new PostId($fetch['id']),
    'title' => $fetch['title'],
    'content' => $fetch['content'],
    ]);
  }

  public function add(Post $post): Post
  {
    $stmt = $this->db->prepare(
    'INSERT INTO posts (title, content) VALUES (?, ?)'
    );

    $stmt->execute([
    $post->title(),
    $post->content(),
    ]);

    $post->setId(new PostId($this->db->lastInsertId()));

    return $post;
  }
}




Once we have the Port and its Adapter defined, the last step to do is to refactor the PostService class so that it uses them.
And this can be easily achieved by using Dependency Injection⁹

ポートとそのアダプタを定義したら、最後のステップは、PostServiceクラスをリファクタリングしてそれらを使用するようにすることです。
そして、これは依存性注入⁹を使用することで簡単に達成できます

⁵³http://www.martinfowler.com/articles/injection.html


  **古い版の方が正しいように思える**

【購入したPDF】
interface PostRepository
{
  public function byId(PostId $id): Post;
  public function add(Post $post): Post;
}


【章番号ありのPDF】
class PostService
{
  private $postRepository;
  
  // PostRepositoryをコンストラクタで受け取る
  // コンストラクタ引数による注入で『依存性注入』を実行している
  public function __construct(PostRepository $postRepository)
  {
    $this->postRepository = $postRepository;
  }
  
  // PostRepositoryのメソッドを実行する
  public function createPost($title, $content)
  {
    $post = Post::writeNewFrom($title, $content);
    $this->postRepository->add($post);
    return $post;
  }
}


This is just a simple example of Hexagonal Architecture.
これは、ヘキサゴナルアーキテクチャの簡単な例です。

It's a flexible architecture that promotes Separation of Concerns, like Layered Architecture.
これは、階層化アーキテクチャのように、関心の分離を促進する柔軟なアーキテクチャです。

It also promotes symmetry, due to having an inside application that communicates with the outside via ports.
また、ポートを介して外部と通信する内部アプリケーションがあるため、対称性が促進されます。

   ポート： レポジトリ抽象クラス
   外部： データベース
   
   アプリケーション層は、ポート： レポジトリ抽象クラス を使って
   外部： データベースと通信できる。
   
   symmetry の意味はわからない。。


From now on, this will be the foundational architecture used to build and explain CQRS and Event Sourcing.
For more examples about this architecture, you can check out the appendix.
For a more detailed example, you should jump to the Application chapter, which explains advanced topics like transactionality and other cross-cutting concerns.

これからは、これがCQRSとイベントソーシングの構築と説明に使用される基本的なアーキテクチャになります。
このアーキテクチャのその他の例については、付録をご覧ください。
より詳細な例については、アプリケーションの章にジャンプする必要があります。この章では、トランザクション性やその他の横断的関心事などの高度なトピックについて説明しています。




## 2.4 Command Query Responsibility Segregation

Hexagonal Architecture is a good foundational architecture but it has some limitations.
ヘキサゴナルアーキテクチャは優れた基本アーキテクチャですが、いくつかの制限があります。

For example, complex UIs can require Aggregate information displayed in diverse forms or they can require data obtained from multiple aggregates.
And in this scenario, we can end up with a lot of finder methods inside the Repositories (maybe as many as the UI views which exist within the application).
Or maybe we can decide to move this complexity to the Application Services - using complex structures to accumulate data from multiple aggregates.
Here is an example:

たとえば、複雑なUIでは、さまざまな形式で表示される集計情報が必要な場合や、複数の集計から取得されたデータが必要な場合があります。
そして、このシナリオでは、リポジトリ内に多くのファインダーメソッド（おそらくアプリケーション内に存在するUIビューと同じ数）が発生する可能性があります。
または、複雑な構造を使用して複数の集計からデータを蓄積することで、この複雑さをアプリケーションサービスに移行することもできます。
次に例を示します。


interface PostRepository
{
  public function save(Post $post): void;
  public function byId(PostId $id): Post;
  public function all(): array;
  public function byCategory(CategoryId $categoryId): Post;
  public function byTag(TagId $tagId): array;
  public function withComments(PostId $id): Post;
  public function groupedByMonth(): array;
   // ...
}



When these techniques are abused, the construction of the UI views can become really painful and we should evaluate the trade-offs between making Application Services return domain model instances and using some kind of Data Transfer Object (DTO) in order to avoid tight coupling between the Domain Model and infrastructure code like web controllers, CLI controllers, and so on.

これらの手法が悪用されると、UIビューの構築が非常に困難になる可能性があるため、
  Application Servicesがドメインモデルインスタンスを返すようにすることと、
  ある種のデータ転送オブジェクト（DTO）を使用することの間のトレードオフを評価して、

ドメインモデルと、Webコントローラー、CLIコントローラーなどのインフラストラクチャコード
との間の密結合を避けるために、

（松本：in order to avoid tight coupling がどこに掛かっているのかがわからない。）
  we should evaluate the trade-offs に掛かっているのか、
  using some kind of Data Transfer Object (DTO) に掛かっているのか。


Luckily, there is another approach.
If the problem is having multiple and disparate views, we can exclude them from the Domain Model and start treating them as a purely infrastructural concern.

幸いなことに、別のアプローチがあります。
問題が複数の異なるビューを持っている場合は、それらをドメインモデルから除外し、純粋にインフラストラクチャの問題として扱い始めることができます。

This option is based on a design principle, named Command Query Separation CQS, defined by Bertrand Meyer which gave birth to a new architectural pattern named Command Query Responsibility Segregation defined by Greg Young.

このオプションは、GregYoungによって定義されたCommandQuery ResponsibilitySegregationという名前の新しいアーキテクチャパターンを生み出したBertrandMeyerによって定義されたCommandQuery Separation CQSという名前の設計原則に基づいています。


=========▼▼（ここから）▼▼=========  
Command Query Separation (CQS)

“Asking a question should not change the answer” – Bertrand Meyer

「質問しても答えは変わらない」–バートランドメイヤー

This design principle states that every method should be either a Command, that performs an action, or a Query, that returns data to the caller, but not both.

この設計原則では、すべてのメソッドは、アクションを実行するコマンド、またはデータを呼び出し元に返すクエリのいずれかである必要がありますが、両方である必要はありません。
=========▲▲（ここまで）▲▲=========

CQRS seeks an even more aggressive separation of concerns splitting the Model in two:
CQRSは、モデルを2つに分割する さらに積極的な関心の分離を求めています。

  separation of concerns -- 関心の分離


• The Write Model: Also known as the Command Model, it performs the writes and takes responsibility for the true domain behaviour.

• The Read Model: It takes responsibility of the reads within the application and treats them as something that should be out of the Domain Model.

•書き込みモデル：コマンドモデルとも呼ばれ、書き込みを実行し、実際のドメインの動作に責任を負います。

•読み取りモデル：アプリケーション内の読み取りを担当し、ドメインモデル外の読み取りとして扱います。

Every time someone triggers a command to the write model, this performs the write to the desired datastore and additionally triggers the read model update in order to display the latest changes on the read model.

誰かが書き込みモデルへのコマンドをトリガーするたびに、これは目的のデータストアへの書き込みを実行し、さらに読み取りモデルの最新の変更を表示するために読み取りモデルの更新をトリガーします。


This strict separation triggers another problem, Eventual Consistency.
この厳密な分離は、別の問題、結果整合性を引き起こします。

The consistency of the read model now is subject to the commands performed by the write model.
In other words, it is said that the read model is eventually consistent.
This is, every time the write model performs a command it will pull up a process that will be responsible to update the read model according to the last updates on the write model.

読み取りモデルの一貫性は、書き込みモデルによって実行されるコマンドの影響を受けるようになりました。
言い換えれば、読み取りモデルは結果整合性があると言われます。
つまり、書き込みモデルがコマンドを実行するたびに、
  書き込みモデルの最後の更新に従って読み取りモデルを更新するプロセスをプルアップします。


There is such a window of time were the UI may present stale information to the user.
In the web scenario this happens often as we are somewhat limited by the current technologies.
Think about a caching system in front of a web application.
Every time the database is updated with new information, the data on the cache layer may potentially be stale, so every time it gets updated there should be a process that updates the cache system.

UIがユーザーに古い情報を提示する可能性があるような時間枠があります。
Webシナリオでは、現在のテクノロジーによって多少制限されているため、これは頻繁に発生します。
Webアプリケーションの前にあるキャッシングシステムについて考えてみてください。
データベースが新しい情報で更新されるたびに、キャッシュレイヤーのデータが古くなる可能性があるため、データベースが更新されるたびに、キャッシュシステムを更新するプロセスが必要になります。

Cache systems are eventually consistent.
This kind of processes, speaking in CQRS terminology, are called Write Model Projections or just Projections.

キャッシュシステムは結果整合性があります。
この種のプロセスは、CQRSの用語で言えば、
  モデルプロジェクションの書き込みまたは単にプロジェクションと呼ばれます。
  
 （松本：プロジェクトとは、投影する、という意味）
   書き込みモデルを読み込みモデルに投影する、つまり、反映させる、という意味だと思う。

We project the write model onto the read model.
This process can be synchronous or asynchronous, depending on your needs, and it can be done thanks to another useful tactical design pattern that will be explained in detail later on in the book, Domain Events.

書き込みモデルを読み取りモデルに投影します。
このプロセスは、ニーズに応じて同期または非同期にすることができます。
これは、本の後半で詳細に説明する別の便利な戦術設計パターン、ドメインイベントのおかげで実行できます。

The basis of the write model projections is to gather all the published domain events and update the read model with all the information coming from the events.

書き込みモデルの予測の基本は、公開されているすべてのドメインイベントを収集し、イベントからのすべての情報で読み取りモデルを更新することです。



### 2.4.1 The Write Model
書き込みモデル

This is the true holder of Domain behavior.
Continuing with our example, 
  the Repository interface would be simplified to the following:

これは、ドメイン動作の真の所有者です。
この例で話を続けると、
 リポジトリインターフェイスは次のように簡略化されます。


interface PostRepository
{
  public function save(Post $post): void;
  public function byId(PostId $id): Post;
}



Now the PostRepository has been freed from all the read concerns except one: the byId, which is responsible for loading the Aggregate by its ID so that we can operate on it.
And once this is done, all the query methods are also stripped down from the Post model, leaving it only with command methods.
This means we'll effectively get rid of all the getter methods and any other methods exposing information about the Post Aggregate.

これで、PostRepositoryは、1つを除くすべての読み取りの懸念から解放されました。
byIdは、IDで集計をロードして、操作できるようにする役割を果たします。
そして、これが行われると、すべてのクエリメソッドもPostモデルから削除され、コマンドメソッドのみが残ります。
これは、PostAggregateに関する情報を公開するすべてのgetterメソッドおよびその他のメソッドを効果的に削除することを意味します。


  （↑↑ 松本 わからない。。）

Instead, Domain Events will be published in order to be able to trigger write model projections by subscribing to them:
代わりに、ドメインイベントは、サブスクライブすることで書き込みモデルの投影をトリガーできるようにするために公開されます。


class AggregateRoot
{
  private array $recordedEvents = [];

  protected function recordApplyAndPublishThat(DomainEvent $event): void
  {
    $this->recordThat($event);
    $this->applyThat($event);
    $this->publishThat($event);
  }

  protected function recordThat(DomainEvent $event): void
  {
    $this->recordedEvents[] = $event;
  }

  protected function applyThat(DomainEvent $event): void
  {
    $className = (new \ReflectionClass($event))->getShortName();

    $modifier = 'apply' . $className;

    $this->$modifier($event);
  }

  protected function publishThat(DomainEvent $event): void
  {
    DomainEventPublisher::instance()->publish($event);
  }

  public function recordedEvents(): array
  {
    return $this->recordedEvents;
  }

  public function clearEvents(): void
  {
    $this->recordedEvents = [];
  }
}



class Post extends AggregateRoot
{
  // ...
  public static function writeNewFrom(string $title, string $content): self
  {
    $postId = PostId::create();
    $post = new static($postId);
    $post->recordApplyAndPublishThat(
      new PostWasCreated($postId, $title, $content)
    );
    return $post;
  }
  public function publish(): void
  {
    $this->recordApplyAndPublishThat(
      new PostWasPublished($this->id)
    );
  }
  public function categorizeIn(CategoryId $categoryId): void
  {
    $this->recordApplyAndPublishThat(
      new PostWasCategorized($this->id, $categoryId)
    );
  }
  public function changeContentFor(string $newContent): void
  {
    $this->recordApplyAndPublishThat(
      new PostContentWasChanged($this->id, $newContent)
    );
  }
  public function changeTitleFor(string $newTitle): void
  {
    $this->recordApplyAndPublishThat(
      new PostTitleWasChanged($this->id, $newTitle)
    );
  }
  protected function applyPostWasCreated(PostWasCreated $event): void
  {
    $this->id = $event->postId();
    $this->title = $event->title();
    $this->content = $event->content();
  }
  protected function applyPostWasPublished(PostWasPublished $event): void
  {
    $this->published = true;
  }
  protected function applyPostWasCategorized(PostWasCategorized $event): void
  {
    $this->categories[$event->categoryId()->id()] = $event->categoryId();
  }
  protected function applyPostContentWasChanged(PostContentWasChanged $event): void
  {
    $this->content = $event->content();
  }
  protected function applyPostTitleWasChanged(PostTitleWasChanged $event): void
  {
    $this->title = $event->title();
  }
}


Take note, all actions that trigger a state change are implemented via Domain Events.
For each Domain Event published, there's an apply method responsible for reflecting the state change.

状態の変更をトリガーするすべてのアクションは、ドメインイベントを介して実装されることに注意してください。
公開されたドメインイベントごとに、状態の変化を反映するための適用メソッドがあります。




### The Read Model
読み取りモデル

The read model, also known as the Query Model, is a pure denormalized data model lifted from Domain concerns.
クエリモデルとも呼ばれる読み取りモデルは、ドメインの懸念から解放された純粋な非正規化データモデルです。


In fact, with CQRS, all the read concerns are treated as reporting processes, an infrastructure concern.
In general, when using CQRS, the read model is subject to the needs of the UI and how complex the views compounding the UI are.

実際、CQRSでは、読み取られたすべての懸念事項は、インフラストラクチャの懸念事項であるレポートプロセスとして扱われます。
一般に、CQRSを使用する場合、読み取りモデルはUIのニーズと、UIを構成するビューの複雑さの影響を受けます。

In a situation where the read model is defined in terms of relational databases, the simplest approach would be to set one-to-one relationships between database tables and UI views.
These database tables and UI views will be updated using write model projections triggered from the Domain Events published by the write side:

読み取りモデルがリレーショナルデータベースの観点から定義されている状況では、最も簡単なアプローチは、データベーステーブルとUIビューの間に1対1の関係を設定することです。
これらのデータベーステーブルとUIビューは、書き込み側によって公開されたドメインイベントからトリガーされた書き込みモデルプロジェクションによって更新されます。


-- Definition of a UI view of a single post with its comments
CREATE TABLE single_post_with_comments (
  id INTEGER NOT NULL,
  post_id INTEGER NOT NULL,
  post_title VARCHAR(100) NOT NULL,
  post_content TEXT NOT NULL,
  post_created_at DATETIME NOT NULL,
  comment_content TEXT NOT NULL
);

-- Set up some data
INSERT INTO single_post_with_comments VALUES
  (1, 1, "Layered architecture", "Lorem ipsum ...", NOW(), "Lorem ipsum ..."),
  (2, 1, "Layered architecture", "Lorem ipsum ...", NOW(), "Lorem ipsum ..."),
  (3, 2, "Hexagonal architecture", "Lorem ipsum ...", NOW(), "Lorem ipsum ..."),
  (4, 2, "Hexagonal architecture", "Lorem ipsum ...", NOW(), "Lorem ipsum ..."),
  (5, 3, "CQRS", "Lorem ipsum ...", NOW(), "Lorem ipsum ..."),
  (6, 3, "CQRS", "Lorem ipsum ...", NOW(), "Lorem ipsum ...");

-- Query it
SELECT * FROM single_post_with_comments WHERE post_id = 1;


An important feature of this architectural style is that the read model should be completely disposable, since the true state of the application is handled by the write model.
This means the read model can be removed and recreated when needed, using write model projections.
Here we can see some examples of possible views within a blog application:

このアーキテクチャスタイルの重要な機能は、
アプリケーションの実際の状態が書き込みモデルによって処理されるため、
読み取りモデルは完全に使い捨てである必要があることです。

これは、書き込みモデルのプロジェクションを使用して、必要に応じて読み取りモデルを削除および再作成できることを意味します。
ここでは、ブログアプリケーション内で可能なビューの例をいくつか見ることができます。



SELECT * FROM posts_grouped_by_month_and_year ORDER BY month DESC, year ASC;
SELECT * FROM posts_by_tags WHERE tag = "ddd";
SELECT * FROM posts_by_author WHERE author_id = 1;


It's important to point out that CQRS doesn't constrain the definition and implementation of the read model to a relational database.
CQRSは、読み取りモデルの定義と実装をリレーショナルデータベースに制約しないことを指摘することが重要です。

It depends exclusively on the needs of the application being built.
It could be a relational database, a document-oriented database, a key-value store, or whatever best suits the needs of your application.


これは、構築するアプリケーションのニーズにのみ依存します。
これは、リレーショナルデータベース、ドキュメント指向データベース、Key-Valueストア、またはアプリケーションのニーズに最適なものであれば何でもかまいません。

Following the blog post application, we'll use Elasticsearch⁵⁵ — a document-oriented database — to implement a read model:
ブログ投稿アプリケーションに続いて、Elasticsearch⁵⁵（ドキュメント指向データベース）を使用して、読み取りモデルを実装します。


class PostController
{
  // すべてのブログ投稿情報を取得する
  public function listAction(): array
  {
    $client = \Elasticsearch\ClientBuilder::create()->build();

    $response = $client->search([
      'index' => 'posts',
      'type' => 'post',
      'body' => [
      'sort' => [
        'created_at' => ['order' => 'desc']
      ]
      ]
    ]);

    return [
      'posts' => $response
    ];
  }
}



The read model code has been drastically simplified to a single query against an Elasticsearch index.
読み取られたモデルコードは、Elasticsearchインデックスに対する単一のクエリに大幅に簡素化されました。

This reveals that the read model doesn't really need an object-relational mapper, as this might be overkill.
これは、読み取りモデルが実際にはオブジェクトリレーショナルマッパーを必要としないことを示しています。これはやり過ぎかもしれないからです。

However, the write model might benefit from the use of an object-relational mapper, as this would allow you to organize and structure the read model according to the needs of the application.
ただし、書き込みモデルは、オブジェクトリレーショナルマッパーを使用することでメリットが得られる場合があります。これにより、アプリケーションのニーズに応じて読み取りモデルを整理および構造化できるためです。




### Synchronizing the Write Model with the Read Model
    書き込みモデルと読み取りモデルの同期


Here comes the tricky part.
ここに注意が必要な部分があります。

How do we synchronize the read model with the write model? 
読み取りモデルを書き込みモデルと同期するにはどうすればよいですか？

We already said we would do it by using Domain Events captured in a write model transaction.
For each type of Domain Event captured, a specific projection will be executed.
So a one-to-one relationship between Domain Events and projections will be set.
Let's have a look at an example of configuring projections so that we can get a better idea.
First of all, we need to define a skeleton for the projections:

書き込みモデルトランザクションでキャプチャされたドメインイベントを使用してこれを行うことはすでに述べました。
キャプチャされたドメインイベントのタイプごとに、特定のプロジェクション(投影)が実行されます。
そのため、ドメインイベントとプロジェクション(投影)の間に1対1の関係が設定されます。
より良いアイデアを得ることができるように、プロジェクション(投影)を構成する例を見てみましょう。
まず、プロジェクション(投影)のスケルトンを定義する必要があります。


/// プロジェクション(投影)のスケルトンを定義
interface Projection
{
  public function listensTo(): string;
  // 投影する
  public function project(DomainEvent $event): void;
}


So defining an Elasticsearch projection for a PostWasCreated event would be as simple as this:

したがって、PostWasCreatedイベントのElasticsearchプロジェクションの定義は、次のように簡単になります。


/// PostWasCreatedイベントのElasticsearchプロジェクションの定義
class PostWasCreatedProjection implements Projection
{
  private Client $client;

  public function __construct(Client $client)
  {
  $this->client = $client;
  }

  // PostWasCreatedイベントが発生したか
  public function listensTo(): string
  {
    return PostWasCreated::class;
  }

  // DomainEventに基づき 投影させる
  // インデックスを更新する
  public function project(DomainEvent $event): void
  {
    $this->client->index([
      'index' => 'posts',
      'type' => 'post',
      'id' => $event->postId()->id(),
      'body' => [
        'title' => $event->title(),
        'content' => $event->content()
      ]
    ]);
  }
}



The Projector implementation is a kind of specialized Domain Event listener.
The main difference between that and the default Domain Event listener is that the Projector reacts to a group of Domain Events instead of only one:

Projectorの実装は、一種の特殊なドメインイベントリスナーです。
これとデフォルトのドメインイベントリスナーの主な違いは、プロジェクターが1つだけではなくドメインイベントのグループに反応することです。



class Projector
{
  private array $projections = [];

  public function register(array $projections): void
  {
    foreach ($projections as $projection) {
      $this->projections[$projection->listensTo()] = $projection;
    }
  }

  public function project(array $events): void
  {
    foreach ($events as $event) {
      if (isset($this->projections[get_class($event)])) {
        $this->projections[get_class($event)]->project($event);
      }
    }
  }
}


The following code shows how the flow between the projector and the events would appear:

次のコードは、プロジェクターとイベントの間のフローがどのように表示されるかを示しています。



$client = \Elasticsearch\ClientBuilder::create()->build();

$projector = new Projector();

$projector->register([
  new PostWasCreatedProjection($client),
  new PostWasPublishedProjection($client),
  new PostWasCategorizedProjection($client),
  new PostTitleWasChangedProjection($client),
  new PostContentWasChangedProjection($client)
]);

$postId = PostId::create();
$categoryId = CategoryId::create();

$projector->project([
  new PostWasCreated($postId, 'A title', 'Some content'),
  new PostWasPublished($postId),
  new PostWasCategorized($postId, $categoryId),
  new PostTitleWasChanged($postId, 'New title'),
  new PostContentWasChanged($postId, 'New content'),
]);



This code is kind of synchronous, but the process can be asynchronous if needed.
And you could make your customers aware of this out-of-sync data by placing some alerts in the view layer.
For the next example, we'll use Bunny⁵⁶ library to emit events asynchronously.

このコードは一種の同期ですが、必要に応じてプロセスを非同期にすることができます。
また、ビューレイヤーにアラートを配置することで、この非同期データを顧客に認識させることができます。
次の例では、Bunny⁵⁶ライブラリを使用してイベントを非同期に発行します。



/// Bunny⁵⁶ライブラリを使用してイベントを非同期に発行する
 // Bunnyライブラリを使用
$bunny = (new \Bunny\Client())->connect();
$channel = $bunny->channel();
$channel->exchangeDeclare('events', 'fanout');

$serializer = new \Zumba\JsonSerializer\JsonSerializer();

$postId = PostId::create();
$categoryId = CategoryId::create();
$projector = new AsyncProjector($channel, $serializer);
$projector->project([
  new PostWasCreated($postId, 'A title', 'Some content'),
  new PostWasPublished($postId),
  new PostWasCategorized($postId, $categoryId),
  new PostTitleWasChanged($postId, 'New title'),
  new PostContentWasChanged($postId, 'New content'),
]);

$channel->close();
$bunny->disconnect();


For this to work, we need an asynchronous projector.
Here's a naive implementation of that, we've used Zumba JsonSerializer⁵⁷ for serialization:

これを機能させるには、非同期プロジェクターが必要です。
これはその素朴な実装です。シリアル化にZumbaJsonSerializer⁵⁷を使用しました。

⁵⁶https://github.com/jakubkulhan/bunny
⁵⁷https://github.com/zumba/json-serializer



class AsyncProjector
{
  private Channel $channel;
  private JsonSerializer $serializer;

  public function __construct(
    Channel $channel,
    JsonSerializer $serializer
  ) {
    $this->channel = $channel;
    $this->serializer = $serializer;
  }

  public function project(array $events): void
  {
    foreach ($events as $event) {
      $this->channel->publish(
        $this->serializer->serialize($event),
        [],
        'events'
      );
    }
  }
}


And the event consumer on the RabbitMQ exchange would look something like this:

また、RabbitMQエクスチェンジのイベントコンシューマーは次のようになります。



$client = \Elasticsearch\ClientBuilder::create()->build();

$projector = new Projector();
$projector->register([
  new PostWasCreatedProjection($client),
  new PostWasPublishedProjection($client),
  new PostWasCategorizedProjection($client),
  new PostTitleWasChangedProjection($client),
  new PostContentWasChangedProjection($client)
]);

$serializer = new \Zumba\JsonSerializer\JsonSerializer();

$bunny = (new \Bunny\Client())->connect();
$channel = $bunny->channel();
$channel->exchangeDeclare('events', 'fanout');
$queue = $channel->queueDeclare('queue');
$channel->queueBind($queue->queue, 'events');

$channel->consume(
  function (
    \Bunny\Message $message,
    \Bunny\Channel $channel,
    \Bunny\Client $client
  ) use ($serializer, $projector) {
    $event = $serializer->unserialize($message->content);
    $projector->project([$event]);
  },
  $queue->queue
);

$bunny->run();


From now on, it could be as simple as making all the needed Repositories consume an instance of the projector and then making them invoke the projection process:

これからは、必要なすべてのリポジトリにプロジェクターのインスタンスを消費させてから、それらにプロジェクションプロセスを呼び出させるのと同じくらい簡単になる可能性があります。



class DoctrinePostRepository implements PostRepository
{
  private EntityManager $em; // EntityManagerはここで初登場
  private Projector $projector;

  public function __construct(EntityManager $em, Projector $projector)
  {
    $this->em = $em;
    $this->projector = $projector;
  }

  public function save(Post $post): void
  {
    $this->em->transactional(function (EntityManager $em) use ($post) {
      // 投稿内容を永続化する
      $em->persist($post);

      // すべてのイベントを永続化する
      foreach ($post->recordedEvents() as $event) {
        $em->persist($event);
      }
    });

    // すべてのイベントを読み取りモデルに投影
    $this->projector->project($post->recordedEvents());
  }

  public function byId(PostId $id): Post
  {
    return $this->em->find(Post::class, $id);
  }
}


The Post instance and the recorded events are triggered and persisted in the same transaction.
This ensures that no events are lost, as we'll project them to the read model if the transaction is successful.
As a result, no inconsistencies will exist between the write model and the read model.

Postインスタンスと記録されたイベントは、
  同じトランザクションでトリガーされおよび永続化されます。
これにより、トランザクションが成功した場合にイベントを読み取りモデルに投影するため、イベントが失われることはありません。
その結果、書き込みモデルと読み取りモデルの間に矛盾は存在しません。



### To ORM or Not To ORM (I)

One of the most common questions when implementing CQRS is if an Object-Relational Mapper (ORM) is really needed.
We strongly believe that using an ORM for the write model is perfectly fine and has all of the advantages of using a tool, which will help us save a lot of work in case we use a relational database.
But we shouldn't forget that we still need to persist and retrieve the write model's state in a relational database.

CQRSを実装する際の最も一般的な質問の1つは、
オブジェクトリレーショナルマッパー（ORM）が本当に必要かどうかです。

書き込みモデルにORMを使用することはまったく問題がなく、
ツールを使用することのすべての利点があると強く信じています。
これにより、リレーショナルデータベースを使用する場合に多くの作業を節約できます。
ただし、リレーショナルデータベースで書き込みモデルの状態を永続化して取得する必要があることを忘れてはなりません。



## 2.5 Event Sourcing


CQRS is a powerful and flexible architecture.
CQRSは、強力で柔軟なアーキテクチャです。

There's an added benefit to it in regard to gathering and saving the Domain Events (which occurred during an Aggregate operation), giving you a highlevel degree of detail of what's going on within your Domain.
Domain Events are one of the key tactical patterns because of their significance within the Domain, as they describe past occurrences.

ドメインイベント（集約操作中に発生した）の収集と保存に関して追加の利点があり、ドメイン内で何が起こっているかについての高レベルの詳細を提供します。
ドメインイベントは、過去の発生を説明するため、ドメイン内での重要性から、重要な戦術パターンの1つです。


Be Careful with Recording Too Many Events
あまりにも多くのイベントを記録することに注意してください

An ever-growing number of events is a smell.
It might reveal an addiction to event recording at the Domain, most likely incentivized by the business.
As a rule of thumb, remember to keep it simple.

増え続けるイベントは匂いです。
それは、おそらくビジネスによって動機付けられた、ドメインでのイベント記録への中毒を明らかにするかもしれません。
経験則として、それを単純に保つことを忘れないでください。



By using CQRS, we've been able to record all the relevant events that occurred in the Domain Layer.
The state of the Domain Model can be represented by reproducing the Domain Events we previously recorded.
We just need a tool for storing all those events in a consistent way.
We need an event store.

CQRSを使用することで、ドメインレイヤーで発生したすべての関連イベントを記録することができました。
ドメインモデルの状態は、以前に記録したドメインイベントを再現することで表すことができます。
これらすべてのイベントを一貫した方法で保存するためのツールが必要です。
イベントストアが必要です。


The fundamental idea behind Event Sourcing is to express the state of Aggregates as a linear sequence of events

イベントソーシングの背後にある基本的な考え方は、集約の状態をイベントの線形シーケンスとして表現することです。


With CQRS, we partially achieved the following: the Post entity alters its state by using Domain Events, but it's persisted, as explained already, thereby mapping the object to a database table.

CQRSを使用して、次のことを部分的に実現しました。Postエンティティはドメインイベントを使用して状態を変更しますが、すでに説明したように永続化されるため、オブジェクトがデータベーステーブルにマッピングされます。


Event Sourcing takes this a step further.
イベントソーシングはこれをさらに一歩進めます。

If we were using a database table to store the state of all the blog posts, another to store the state of all the blog post comments, and so on, using Event Sourcing would allow us to use a single database table: a single append-only database table that would store all the Domain Events published by all the Aggregates within the Domain Model.

データベーステーブルを使用してすべてのブログ投稿の状態を保存し、別のテーブルを使用してすべてのブログ投稿コメントの状態を保存する場合など、イベントソーシングを使用すると、単一のデータベーステーブルを使用できます。

ドメインモデル内のすべてのアグリゲートによって公開されたすべてのドメインイベントを格納するデータベーステーブルのみ。

Yes, you read that correctly.
A single database table.
はい、あなたはそれを正しく読んでいます。
単一のデータベーステーブル。

With this model in mind, tools like object-relational mappers are no longer needed.
The only tool needed would be a simple database abstraction layer by which events can be appended:

このモデルを念頭に置くと、オブジェクトリレーショナルマッパーのようなツールは不要になります。
必要な唯一のツールは、イベントを追加できる単純なデータベース抽象化レイヤーです。


/// イベントを追加できる単純なデータベース抽象化レイヤー
abstract class EventSourcedAggregateRoot extends AggregateRoot
{
  // イベントを追加する
  public abstract static function reconstitute(EventStream $events):
    EventSourcedAggregateRoot;

  // 状態を段階的に再生できるメソッド
  public function replay(EventStream $history): void
  {
    foreach ($history as $event) {
      $this->applyThat($event);
    }
  }
}


/// 上のデータベース抽象化レイヤーを実装したクラス
class Post extends EventSourcedAggregateRoot
{
  // ...

  public static function reconstitute(EventStream $history):
    EventSourcedAggregateRoot
  {
    $post = new static(new PostId($history->getAggregateId()));

    // 状態を段階的に再生
    $post->replay($history);

    return $post;
  }
}


Now the Post Aggregate has a method which, when given a set of events (or, in other words, an event stream), is able to replay the state step by step until it reaches the current state, all before saving.

これで、Post Aggregateには、一連のイベント（つまり、イベントストリーム）が与えられたときに、現在の状態に達するまで、すべて保存する前に、状態を段階的に再生できるメソッドがあります。


The next step would be building an adapter of the PostRepository port that will fetch all the published events from the Post Aggregate and append them to the data store where all the events are appended.

This is what we call an event store:

次のステップは、PostRepositoryポートのアダプターを構築して、公開されたすべてのイベントをPost Aggregateからフェッチし、すべてのイベントが追加されるデータストアに追加することです。
これが、私たちがイベントストアと呼んでいるものです。



class EventStorePostRepository implements PostRepository
{
  private EventStore $eventStore;
  private Projector $projector;

  public function __construct(EventStore $eventStore, Projector $projector)
  {
    $this->eventStore = $eventStore;
    $this->projector = $projector;
  }

  public function save(Post $post): void
  {
    $events = $post->recordedEvents();

    // イベントストアにデータを保存する
    $this->eventStore->append(new EventStream($post->id()->id(), $events));
    $post->clearEvents();

    // 読み取りオブジェクトに投影する
    $this->projector->project($events);
  }

  public function byId(PostId $id): Post
  {
    return Post::reconstitute(
      $this->eventStore->getEventsFor($id->id())
    );
  }
}



This is how the implementation of the PostRepository looks when we use an event store to save all the events published by the Post Aggregate.
We have included a way to restore an Aggregate from its events history.
A reconstitute method implemented by the Post Aggregate and used to rebuild a blog post state from triggered events comes in handy.
The event store is the workhorse that carries out all the responsibility in regard to saving and restoring event streams.
Its public API is composed of two simple methods: append and getEventsFrom.

これは、イベントストアを使用してPost Aggregateによって公開されたすべてのイベントを保存するときに、PostRepositoryの実装がどのように見えるかを示しています。

イベント履歴からアグリゲートを復元する方法が含まれています。
Post Aggregateによって実装され、トリガーされたイベントからブログ投稿の状態を再構築するために使用される再構成メソッドが役立ちます。
イベントストアは、イベントストリームの保存と復元に関するすべての責任を実行する主力製品です。
そのパブリックAPIは、appendとgetEventsFromの2つの単純なメソッドで構成されています。


The former appends an event stream to the event store, and the latter loads event streams to allow Aggregate rebuilding.
We could use a key-value implementation to store all events:

前者はイベントストリームをイベントストアに追加し、後者はイベントストリームをロードしてAggregateの再構築を可能にします。
Key-Value実装を使用して、すべてのイベントを格納できます。



class EventStore
{
  private Client $redis;
  private JsonSerializer $serializer;

  public function __construct(Client $redis, JsonSerializer $serializer)
  {
    $this->redis = $redis;
    $this->serializer = $serializer;
  }

  public function append(EventStream $eventstream): void
  {
    foreach ($eventstream as $event) {
      $data = $this->serializer->serialize($event);

      $date = (new \DateTimeImmutable())->format('YmdHis');

      $this->redis->rpush(
        'events:' . $eventstream->getAggregateId(),
        $this->serializer->serialize([
          'type' => get_class($event),
          'created_on' => $date,
          'data' => $data
        ])
      );
    }
  }

  public function getEventsFor(string $id): EventStream
  {
  return $this->fromVersion($id, 0);
  }

  public function fromVersion(string $id, int $version): EventStream
  {
    $serializedEvents = $this->redis->lrange(
      'events:' . $id,
      $version,
      -1
    );

    $eventStream = [];

    foreach ($serializedEvents as $serializedEvent) {
      $eventData = $this->serializer->unserialize($serializedEvent);

      $eventStream[] = $this->serializer->unserialize($eventData['data']);
    }

    return new EventStream($id, $eventStream);
  }

  public function countEventsFor(string $id): int
  {
    return $this->redis->llen('events:' . $id);
  }
}



This event store implementation is built upon Redis⁵⁸, a widely used key-value store.
We can use Predis⁵⁹ as a client for Redis and Zumba JsonSerializer⁶⁰ for event serialization.
The events are appended in a list using the prefix events:.
In addition, before persisting the events, we extract some metadata like the event class or the creation date, as it will come in handy later.

このイベントストアの実装は、広く使用されているKey-ValueストアであるRedis⁵⁸に基づいて構築されています。
⁵⁸http://redis.io


Predis⁵⁹をRedisのクライアントとして使用し、ZumbaJsonSerializer⁶⁰をイベントのシリアル化に使用できます。
⁵⁹https://github.com/nrk/predis
⁶⁰https://github.com/zumba/json-serializer


イベントは、プレフィックスeventsを使用してリストに追加されます。
さらに、イベントを永続化する前に、後で役立つように、イベントクラスや作成日などのメタデータを抽出します。


Obviously, in terms of performance, it's expensive for an Aggregate to go over its full event history to reach its final state all of the time.
This is especially the case when an event stream has hundreds or even thousands of events.

明らかに、パフォーマンスの観点から、Aggregateがイベント履歴全体を調べて、常に最終状態に到達するのはコストがかかります。
これは特に、イベントストリームに数百または数千ものイベントがある場合に当てはまります。


The best way to overcome this situation is to take a snapshot from the Aggregate and replay only the events in the event stream that occurred after the snapshot was taken.
A snapshot is just a simple serialized version of the Aggregate state at any given moment.
It can be based on the number of events of the Aggregate's event stream, or it can be time based.

この状況を克服する最善の方法は、Aggregateからスナップショットを取得し、スナップショットの作成後に発生したイベントストリーム内のイベントのみを再生することです。
スナップショットは、任意の時点での集約状態の単純なシリアル化バージョンです。
これは、Aggregateのイベントストリームのイベント数に基づくことも、時間ベースにすることもできます。


With the first approach, a snapshot will be taken every n triggered events (every 50, 100, or 200 events, for example).

最初のアプローチでは、n個のトリガーされたイベントごと（たとえば、50、100、または200イベントごと）にスナップショットが作成されます。

With the second approach, a snapshot will be taken every n seconds.
2番目のアプローチでは、スナップショットがn秒ごとに作成されます。

To follow the example, we'll use the first way of snapshotting.
In the event's metadata, we store an additional field, the version, from which we'll start replaying the Aggregate history:

例に従うために、スナップショットの最初の方法を使用します。
イベントのメタデータには、追加のフィールドであるバージョンが格納されており、そこから集計履歴の再生が開始されます。

```php
class SnapshotRepository
{
  // 依存パッケージを受け取る
  private Client $redis;
  private JsonSerializer $serializer;

  public function __construct(Client $redis, JsonSerializer $serializer)
  {
    $this->redis = $redis;
    $this->serializer = $serializer;
  }

  // IDを指定してスナップショットを取得する
  public function byId(string $id): ?Snapshot
  {
    $key = 'snapshots:' . $id;

    $data = $this->redis->get($key);

    if (null === $data) {
      return null;
    }

    $metadata = $this->serializer->unserialize($data);

    return new Snapshot(
      $this->serializer->unserialize(
        $metadata['snapshot']['data']
      ),
      $metadata['version']
    );
  }

  // スナップショットを保存する
  public function save(string $id, Snapshot $snapshot): void
  {
    $key = 'snapshots:' . $id;
    $aggregate = $snapshot->aggregate();

    // スナップショットを作成
    $snapshot = [
      'version' => $snapshot->version(),
      'snapshot' => [
        'type' => get_class($aggregate),
        'data' => $this->serializer->serialize(
          $aggregate
        )
      ]
    ];

    // Key-ValueストアRedisにシリアライズ化されたスナップショットを投入する
    $this->redis->set(
      $key,
      $this->serializer->serialize($snapshot)
    );
  }

  // 指定したIDのスナップショットがあるかどうかをチェックする
  public function has(string $id, int $version): bool
  {
    $snapshot = $this->byId($id);

    if (null === $snapshot) {
      return false;
    }

    return $snapshot->version() === $version;
  }
}


And now we need to refactor the EventStore class so that it starts using the SnapshotRepository to load the Aggregate with acceptable performance times:

次に、EventStoreクラスをリファクタリングして、SnapshotRepositoryの使用を開始し、許容可能なパフォーマンス時間でAggregateをロードする必要があります。


public function byId(PostId $id): Post
{
  $snapshot = $this->snapshotRepository->byId($id->id());

  if (null === $snapshot) {
    return Post::reconstitute(
    $this->eventStore->getEventsFrom($id->id())
    );
  }

  $post = $snapshot->aggregate();

  $post->replay(
  $this->eventStore->fromVersion($id->id(), $snapshot->version())
  );

  return $post;
}
```

We just need to take Aggregate snapshots periodically.
We could do this synchronously or asynchronously by a process responsible for monitoring the event store.
The following code is a simple example demonstrating the implementation of Aggregate snapshotting:

定期的に集約スナップショットを作成する必要があります。
これは、イベントストアの監視を担当するプロセスによって同期的または非同期的に実行できます。
次のコードは、集約スナップショットの実装を示す簡単な例です。

```php
public function save(Post $post): void
{
  $id = $post->id();

  $events = $post->recordedEvents();
  $post->clearEvents();

  $this->eventStore->append(
    new EventStream($id->id(), $events)
  );

  $countOfEvents = $this->eventStore->countEventsFor(
    $id->id()
  );

  // 100回ごとに保存処理を実施
  $version = $countOfEvents / 100;

  if (!$this->snapshotRepository->has($id->id(), $version)) {
    $this->snapshotRepository->save(
      $id->id(),
      new Snapshot(
        $post,
        $version
      )
    );
  }

  // 読み取りオブジェクトに投影
  $this->projector->project($events);
}
```

=========▼▼（ここから）▼▼=========  
### To ORM or Not To ORM (II)

It's clear from the use case of this architectural style that using an ORM just to persist / fetch events would be overkill.
Even if we use a relational database for storing them, we only need to persist / fetch events from the data store.

このアーキテクチャスタイルのユースケースから、イベントを永続化/フェッチするためだけにORMを使用するのはやり過ぎであることは明らかです。
それらを格納するためにリレーショナルデータベースを使用する場合でも、データストアからイベントを永続化/フェッチするだけで済みます。
=========▲▲（ここまで）▲▲=========


## 2.6 Wrap-Up

As there are plenty of options for architectural styles, you may have gotten a bit confused in this chapter.
You'll have to consider the trade-offs for each one of them in order to choose wisely.

アーキテクチャのスタイルにはたくさんのオプションがあるので、この章では少し混乱しているかもしれません。
賢明に選択するには、それぞれのトレードオフを考慮する必要があります。

One thing is clear: the Big Ball of Mud approach is not an option, as the code will rot pretty fast.

一つ確かなことがあります。
コードがかなり速く腐敗するため、大きな泥だんごアプローチはオプションではありません。

Layered Architecture is a better option, but it presents some disadvantages, like tight coupling between layers.
Arguably, the most balanced option would be Hexagonal Architecture, as it can be used as a foundational base architecture, and it promotes a high-level degree of decoupling and symmetry between the inside and outside of the application.
This is what we recommend for most scenarios.

階層化アーキテクチャの方が優れたオプションですが、レイヤー間の密結合など、いくつかの欠点があります。
おそらく、最もバランスの取れたオプションは、基本的な基本アーキテクチャとして使用できるヘキサゴナルアーキテクチャであり、アプリケーションの内部と外部の間の高度なデカップリングと対称性を促進します。
これは、ほとんどのシナリオで推奨されるものです。


We've also seen CQRS and Event Sourcing as relatively flexible architectures that will help you in fighting serious complexity.
また、CQRSとイベントソーシングは、深刻な複雑さとの戦いに役立つ比較的柔軟なアーキテクチャと見なされています。

CQRS and Event Sourcing both have their places, 
  but don't let the “coolness factor” distract you from the value they provide.
CQRSとイベントソーシングはどちらにも利用するのに適した場所がありますが、
「クールネスファクター」が提供する価値から気をそらさないようにしてください。

As they both come with some overhead, you should have a technical reason for justifying their use.
どちらにもオーバーヘッドが伴うため、使用を正当化する技術的な理由が必要です。

These architectural styles are indeed really useful, and the heuristics to start using them can be discovered in the number of finders on the Repositories for CQRS and the volume of triggered events for Event Sourcing.
これらのアーキテクチャスタイルは確かに非常に便利であり、それらの使用を開始するためのヒューリスティックは、CQRSのリポジトリのファインダーの数とイベントソーシングのトリガーされたイベントの量で見つけることができます。

If the number of finder methods starts growing and Repositories become difficult to maintain, then it's time to consider the use of CQRS, in order to split read and write concerns.
ファインダーメソッドの数が増え始め、リポジトリの保守が困難になった場合は、読み取りと書き込みの問題を分割するために、CQRSの使用を検討するときが来ました。

And after that, if the volume of events on each Aggregate operation tends to grow and the business is interested in more granular information, then an option to consider is whether a move toward Event Sourcing might pay off.

その後、各集約操作でのイベントの量が増加する傾向があり、ビジネスがより詳細な情報に関心を持っている場合、検討するオプションは、イベントソーシングへの移行が報われるかどうかです。
