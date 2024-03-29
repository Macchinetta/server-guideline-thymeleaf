チュートリアル(Todoアプリケーション)
********************************************************************************

.. only:: html

.. contents:: 目次
  :depth: 3
  :local:

|

はじめに
================================================================================

このチュートリアルで学ぶこと
--------------------------------------------------------------------------------

* Macchinetta Server Framework (1.x)による基本的なアプリケーションの開発方法
* MavenおよびSTS(Eclipse)プロジェクトの構築方法
* Macchinetta Server Framework (1.x)の\ :doc:`../Overview/ApplicationLayering`\ に従った開発方法

|

対象読者
--------------------------------------------------------------------------------

* SpringのDIやAOPに関する基礎的な知識がある
* Servlet/テンプレートエンジン(JSPなど)を使用してWebアプリケーションを開発したことがある
* SQLに関する知識がある

|

検証環境
--------------------------------------------------------------------------------

このチュートリアルは以下の環境で動作確認している。他の環境で実施する際は本書をベースに適宜読み替えて設定していくこと。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 30 70

  * - 種別
    - 名前
  * - OS
    - Windows 10
  * - JVM
    - \ `Java <https://developers.redhat.com/products/openjdk/download>`_\  17
  * - IDE
    - \ `Spring Tool Suite <https://spring.io/tools>`_\  4.17.1.RELEASE (以降「STS」と呼ぶ。設定方法は\ :doc:`../Appendix/SpringToolSuite4`\ を参照されたい。)
  * - Build Tool
    - \ `Apache Maven <https://maven.apache.org/download.cgi>`_\  3.8.6 (以降「Maven」と呼ぶ)
  * - Application Server
    - \ `Apache Tomcat <https://tomcat.apache.org/tomcat-10.1-doc/index.html>`_\  10.1.7
  * - Web Browser
    - \ `Google Chrome <https://www.google.co.jp/chrome/>`_\  108

|

作成するアプリケーションの説明
================================================================================

本チュートリアルでは、ViewとしてThymeleafを使用して開発するメリットを体感できるよう、
最初にHTMLで画面デザインのみ実装したモックアップ（以降、プロトタイプと呼ぶ）を作成し、そこにアプリケーションの機能を追加していく。
なお本ガイドラインでは、HTMLで作成したプロトタイプにThymeleafの属性を付与してテンプレート化したものを、「テンプレートHTML」と呼ぶ。

.. _tutorial-todo-application-overview-label:

アプリケーションの概要
--------------------------------------------------------------------------------

TODOを管理するアプリケーションを作成する。TODOの一覧表示、TODOの登録、TODOの完了、TODOの削除を行える。

.. figure:: ./images_TutorialTodo/image001.png
  :width: 50%

|

.. _app-requirement:

アプリケーションの業務要件
--------------------------------------------------------------------------------
アプリケーションの業務要件は、以下の通りとする。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 20 80

  * - ルールID
    - 説明
  * - B01
    - 未完了のTODOは5件までしか登録できない
  * - B02
    - 完了済みのTODOは完了できない

.. note::

   本要件は学習のためのもので、現実的なTODO管理アプリケーションとしては適切ではない。

|

アプリケーションの処理仕様
--------------------------------------------------------------------------------
アプリケーションの処理仕様と画面遷移は、以下の通りとする。

.. figure:: ./images_TutorialTodo/image002.png
  :width: 60%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 20 10 15 45

  * - 項番
    - プロセス名
    - HTTPメソッド
    - URL
    - 備考
  * - 1
    - Show all TODO
    - \-
    - /todo/list
    -
  * - 2
    - Create TODO
    - POST
    - /todo/create
    - 作成処理終了後、Show all TODOへリダイレクト
  * - 3
    - Finish TODO
    - POST
    - /todo/finish
    - 完了処理終了後、Show all TODOへリダイレクト
  * - 4
    - Delete TODO
    - POST
    - /todo/delete
    - 削除処理終了後、Show all TODOへリダイレクト

|

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* TODOを全件表示する
* 未完了のTODOに対しては「Finish」と「Delete」用のボタンが付く
* 完了のTODOは打ち消し線で装飾する
* TODOの件名のみ表示する

|

Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたTODOを保存する
* TODOの件名は1文字以上30文字以下であること
* :ref:`app-requirement`\ のB01を満たさない場合はエラーコードE001でビジネス例外をスローする
* 処理が成功した場合は、遷移先の画面で「Created successfully!」を表示する

|

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信された\ ``todoId``\ に対応するTODOを完了済みにする
* 該当するTODOが存在しない場合はエラーコードE404でリソース未検出例外をスローする
* \ :ref:`app-requirement`\ のB02を満たさない場合はエラーコードE002でビジネス例外をスローする
* 処理が成功した場合は、遷移先の画面で「Finished successfully!」を表示する

|

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信された\ ``todoId``\ に対応するTODOを削除する
* 該当するTODOが存在しない場合はエラーコードE404でリソース未検出例外をスローする
* 処理が成功した場合は、遷移先の画面で「Deleted successfully!」を表示する

|

エラーメッセージ一覧
--------------------------------------------------------------------------------

エラーメッセージとして、以下の3つを定義する。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.50\linewidth}|p{0.35\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 15 50 35

  * - エラーコード
    - メッセージ
    - 置換パラメータ
  * - E001
    - [E001] The count of un-finished Todo must not be over {0}.
    - {0}… max unfinished count
  * - E002
    - [E002] The requested Todo is already finished. (id={0})
    - {0}… todoId
  * - E404
    - [E404] The requested Todo is not found. (id={0})
    - {0}… todoId

|

環境構築
================================================================================

本チュートリアルでは、インフラストラクチャ層のRepositoryImplの実装として、

* データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImpl
* MyBatis3を使用してデータベースにアクセスするRepositoryImpl

の2種類を用意している。用途に応じていずれかを選択する。

チュートリアルの進行上、まずはインメモリ実装を試し、その後MyBatis3を選ぶのが円滑である。

|

プロジェクトの作成
--------------------------------------------------------------------------------

まず、\ ``mvn archetype:generate``\ を利用して、実装するインフラストラクチャ層向けのブランクプロジェクトを作成する。
ここでは、Windowsのコマンドプロンプトを使用してブランクプロジェクトを作成する手順となっている。

.. note::

  インターネット接続するために、プロキシサーバーを介する必要がある場合、以下の作業を行うため、STSのProxy設定と、\ `MavenのProxy設定 <https://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。

.. tip::

  Bash上で\ ``mvn archetype:generate``\ を実行する場合は、以下のように"\ ``^``\ " を"\ ``\``\ " に置き換えて実行すればよい。

    .. code-block:: bash

      mvn archetype:generate -B\
       -DarchetypeGroupId=com.github.macchinetta.blank\
       -DarchetypeArtifactId=macchinetta-web-blank-noorm-thymeleaf-archetype\
       -DarchetypeVersion=1.9.1.RELEASE\
       -DgroupId=com.example.todo\
       -DartifactId=todo\
       -Dversion=1.0.0-SNAPSHOT

|

.. _TutorialCreatePlainBlankProject:

O/R Mapperに依存しないブランクプロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImpl用のプロジェクトを作成する場合は、以下のコマンドを実行してO/R Mapperに依存しないブランクプロジェクトを作成する。\ **本チュートリアルを順序通り読み進める場合は、まずはこの方法でプロジェクトを作成すること**\ 。

.. code-block:: console

  mvn archetype:generate -B^
   -DarchetypeGroupId=com.github.macchinetta.blank^
   -DarchetypeArtifactId=macchinetta-web-blank-noorm-thymeleaf-archetype^
   -DarchetypeVersion=1.9.1.RELEASE^
   -DgroupId=com.example.todo^
   -DartifactId=todo^
   -Dversion=1.0.0-SNAPSHOT

.. _TutorialCreateMyBatis3BlankProject:

MyBatis3用のブランクプロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3を使用してデータベースにアクセスするRepositoryImpl用のプロジェクトを作成する場合は、以下のコマンドを実行してMyBatis3用のブランクプロジェクトを作成する。このプロジェクト作成方法は\ :ref:`using_MyBatis3`\ で使用する。

.. code-block:: console

  mvn archetype:generate -B^
   -DarchetypeGroupId=com.github.macchinetta.blank^
   -DarchetypeArtifactId=macchinetta-web-blank-thymeleaf-archetype^
   -DarchetypeVersion=1.9.1.RELEASE^
   -DgroupId=com.example.todo^
   -DartifactId=todo^
   -Dversion=1.0.0-SNAPSHOT

|

プロジェクトのインポート
--------------------------------------------------------------------------------

作成したブランクプロジェクトをSTSへインポートする。

STSのメニューから、[File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next]を選択し、archetypeで作成したプロジェクトを選択する。

.. figure:: images_TutorialTodo/NewMVCProjectImport.png
  :alt: New MVC Project Import
  :width: 60%

|

Root Directoryに\ ``C:\work\todo``\ を設定し、Projectsにtodoのpom.xmlが選択された状態で、 [Finish] を押下する。

.. figure:: images_TutorialTodo/NewMVCProjectCreate.png
  :alt: New MVC Project Import
  :width: 60%

|

インポートが完了すると、Package Explorerに次のようなプロジェクトが表示される。

.. figure:: images_TutorialTodo/image004.png
  :alt: workspace

.. note::

  インポート後にビルドエラーが発生する場合は、プロジェクト名を右クリックし、「Maven」->「Update Project...」をクリックし、「OK」ボタンをクリックすることでエラーが解消されるケースがある。

  .. figure:: images_TutorialTodo/update-project.png
    :width: 70%

.. tip::

  パッケージの表示形式は、デフォルトは「Flat」だが、「Hierarchical」にしたほうが見通しがよい。

  Package Explorerの「View Menu」 (右端の下矢印)をクリックし、「Package Presentation」->「Hierarchical」を選択する。

  .. figure:: ./images_TutorialTodo/presentation-hierarchical.png
    :width: 80%

  Package PresentationをHierarchicalにすると、以下の様な表示になる。

  .. figure:: ./images_TutorialTodo/presentation-hierarchical-view.png

.. warning::

  O/R Mapperを使用するブランクプロジェクトの場合、H2 Databaseがdependencyとして定義されているが、この設定は簡易的なアプリケーションを簡単に作成するためのものであり、実際のアプリケーション開発で使用されることは想定していない。

  以下の定義は、実際のアプリケーション開発を行う際は削除すること。

    .. code-block:: xml

      <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
          <scope>runtime</scope>
      </dependency>

.. note::

  上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。

  上記の依存ライブラリはterasoluna-gfw-parentが依存している\ `Spring Boot <https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#dependency-versions>`_\ で管理されている。

|

プロジェクトの構成
--------------------------------------------------------------------------------

本チュートリアルで作成するプロジェクトの構成を以下に示す。

.. note::

  \ :ref:`前節の「プロジェクト構成」 <application-layering_project-structure>`\ ではマルチプロジェクトにすることを推奨していたが、本チュートリアルでは、学習容易性を重視しているためシングルプロジェクト構成にしている。

  \ **ただし、実プロジェクトで適用する場合は、マルチプロジェクト構成を強く推奨する。**\

  マルチプロジェクトの作成方法は、「\ :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`\ 」を参照されたい。

|

\ **[O/R Mapperに依存しないブランクプロジェクトを作成した場合の構成]**\

.. code-block:: console

  src
    └main
        ├java
        │  └com
        │    └example
        │      └todo
        │        ├ app ... (1)
        │        │   └todo
        │        └domain ... (2)
        │            ├model ... (3)
        │            ├repository ... (4)
        │            │   └todo
        │            └service ... (5)
        │                └todo
        ├resources
        │  └META-INF
        │      └spring ... (6)
        └webapp
            ├resources
            │  └app
            │    └css ... (7)
            └WEB-INF
                └views ... (8)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - アプリケーション層のクラスを格納するパッケージ。

      本チュートリアルでは、Todo管理業務用のクラスを格納するためのパッケージを作成する。
  * - | (2)
    - ドメイン層のクラスを格納するパッケージ。
  * - | (3)
    - Domain Objectを格納するパッケージ。
  * - | (4)
    - Repositoryを格納するパッケージ。

      本チュートリアルでは、Todoオブジェクト(Domain Object)用のRepositoryを格納するためのパッケージを作成する
  * - | (5)
    - Serviceを格納するパッケージ。

      本チュートリアルでは、Todo管理業務用のServiceを格納するためのパッケージを作成する。
  * - | (6)
    - Spring関連の設定ファイルを格納するディレクトリ。
  * - | (7)
    - cssファイルを格納するディレクトリ。
  * - | (8)
    - ThymeleafのテンプレートHTMLを格納するディレクトリ。

|

\ **[MyBatis3用のブランクプロジェクトを作成した場合の構成]**\

.. code-block:: console

  src
    └main
        ├java
        │  └com
        │    └example
        │      └todo
        │        ├ app
        │        │   └todo
        │        └domain
        │            ├model
        │            ├repository
        │            │   └todo
        │            └service
        │                └todo
        ├resources
        │  ├META-INF
        │  │  ├mybatis ... (9)
        │  │  └spring
        │  └com
        │    └example
        │      └todo
        │        └domain
        │            └repository ... (10)
        │                 └todo
        └webapp
            ├resources
            │  └app
            │    └css
            └WEB-INF
                └views

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (9)
    - MyBatis関連の設定ファイルを格納するディレクトリ。
  * - | (10)
    - SQLを記述するMyBatisのMapperファイルを格納するディレクトリ。

      本チュートリアルでは、Todoオブジェクト用のRepositoryのMapperファイルを格納するためのディレクトリを作成する。

|

設定ファイルの確認
--------------------------------------------------------------------------------
チュートリアルを進める上で必要となる設定の多くは、作成したブランクプロジェクトに既に設定済みの状態である。

チュートリアルを実施するだけであれば、これらの設定の理解は必須ではないが、アプリケーションを動かすためにどのような設定が必要なのかを理解しておくことを推奨する。

アプリケーションを動かすために必要な設定(設定ファイル)の解説については、「\ :ref:`TutorialTodoAppendixExpoundConfigurations`\ 」を参照されたい。

.. note::

  まず、手を動かしてTodoアプリケーションを作成したい場合は、設定ファイルの確認は読み飛ばしてもよいが、Todoアプリケーションを作成した後に一読して頂きたい。

|

プロジェクトの動作確認
--------------------------------------------------------------------------------
Todoアプリケーションの開発を始める前に、プロジェクトの動作確認を行う。

ブランクプロジェクトでは、トップページを表示するためのControllerとテンプレートHTMLの実装が用意されているため、トップページを表示する事で動作確認を行う事ができる。

ブランクプロジェクトから提供されているController(\ :file:`src/main/java/com/example/todo/app/welcome/HelloController.java`\ )は、以下のような実装となっている。

.. code-block:: java
  :emphasize-lines: 16, 20, 27, 30, 39, 42

  package com.example.todo.app.welcome;

  import java.text.DateFormat;
  import java.util.Date;
  import java.util.Locale;

  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.GetMapping;

  /**
   * Handles requests for the application home page.
   */
  // (1)
  @Controller
  public class HelloController {

      // (2)
      private static final Logger logger = LoggerFactory
              .getLogger(HelloController.class);

      /**
       * Simply selects the home view to render by returning its name.
       */
      // (3)
      @GetMapping(value = "/")
      public String home(Locale locale, Model model) {
          // (4)
          logger.info("Welcome home! The client locale is {}.", locale);

          Date date = new Date();
          DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG,
                  DateFormat.LONG, locale);

          String formattedDate = dateFormat.format(date);

          // (5)
          model.addAttribute("serverTime", formattedDate);

          // (6)
          return "welcome/home";
      }

  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Controller``\ アノテーションが付与している。
  * - | (2)
    - | (4)でログ出力するためのロガーを生成している。
      | ロガーの実装はlogbackのものであるが、APIはSLF4Jの\ ``org.slf4j.Logger``\ を使用している。
  * - | (3)
    - | ``@GetMapping`` アノテーションを使用して、"\ ``/``\" (ルート)へのアクセスに対するメソッドとしてマッピングを行っている。
  * - | (4)
    - | メソッドが呼ばれたことを通知するためのログをinfoレベルで出力している。
  * - | (5)
    - | 画面に表示するための日付文字列を、\ ``serverTime``\ という属性名でModelに設定している。
  * - | (6)
    - | view名として\ ``welcome/home``\ を返す。\ ``ViewResolver``\ の設定によりテンプレートHTMLとして\ ``WEB-INF/views/welcome/home.html``\ を利用して生成したHTMLが返される。

|

ブランクプロジェクトから提供されているテンプレートHTML(\ :file:`src/main/webapp/WEB-INF/views/welcome/home.html`\ )は、以下のような実装となっている。

.. code-block:: html
  :emphasize-lines: 12

  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta charset="utf-8">
  <title>Home</title>
  <link rel="stylesheet"
      href="../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
  </head>
  <body>
      <div id="wrapper">
          <h1 id="title">Hello world!</h1>
          <!-- (7) -->
          <p th:text="|The time on the server is ${serverTime}.|">The time on the server is 2018/01/01 00:00:00 JST.</p>
      </div>
  </body>
  </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90


  * - 項番
    - 説明
  * - | (7)
    - | ControllerでModelに設定した\ ``serverTime``\ を表示する。
      | \ ``th:text``\属性は、記述した要素のコンテンツを属性値で上書きする。
      | \ ``th:text``\属性に、変数式\ ``${}``\で変数名を指定することで、ControllerでModelに登録した変数を参照できる。
      | ユーザの入力値を表示する場合は、\ ``th:text``\ 属性を用いて、必ずXSS対策を行うこと。

|

プロジェクトを右クリックして「Run As」->「Run on Server」を選択する。

.. figure:: ./images_TutorialTodo/image031.jpg
  :width: 70%

|

APサーバー(Tomcat v10.1 Server at localhost)を選択し、「Next」をクリックする。

.. figure:: ./images_TutorialTodo/image032.jpg
  :width: 70%

|

todoが「Configured」に含まれていることを確認して「Finish」をクリックしてサーバーを起動する。

.. figure:: ./images_TutorialTodo/image033.jpg
  :width: 70%

|

起動すると以下のようなログが出力される。
"\ ``/``\ " というパスに対して\ ``com.example.todo.app.welcome.HelloController``\ のhomeメソッドがマッピングされていることが分かる。

.. code-block:: console
  :emphasize-lines: 3-4

  date:2022-12-01 11:41:47	thread:main	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:Initializing Servlet 'appServlet'
  date:2022-12-01 11:41:47	thread:main	X-Track:	level:TRACE	logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping      	message:
      c.e.t.a.w.HelloController:
      {GET [/]}: home(Locale,Model)
  date:2022-12-01 11:41:47	thread:main	X-Track:	level:DEBUG	logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping      	message:1 mappings in 'requestMappingHandlerMapping'
  date:2022-12-01 11:41:48	thread:main	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:Completed initialization in 915 ms

|

ブラウザで http://localhost:8080/todo にアクセスすると、以下のように表示される。

.. figure:: ./images_TutorialTodo/image034.png
  :width: 60%


コンソールを見ると、

* 共通ライブラリから提供している\ ``TraceLoggingInterceptor``\ のTRACEログ
* Controllerで実装したINFOログ

が出力されていることがわかる。

.. code-block:: console
  :emphasize-lines: 1-4

  date:2022-12-01 11:48:22	thread:http-nio-8080-exec-3	X-Track:d41df6b34f7d4002b7d1cf415e5ada95	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] HelloController.home(Locale,Model)
  date:2022-12-01 11:48:22	thread:http-nio-8080-exec-3	X-Track:d41df6b34f7d4002b7d1cf415e5ada95	level:INFO 	logger:com.example.todo.app.welcome.HelloController    	message:Welcome home! The client locale is ja.
  date:2022-12-01 11:48:22	thread:http-nio-8080-exec-3	X-Track:d41df6b34f7d4002b7d1cf415e5ada95	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] HelloController.home(Locale,Model)-> view=welcome/home, model={serverTime=2022年12月1日 11:48:22 JST}
  date:2022-12-01 11:48:22	thread:http-nio-8080-exec-3	X-Track:d41df6b34f7d4002b7d1cf415e5ada95	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] HelloController.home(Locale,Model)-> 38,729,900 ns

.. note::
 
  \ ``TraceLoggingInterceptor``\ はControllerの開始、終了でログを出力する。終了時には\ ``View``\ と\ ``Model``\ の情報および処理時間が出力される。

|


.. _create-prototype-of-tutorial-todo-label:

Todoアプリケーションのプロトタイプ作成
================================================================================

HTMLでTodoアプリケーションのプロトタイプを作成する。

本チュートリアルでは、ここで作成したプロトタイプにThymeleafの属性を付与して、Todoアプリケーションの画面を実装していく。

プロトタイプ作成
--------------------------------------------------------------------------------

 :ref:`tutorial-todo-application-overview-label` で示した画面をプロトタイプとして作成する。

.. figure:: ./images_TutorialTodo/image001.png
    :width: 40%


.. note:: **実際のアプリケーション開発で作成するプロトタイプ**
   
   実際のアプリケーション開発では、ユースケースごとに画面の状態が確認できるプロトタイプ（本チュートリアルの例では、「TODOを作成した状態」や「TODOを完了した状態」など）を作成するのが一般的だと思われるが、
   今回はThymeleafを使用したアプリケーションの作成を学ぶチュートリアルで、プロトタイプの正しい作り方を解説することは主眼ではないため、省略する。
   
   また、プロトタイプをブランクプロジェクトベースで作成するかは開発プロジェクトの判断に任せるが、本チュートリアルでは、プロトタイプからアプリケーションを開発する工程を理解しやすいように、ブランクプロジェクトベースでプロトタイプを作成している。

Package Explorer上で右クリック -> New -> File を選択し、「Create New File」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Enter or select the parent folder
      - ``todo/src/main/webapp/WEB-INF/views/todo``
    * - 2
      - File name
      - ``list.html``

を入力して「Finish」する。

作成したファイルは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/create-list-jsp.png

 :ref:`tutorial-todo-application-overview-label` で示した画面をHTMLとして表示するために必要なプロトタイプの実装を行う。

.. code-block:: html
    :emphasize-lines: 19, 29, 48

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .inline {
        display: inline-block;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (1) -->
            <form action="/todo/create" method="post">
                <input type="text">
                <button>Create Todo</button>
            </form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <li>
                    <!-- (2) -->
                    <span>Send a e-mail</span>
                    <form action="/todo/finish" method="post" class="inline">
                        <button>Finish</button>
                    </form>
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
                <li>
                    <span>Have a lunch</span>
                    <form action="/todo/finish" method="post" class="inline">
                        <button>Finish</button>
                    </form>
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
                <li>
                    <span class="strike">Read a book</span><!-- (3) -->
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 新規作成処理用のformを表示する。
       | \ ``action``\ 属性には新規作成処理を実行するためのパス(\ ``/todo/create``\ )を指定する。
       | 新規作成処理は更新系の処理なので、\ ``method``\属性には\ ``POST``\ メソッドを指定する。
   * - | (2)
     - | 未完了のTODOに対しては「Finish」と「Delete」用のボタンを表示する。
       | \ ``action``\ 属性には更新処理、削除処理を実行するためのパス(\ ``/todo/finish``\ or \ ``/todo/delete``\ )を指定する。
       | 更新処理、削除処理は更新系の処理なので、\ ``method``\属性には\ ``POST``\ メソッドを指定する。
       | なお、「Finish」と「Delete」用のボタンをインラインブロック要素（\ ``display: inline-block;``\）としてTODOの横に表示させている。
   * - | (3)
     - | 完了しているTODOには、打ち消し線(\ ``text-decoration: line-through;``\ )を装飾する。
       | 完了しているTODOに対しては「Delete」用のボタンのみを表示する。

|

画面の静的表示の確認
--------------------------------------------------------------------------------

作成したプロトタイプのデザインをWebブラウザで確認すると、以下のように表示される。（以降、プロトタイプやテンプレートHTMLをブラウザで直接開く事を静的表示と呼ぶ。）

.. figure:: ./images_TutorialTodo/image001.png
    :width: 40%

|

CSSファイルの使用
--------------------------------------------------------------------------------

上記例ではスタイルシートをHTMLファイルの中で直接定義していたが、
実際のアプリケーションを開発する場合は、CSSファイルに定義するのが一般的である。

ここでは、スタイルシートをCSSファイルに定義する方法について説明する。

ブランクプロジェクトから提供しているCSSファイル(\ ``src/main/webapp/resources/app/css/styles.css``\ )にスタイルシートの定義を追加する。  
なお、ここでは、以降で使用するスタイルシートも含めて、CSSファイルに定義している。

.. code-block:: css

    /* ... */

    .strike {
        text-decoration: line-through;
    }

    .inline {
        display: inline-block;
    }

    .alert {
        border: 1px solid;
        margin-bottom: 5px;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }

    .alert ul {
        margin: 15px 0px 15px 0px;
    }

    #todoList li {
        margin-top: 5px;
    }

|

プロトタイプからCSSファイルを読み込む。

.. code-block:: html
    :emphasize-lines: 6-7

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <!-- (1) -->
    <link rel="stylesheet" href="../../../resources/app/css/styles.css">
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <form action="/todo/create" method="post">
                <input type="text">
                <button>Create Todo</button>
            </form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <li>
                    <span>Send a e-mail</span>
                    <form action="/todo/finish" method="post" class="inline">
                        <button>Finish</button>
                    </form>
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
                <li>
                    <span>Have a lunch</span>
                    <form action="/todo/finish" method="post" class="inline">
                        <button>Finish</button>
                    </form>
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
                <li>
                    <span class="strike">Read a book</span>
                    <form action="/todo/delete" method="post" class="inline">
                        <button>Delete</button>
                    </form>
                </li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | HTMLからスタイルシートの定義を削除し、代わりにスタイルシートを定義したCSSファイルを読み込む。

|

CSSファイルを適用すると、以下のようなレイアウトになる。

.. figure:: ./images_TutorialTodo/list-screen-css.png
    :width: 40%

|

Todoアプリケーションの作成
================================================================================
| プロトタイプからTodoアプリケーションを作成する。作成する順は、以下の通りである。

* ドメイン層(+ インフラストラクチャ層)

  * Domain Object作成
  * Repository作成
  * RepositoryImpl作成
  * Service作成

* アプリケーション層

  * Controller作成
  * Form作成
  * View作成

|

RepositoryImplの作成は、選択したインフラストラクチャ層の種類に応じて実装方法が異なる。

| ここでは、データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImplを作成する方法について説明を行う。
| データベースを使用する場合は、「\ :ref:`tutorial-todo_infra`\ 」に記載されている内容で読み替えて、Todoアプリケーションを作成して頂きたい。

|

ドメイン層の作成
--------------------------------------------------------------------------------

Domain Objectの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domainオブジェクトを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.domain.model``\
    * - 2
      - Name
      - \ ``Todo``\
    * - 3
      - Interfaces
      - \ ``java.io.Serializable``\

を入力して「Finish」する。

.. figure:: ./images_TutorialTodo/image057.png
  :width: 70%

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image058.png

|

作成したクラスに以下のプロパティを追加する。

* ID → todoId
* タイトル → todoTitle
* 完了フラグ → finished
* 作成日 → createdAt

.. code-block:: java

  package com.example.todo.domain.model;

  import java.io.Serializable;
  import java.util.Date;

  public class Todo implements Serializable {

      private static final long serialVersionUID = 1L;

      private String todoId;

      private String todoTitle;

      private boolean finished;

      private Date createdAt;

      public String getTodoId() {
          return todoId;
      }

      public void setTodoId(String todoId) {
          this.todoId = todoId;
      }

      public String getTodoTitle() {
          return todoTitle;
      }

      public void setTodoTitle(String todoTitle) {
          this.todoTitle = todoTitle;
      }

      public boolean isFinished() {
          return finished;
      }

      public void setFinished(boolean finished) {
          this.finished = finished;
      }

      public Date getCreatedAt() {
          return createdAt;
      }

      public void setCreatedAt(Date createdAt) {
          this.createdAt = createdAt;
      }
  }

.. tip::

  Getter/SetterメソッドはSTSの機能を使って自動生成することができる。
  フィールドを定義した後、エディタ上で右クリックし、「Source」->「Generate Getter and Setters…」を選択する。

  .. figure:: ./images_TutorialTodo/image059.png
    :width: 90%

  serialVersionUID以外を選択して「OK」

  .. figure:: ./images_TutorialTodo/image060.png
    :width: 60%

|

.. _TutorialTodoCreateRepository:

Repositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ ``TodoRepository``\ インタフェースを作成する。
| データベースを使用する場合は、「\ :ref:`tutorial-todo_infra`\ 」に記載されている内容で読み替えて、Repositoryを作成する。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.domain.repository.todo``\
    * - 2
      - Name
      - \ ``TodoRepository``\

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image061.png

作成したインタフェースに、今回のアプリケーションで必要となる以下のCRUD操作を行うメソッドを定義する。

* TODOの1件取得 → findById
* TODOの全件取得 → findAll
* TODOの1件作成 → create
* TODOの1件更新 → update
* TODOの1件削除 → delete
* 完了済みTODO件数の取得 → countByFinished

.. code-block:: java

  package com.example.todo.domain.repository.todo;

  import java.util.Collection;

  import com.example.todo.domain.model.Todo;

  public interface TodoRepository {
      Todo findById(String todoId);

      Collection<Todo> findAll();

      void create(Todo todo);

      boolean update(Todo todo);

      void delete(Todo todo);

      long countByFinished(boolean finished);
  }

.. note::

  ここでは、\ ``TodoRepository``\ の汎用性を上げるため、「完了済み件数を取得する」メソッド(\ ``long countFinished()``\ )ではなく、「完了状態がxxである件数を取得する」メソッド(\ ``long countByFinished(boolean)``\ )として定義している。

  \ ``long countByFinished(boolean)``\ の引数として\ ``true``\ を渡すと「完了済みの件数」、\ ``false``\ を渡すと「未完了の件数」が取得できる仕様としている。

|

RepositoryImplの作成(インフラストラクチャ層)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ここでは、説明を単純化するため、\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImplを作成する。
| データベースを使用する場合は、「\ :ref:`tutorial-todo_infra`\ 」に記載されている内容で読み替えて、RepositoryImplを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.domain.repository.todo``\
    * - 2
      - Name
      - \ ``TodoRepositoryImpl``\
    * - 3
      - Interfaces
      - \ ``com.example.todo.domain.repository.todo.TodoRepository``\

を入力して「Finish」する。

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image062.png

作成したクラスにCRUD操作を実装する。

.. note::

  RepositoryImplには、業務ロジックは含めず、Domainオブジェクトの保存先への出し入れ(CRUD操作)に終始することが実装ポイントである。

.. code-block:: java
  :emphasize-lines: 11

  package com.example.todo.domain.repository.todo;

  import java.util.Collection;
  import java.util.Map;
  import java.util.concurrent.ConcurrentHashMap;

  import org.springframework.stereotype.Repository;

  import com.example.todo.domain.model.Todo;

  @Repository // (1)
  public class TodoRepositoryImpl implements TodoRepository {
      private static final Map<String, Todo> TODO_MAP = new ConcurrentHashMap<String, Todo>();

      @Override
      public Todo findById(String todoId) {
          return TODO_MAP.get(todoId);
      }

      @Override
      public Collection<Todo> findAll() {
          return TODO_MAP.values();
      }

      @Override
      public void create(Todo todo) {
          TODO_MAP.put(todo.getTodoId(), todo);
      }

      @Override
      public boolean update(Todo todo) {
          TODO_MAP.put(todo.getTodoId(), todo);
          return true;
      }

      @Override
      public void delete(Todo todo) {
          TODO_MAP.remove(todo.getTodoId());
      }

      @Override
      public long countByFinished(boolean finished) {
          long count = 0;
          for (Todo todo : TODO_MAP.values()) {
              if (finished == todo.isFinished()) {
                  count++;
              }
          }
          return count;
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 80

  * - 項番
    - 説明
  * - | (1)
    - | Repositoryとしてcomponent-scan対象とするため、クラスレベルに\ ``@Repository``\ アノテーションをつける。

.. note::

  本チュートリアルでは、インフラストラクチャ層に属するクラス(RepositoryImpl)をドメイン層のパッケージ(\ ``com.example.todo.domain``\)に格納しているが、完全に層別にパッケージを分けるのであれば、インフラストラクチャ層のクラスは、\ ``com.example.todo.infra``\ 以下に作成した方が良い。

  ただし、通常のプロジェクトでは、インフラストラクチャ層が変更されることを前提としていない(そのような前提で進めるプロジェクトは、少ない)。

  そこで、作業効率向上のために、ドメイン層のRepositoryインタフェースと同じ階層に、RepositoryImplを作成しても良い。

|

Serviceの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まず、\ ``TodoService``\ インタフェースを作成する。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.domain.service.todo``\
    * - 2
      - Name
      - \ ``TodoService``\

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image063.png

作成したインタフェースに以下の業務処理を行うメソッドを定義する。

* Todoの全件取得 → findAll
* Todoの新規作成 → create
* Todoの完了 → finish
* Todoの削除 → delete

.. code-block:: java

  package com.example.todo.domain.service.todo;

  import java.util.Collection;

  import com.example.todo.domain.model.Todo;

  public interface TodoService {
      Collection<Todo> findAll();

      Todo create(Todo todo);

      Todo finish(String todoId);

      void delete(String todoId);
  }

|

次に、\ ``TodoService``\ インタフェースに定義したメソッドを実装する\ ``TodoServiceImpl``\ クラスを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.domain.service.todo``\
    * - 2
      - Name
      - \ ``TodoServiceImpl``\
    * - 3
      - Interfaces
      - \ ``com.example.todo.domain.service.todo.TodoService``\

を入力して「Finish」する。

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image064.png

.. code-block:: java
  :emphasize-lines: 19, 20, 25-26, 29, 38-39, 43-44, 47-48, 81-82, 89-90

  package com.example.todo.domain.service.todo;

  import java.util.Collection;
  import java.util.Date;
  import java.util.UUID;

  import jakarta.inject.Inject;

  import org.springframework.stereotype.Service;
  import org.springframework.transaction.annotation.Transactional;
  import org.terasoluna.gfw.common.exception.BusinessException;
  import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
  import org.terasoluna.gfw.common.message.ResultMessage;
  import org.terasoluna.gfw.common.message.ResultMessages;

  import com.example.todo.domain.model.Todo;
  import com.example.todo.domain.repository.todo.TodoRepository;

  @Service// (1)
  @Transactional // (2)
  public class TodoServiceImpl implements TodoService {

      private static final long MAX_UNFINISHED_COUNT = 5;

      @Inject// (3)
      TodoRepository todoRepository;

      @Override
      @Transactional(readOnly = true) // (4)
      public Collection<Todo> findAll() {
          return todoRepository.findAll();
      }

      @Override
      public Todo create(Todo todo) {
          long unfinishedCount = todoRepository.countByFinished(false);
          if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
              // (5)
              ResultMessages messages = ResultMessages.error();
              messages.add(ResultMessage
                      .fromText("[E001] The count of un-finished Todo must not be over "
                              + MAX_UNFINISHED_COUNT + "."));
              // (6)
              throw new BusinessException(messages);
          }

          // (7)
          String todoId = UUID.randomUUID().toString();
          Date createdAt = new Date();

          todo.setTodoId(todoId);
          todo.setCreatedAt(createdAt);
          todo.setFinished(false);

          todoRepository.create(todo);

          return todo;
      }

      @Override
      public Todo finish(String todoId) {
          Todo todo = findOne(todoId);
          if (todo.isFinished()) {
              ResultMessages messages = ResultMessages.error();
              messages.add(ResultMessage
                      .fromText("[E002] The requested Todo is already finished. (id="
                              + todoId + ")"));
              throw new BusinessException(messages);
          }
          todo.setFinished(true);
          todoRepository.update(todo);
          return todo;
      }

      @Override
      public void delete(String todoId) {
          Todo todo = findOne(todoId);
          todoRepository.delete(todo);
      }

      // (8)
      private Todo findOne(String todoId) {
          Todo todo = todoRepository.findById(todoId);
          if (todo == null) {
              ResultMessages messages = ResultMessages.error();
              messages.add(ResultMessage
                      .fromText("[E404] The requested Todo is not found. (id="
                              + todoId + ")"));
              // (9)
              throw new ResourceNotFoundException(messages);
          }
          return todo;
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
  * - | (2)
    - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
      | アノテーションを付与することで、メソッド開始時にトランザクションを開始、メソッド正常終了時にトランザクションのコミットが行われる。
      | また、途中で非検査例外が発生した場合は、トランザクションがロールバックされる。
      |
      | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
  * - | (3)
    - | \ ``@Inject``\ アノテーションで、\ ``TodoRepository``\ の実装をインジェクションする。
  * - | (4)
    - | 参照のみ行う処理に関しては、\ ``readOnly=true``\ をつける。
      | O/R Mapperによっては、この設定により、参照時のトランザクション制御の最適化が行われる。
      |
      | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
  * - | (5)
    - | 結果メッセージを格納するクラスとして、共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.message.ResultMessage``\ を用いる。
      | 今回は、エラーメッセージを例外に追加する際に、\ ``ResultMessages.error()``\ でメッセージ種別を指定して、\ ``ResultMessage``\ を追加している。
  * - | (6)
    - | 業務エラーが発生した場合、共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.exception.BusinessException``\ をスローする。
  * - | (7)
    - | 一意性のある値を生成するために、UUIDを使用している。データベースのシーケンスを用いてもよい。
  * - | (8)
    - | 1件取得は、\ ``finish``\ メソッドでも\ ``delete``\ メソッドでも使用するため、メソッドとして用意しておく(interfaceに公開しても良い)。
  * - | (9)
    - | 取得したデータを返す。対象のデータが存在しない場合は共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\ をスローする。

.. note::

  本節では、説明を単純化するため、エラーメッセージをハードコードしているが、メンテナンスの観点で本来は好ましくない。通常、メッセージは、プロパティファイルに外部化することが推奨される。

  プロパティファイルに外部化する方法は、\ :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`\ を参照されたい。

|

アプリケーション層の作成
--------------------------------------------------------------------------------

ドメイン層の実装が完了したので、次はドメイン層を利用して、アプリケーション層の作成に取り掛かる。
画面（テンプレートHTML）には、プロトタイプとして作成したHTMLファイルを使用する。

Controllerの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まずは、Todo管理業務にかかわる画面遷移を、制御するControllerを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.app.todo``\
    * - 2
      - Name
      - \ ``TodoController``\

を入力して「Finish」する。

.. note::

  \ **上位パッケージがドメイン層と異なるので注意すること。**\

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image065.png

.. code-block:: java
  :emphasize-lines: 6, 7

  package com.example.todo.app.todo;

  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;

  @Controller // (1)
  @RequestMapping("todo") // (2)
  public class TodoController {

  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに、\ ``@Controller``\ アノテーションをつける。
  * - | (2)
    - | \ ``TodoController``\ が扱う画面遷移のパスを、すべて\ ``<contextPath>/todo``\ 配下にするため、クラスレベルに\ ``@RequestMapping(“todo”)``\ を設定する。

|

Show all TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本チュートリアルで作成する画面では、

* 新規作成フォームの表示
* TODOの全件表示

を行う。

はじめに、TODOの全件表示を行うための処理を実装する。

|

Formの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Formクラス(JavaBean)を作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.app.todo``\
    * - 2
      - Name
      - \ ``TodoForm``\
    * - 3
      - Interfaces
      - \ ``java.io.Serializable``\

を入力して「Finish」する。

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/image066.png

作成したクラスに以下のプロパティを追加する。

* タイトル → todoTitle

.. code-block:: java

  package com.example.todo.app.todo;

  import java.io.Serializable;

  public class TodoForm implements Serializable {
      private static final long serialVersionUID = 1L;

      private String todoTitle;

      public String getTodoTitle() {
          return todoTitle;
      }

      public void setTodoTitle(String todoTitle) {
          this.todoTitle = todoTitle;
      }

  }

|

Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

一覧画面表示処理を\ ``TodoController``\ に追加する。

.. code-block:: java
  :emphasize-lines: 20-21, 23-24, 29, 32, 33

  package com.example.todo.app.todo;

  import java.util.Collection;

  import jakarta.inject.Inject;

  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.RequestMapping;

  import com.example.todo.domain.model.Todo;
  import com.example.todo.domain.service.todo.TodoService;

  @Controller
  @RequestMapping("todo")
  public class TodoController {

      @Inject // (1)
      TodoService todoService;

      @ModelAttribute // (2)
      public TodoForm setUpForm() {
          TodoForm form = new TodoForm();
          return form;
      }

      @GetMapping("list") // (3)
      public String list(Model model) {
          Collection<Todo> todos = todoService.findAll();
          model.addAttribute("todos", todos); // (4)
          return "todo/list"; // (5)
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``TodoService``\ を、DIコンテナによってインジェクションさせるために、\ ``@Inject``\ アノテーションをつける。
      |
      | DIコンテナの管理する\ ``TodoService``\ 型のインスタンス(\ ``TodoServiceImpl``\ のインスタンス)がインジェクションされる。
  * - | (2)
    - | Formを初期化する。
      |
      | \ ``@ModelAttribute``\ アノテーションをつけることで、このメソッドの返り値のformオブジェクトが、\ ``todoForm``\ という名前で\ ``Model``\ に追加される。
      | これは、\ ``TodoController``\ の各処理で、\ ``model.addAttribute("todoForm", form)``\ を実装するのと同義である。
  * - | (3)
    - | \ ``/todo/list``\ というパスに\ ``GET``\ メソッドを使用してリクエストされた際に、一覧画面表示処理用のメソッド(\ ``list``\ メソッド)が実行されるように\ ``@GetMapping``\ アノテーションを設定する。
      |
      | クラスレベルに\ ``@RequestMapping(“todo”)``\ が設定されているため、ここでは\ ``@GetMapping("list")``\ のみで良い。
  * - | (4)
    - | \ ``Model``\ にTodoのリストを追加して、Viewに渡す。
  * - | (5)
    - | View名として\ ``todo/list``\ を返すと、spring-mvc.xmlに定義した\ ``ViewResolver``\ の設定によりテンプレートHTMLとして\ :file:`WEB-INF/views/todo/list.html`\を利用して生成したHTMLが返される。

.. note::

  \ ``@GetMapping``\ や以降に登場する\ ``@PostMapping``\ は、対応するHTTPメソッドにマッピングする。

  詳細は、\ :ref:`controller_mapping-label`\ を参照されたい。

|

テンプレートHTMLの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:ref:`create-prototype-of-tutorial-todo-label` で作成したプロトタイプにThymeleafの属性を付与してテンプレートHTMLを実装し、Controllerから渡されたModelを表示する。

TODOの一覧表示エリアを表示するために必要なテンプレートHTMLの実装を行う。 

.. code-block:: html
  :emphasize-lines: 2-3, 7-9, 21-28

  <!DOCTYPE html>
  <!-- (1) -->
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Todo List</title>
  <!-- (2) -->
  <link rel="stylesheet"
      href="../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
  </head>
  <body>
      <h1>Todo List</h1>
      <div id="todoForm">
          <form action="/todo/create" method="post">
              <input type="text">
              <button>Create Todo</button>
          </form>
      </div>
      <hr />
      <div id="todoList">
          <!-- (3) -->
          <ul th:remove="all-but-first">
              <!-- (4) -->
              <li th:each="todo : ${todos}">
                  <!-- (5)(6) -->
                  <span th:class="${todo.finished} ? 'strike'" th:text="${todo.todoTitle}">Send a e-mail</span>
                  <!-- (7) -->
                  <form th:if="${!todo.finished}" action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span>Have a lunch</span>
                  <form action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span class="strike">Read a book</span>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
          </ul>
      </div>
  </body>
  </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | Thymeleaf独自の属性を使用するため、\ ``<html>``\ タグにThymeleafのネームスペースを付与する。
  * - | (2)
    - | \ ``<link>``\ タグに\ ``th:href``\ 属性を付与する。
      | \ ``th:href``\ 属性値には、リンクURL式 \ ``@{}``\ を用いている。
      | リンクURL式に"\ ``/``\ "（スラッシュ）から始まるパスを指定することで、コンテキストルートからの相対パスが出力される。
  * - | (3)
    - | 最初の子要素をThymeleafのテンプレートとして利用し、2番目以降の子要素は静的表示時のみに表示するために、Thymeleafの\ ``th:remove``\ 属性を使用する。
      | \ ``th:remove``\ 属性に\ ``all-but-first``\ を指定することで、Thymeleafでの処理時には、指定したタグにおける最初の子要素以外の要素が削除される。
  * - | (4)
    - | \ ``th:each``\ 属性の右項にはControllerでModelに追加したコレクション\ ``todos``\ を指定し、左項にはコレクションの要素オブジェクトを格納する変数名\ ``todo``\を指定している。
      | これにより、\ ``th:each``\ 属性を付与した配下の要素が\ ``todos``\ の要素数分繰り返し出力される。
  * - | (5)
    - | \ ``th:class``\ 属性を使用することで、動的に\ ``class``\ 属性を設定できる。
      | \ ``th:text``\ 属性と同様に、変数式を利用してModelに登録した変数や\ ``th:each``\ 属性で定義した変数を参照できる。
      | ここではEL式を利用して、\ ``th:each``\ 属性で取り出した\ ``Todo``\ 型オブジェクト\ ``todo``\ の\ ``finished``\ プロパティを参照して打ち消し線(\ ``text-decoration: line-through;``\ )を装飾するかどうかを判断する。
  * - | (6)
    - | \ ``th:text``\ 属性を使用することで、記述した要素のコンテンツを属性値で上書きする。
      | \ **文字列値を出力する際は、XSS対策のため、必ずth:text属性を使用してHTMLエスケープを行うこと。**\
      | XSS対策についての詳細は、\ :ref:`xss_how_to_use_ouput_escaping`\ を参照されたい。
  * - | (7)
    - | \ ``th:if``\ 属性は条件に応じて、要素を出力するかどうか制御するための属性であり、\ ``todo``\ の\ ``finished``\ プロパティを参照して「Finish」ボタンの生成を判断する。

.. note::

  Thymeleafの\ ``th:object``\ 属性を用いると、オブジェクト名を省略してプロパティを指定することが出来る。
    
  list.htmlの\ ``<li>``\ タグの部分は、\ ``th:object``\ 属性を用いることで以下のように記述量を減らすことが出来る。


  * \ ``list.html``\

    .. code-block:: html
      :emphasize-lines: 1, 3

      <!-- (1) -->
      <li th:each="todo : ${todos}" th:object="${todo}">
          <!-- (2) -->
          <span th:class="*{finished} ? 'strike'" th:text="*{todoTitle}">Send a e-mail</span>
          <form th:if="*{!finished}" action="/todo/finish" method="post" class="inline">
              <button>Finish</button>
          </form>
          <form action="/todo/delete" method="post" class="inline">
              <button>Delete</button>
          </form>
      </li>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``th:object``\ 属性にオブジェクトを変数式\ ``${}``\ で指定する。
      * - | (2)
        - | オブジェクトのプロパティを選択変数式\ ``*{}``\ で指定する。これは、変数式を用いて\ ``th:class="${todo.finished} ? 'strike'"``\ や\ ``th:text="${todo.todoTitle}"``\ と指定するのと同じ結果になる。

|

| STSで「todo」プロジェクトを右クリックし、「Run As」→「Run on Server」でWebアプリケーションを起動する。
| ブラウザで http://localhost:8080/todo/todo/list にアクセスすると、以下のような画面が表示される。

.. figure:: ./images_TutorialTodo/image067.png
  :width: 25%

なお、表示されている「Create Todo」ボタンについては、「Create TODO」の実装が終了していないため、表示はされるが機能しない。

|

.. note::

  上記で表示されている画面には、TODOが1件も登録されていないため、TODOの一覧は出力されない。
    
  以下のように、ドメイン層の作成で作成したTodoRepositoryImplを一時的に修正し初期データを登録することで、TODOの一覧が出力されることを確認できる。
    
  なお、次節「\ :ref:`CreateTodoImplementation`\ 」で実際にTODOを登録できるようになるため、一覧の出力が確認できたら削除して構わない。

  * \ ``TodoRepositoryImpl.java``\

    .. code-block:: java
      :emphasize-lines: 15-29

      package com.example.todo.domain.repository.todo;

      import java.util.Collection;
      import java.util.Map;
      import java.util.concurrent.ConcurrentHashMap;

      import org.springframework.stereotype.Repository;

      import com.example.todo.domain.model.Todo;

      @Repository
      public class TodoRepositoryImpl implements TodoRepository {
          private static final Map<String, Todo> TODO_MAP = new ConcurrentHashMap<String, Todo>();

          static {
              Todo todo1 = new Todo();
              todo1.setTodoId("1");
              todo1.setTodoTitle("Send a e-mail");
              Todo todo2 = new Todo();
              todo2.setTodoId("2");
              todo2.setTodoTitle("Have a lunch");
              Todo todo3 = new Todo();
              todo3.setTodoId("3");
              todo3.setTodoTitle("Read a book");
              todo3.setFinished(true);
              TODO_MAP.put(todo1.getTodoId(), todo1);
              TODO_MAP.put(todo2.getTodoId(), todo2);
              TODO_MAP.put(todo3.getTodoId(), todo3);
          }

          // omitted

  以下のように画面に出力される。

  .. figure:: ./images_TutorialTodo/show-all-todo-note.png
    :width: 30%

|

.. _CreateTodoImplementation:

Create TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

次に、一覧表示画面から「Create TODO」ボタンを押した後の、新規作成処理を実装する。

はじめに、TODOの全件表示を行うための処理を実装する。

|

マッパーインタフェースの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Beanマッピングのマッパーインタフェースを作成する。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - \ ``com.example.todo.app.todo``\
    * - 2
      - Name
      - \ ``TodoMapper``\

を入力して「Finish」する。

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/create-bean-mapper.png

作成したクラスに以下の\ ``@Mapper``\ アノテーションを付与したBeanマッピングメソッドを追加する。

* Todo map(TodoForm form)

  * \ ``@Mapping``\ アノテーションによるマッピング除外項目定義

    * createdAt
    * finished

.. code-block:: java

  package com.example.todo.app.todo;

  import org.mapstruct.Mapper;
  import org.mapstruct.Mapping;

  import com.example.todo.domain.model.Todo;

  @Mapper
  public interface TodoMapper {

      @Mapping(target = "createdAt", ignore = true)
      @Mapping(target = "finished", ignore = true)
      Todo map(TodoForm form);

  }

.. note::

  マッパーインタフェース追加後、以下のようなビルドエラーが発生する場合がある。

    .. code-block:: console

      Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.example.todo.app.todo.TodoMapper'

  この場合は、プロジェクト名を右クリックし、「Run As」->「Maven build」をクリックする。
  Goalsに「compile」を指定し「Run」をクリックする。

  .. figure:: ./images_TutorialTodo/mvnBuild.png
    :width: 40%

  ビルドが成功した後、プロジェクト名を右クリックし、「Run As」->「Maven install」をクリックする。

|

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

新規作成処理を\ ``TodoController``\ に追加する。

.. code-block:: java
  :emphasize-lines: 30-32,47-71

  package com.example.todo.app.todo;

  import java.util.Collection;

  import jakarta.inject.Inject;
  import jakarta.validation.Valid;

  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingResult;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.servlet.mvc.support.RedirectAttributes;
  import org.terasoluna.gfw.common.exception.BusinessException;
  import org.terasoluna.gfw.common.message.ResultMessage;
  import org.terasoluna.gfw.common.message.ResultMessages;

  import com.example.todo.domain.model.Todo;
  import com.example.todo.domain.service.todo.TodoService;

  @Controller
  @RequestMapping("todo")
  public class TodoController {

      @Inject
      TodoService todoService;

      // (1)
      @Inject
      TodoMapper beanMapper;

      @ModelAttribute
      public TodoForm setUpForm() {
          TodoForm form = new TodoForm();
          return form;
      }

      @GetMapping("list")
      public String list(Model model) {
          Collection<Todo> todos = todoService.findAll();
          model.addAttribute("todos", todos);
          return "todo/list";
      }

      @PostMapping("create") // (2)
      public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (3)
              Model model, RedirectAttributes attributes) { // (4)

          // (5)
          if (bindingResult.hasErrors()) {
              return list(model);
          }

          // (6)
          Todo todo = beanMapper.map(todoForm);

          try {
              todoService.create(todo);
          } catch (BusinessException e) {
              // (7)
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          // (8)
          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Created successfully!")));
          return "redirect:/todo/list";
      }

  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | FormオブジェクトをDomainObjectに変換するために、``TodoMapper``\ インタフェースをインジェクションする。
  * - | (2)
    - | \ ``/todo/create``\ というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、新規作成処理用のメソッド(\ ``create``\ メソッド)が実行されるように\ ``@PostMapping``\ アノテーションを設定する。
  * - | (3)
    - | フォームの入力チェックを行うため、Formの引数に\ ``@Valid``\ アノテーションをつける。入力チェック結果は、その直後の引数\ ``BindingResult``\ に格納される。
  * - | (4)
    - | 正常に作成が完了した後にリダイレクトし、一覧画面を表示する。
      | リダイレクト先への情報を格納するために、引数に\ ``RedirectAttributes``\ を加える。
  * - | (5)
    - | 入力エラーがあった場合、一覧画面に戻る。
      | Todo全件取得を再度行う必要があるので、\ ``list``\ メソッドを再実行する。
  * - | (6)
    - | \ ``Mapstruct``\ を用いて、\ ``TodoForm``\ オブジェクトから\ ``Todo``\ オブジェクトを作成する。
  * - | (7)
    - | 業務処理を実行して、\ ``BusinessException``\ が発生した場合、結果メッセージを\ ``Model``\ に追加して、一覧画面に戻る。
  * - | (8)
    - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。
      | リダイレクトすることにより、ブラウザを再読み込みして、再び新規登録処理が\ ``POST``\ されることがなくなる。（詳しくは、「\ :ref:`DoubleSubmitProtectionAboutPRG`\ 」を参照されたい）
      | なお、今回は成功メッセージであるため、\ ``ResultMessages.success()``\ を使用している。

|

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

入力チェックのルールを定義するため、Formオブジェクトにアノテーションを追加する。

.. code-block:: java
  :emphasize-lines: 5-6,11-12

  package com.example.todo.app.todo;

  import java.io.Serializable;

  import jakarta.validation.constraints.NotNull;
  import jakarta.validation.constraints.Size;

  public class TodoForm implements Serializable {
      private static final long serialVersionUID = 1L;

      @NotNull // (1)
      @Size(min = 1, max = 30) // (2)
      private String todoTitle;

      public String getTodoTitle() {
          return todoTitle;
      }

      public void setTodoTitle(String todoTitle) {
          this.todoTitle = todoTitle;
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 80

  * - 項番
    - 説明
  * - | (1)
    - | \ ``@NotNull``\ アノテーションを使用して必須チェックを有効化する。
  * - | (2)
    - | \ ``@Size``\ アノテーションを使用して文字数チェックを有効化する。

|

テンプレートHTMLの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

TODOを新規作成するため、テンプレートHTMLに以下の実装を追加する。

* TODOの入力フォームにThymeleafの属性を付与する
* 入力チェックエラーを表示するエリアを追加する
* 結果メッセージを表示するエリアを追加する

.. code-block:: html
  :emphasize-lines: 12-25

  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Todo List</title>
  <link rel="stylesheet"
      href="../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
  </head>
  <body>
      <h1>Todo List</h1>
      <div id="todoForm">
          <!-- (1) -->
          <div th:if="${resultMessages} != null" class="alert alert-success" th:class="|alert alert-${resultMessages.type}|">
              <ul>
                  <li th:each="message : ${resultMessages}" th:text="${message.text}">Created successfully!</li>
              </ul>
          </div>
          <!-- (2) -->
          <form action="/todo/create" th:action="@{/todo/create}" method="post">
              <!-- (3) -->
              <input type="text" th:field="${todoForm.todoTitle}" />
              <!-- (4) -->
              <span id="todoTitle.errors" th:errors="${todoForm.todoTitle}" class="text-error">size must be between 1 and 30</span>
              <button>Create Todo</button>
          </form>
      </div>
      <hr />
      <div id="todoList">
          <ul th:remove="all-but-first">
              <li th:each="todo : ${todos}">
                  <span th:class="${todo.finished} ? 'strike'" th:text="${todo.todoTitle}">Send a e-mail</span>
                  <form th:if="${!todo.finished}" action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span>Have a lunch</span>
                  <form action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span class="strike">Read a book</span>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
          </ul>
      </div>
  </body>
  </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 80

  * - 項番
    - 説明
  * - | (1)
    - | 新規作成処理の結果メッセージを表示する。
      | \ ``th:if``\ 属性を使用し、ServiceやControllerで\ ``resultMessages``\ オブジェクトがModelに登録されている場合のみ、結果メッセージを表示している。
      | また、\ ``th:class``\ 属性を使用することで、\ ``ResultMessages``\ に設定されたメッセージタイプ（例:\ ``info``\ ,\ ``error``\ ）に応じた\ ``class``\ 属性を設定している。

       .. note::

         一般的にThymeleafを利用して画面を実装する場合、HTMLファイルを直接ブラウザで表示することを考慮し、Thymeleafのテンプレートとしては不要だがHTML表示時に必要となる属性や文字列（コード例における\ ``class="alert alert-success"``\ や\ ``Created successfully!``\ ）を記述する。

  * - | (2)
    - | 新規作成処理用のformを実装する。
      | \ ``th:action``\ 属性には、リンクURL式 \ ``@{}``\ を用いて新規作成処理を実行するためのパス（\ ``/todo/create``\ ）を指定する。
  * - | (3)
    - | \ ``<input>``\ タグでフォームのプロパティをバインドする。
      | \ ``th:field``\ 属性値を\ ``<input>``\ タグに適用すると、\ ``id``\ 属性、\ ``name``\ 属性、\ ``value``\ 属性が付加される。
  * - | (4)
    - | \ ``th:errors``\ 属性を付与することで、指定したプロパティに対する入力エラーがあった場合に表示される。\ ``th:errors``\ 属性の値は、\ ``<input>``\ タグの\ ``th:field``\ 属性と合わせる。

|

フォームに適切な値を入力してsubmitすると、以下のように、成功メッセージが表示される。

.. figure:: ./images_TutorialTodo/image068.png
  :width: 40%

.. figure:: ./images_TutorialTodo/image069.png
  :width: 40%

なお、TODOの横に表示されている「Finish」、「Delete」ボタンについては、「Finish TODO」、「Delete TODO」の実装が終了していないため、表示はされるが機能しない。

未完了のTODOが5件登録済みの場合は、業務エラーとなり、エラーメッセージが表示される。

.. figure:: ./images_TutorialTodo/image070.png
  :width: 60%


入力フォームを、空文字にしてsubmitすると、以下のように、エラーメッセージが表示される。

.. figure:: ./images_TutorialTodo/image071.png
  :width: 65%

|

Finish TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「Finish」ボタンにTODOを完了させるための処理を追加する。

|

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理用のFormについても、\ ``TodoForm``\ を使用する。

| \ ``TodoForm``\ に\ ``todoId``\ プロパティを追加する必要があるが、単純に追加してしまうと、新規作成処理でも\ ``todoId``\プロパティのチェックが実行されてしまう。
| 一つのFormクラスを使用して複数のformから送信されるリクエストパラメータをバインドする場合は、\ ``groups``\ 属性を使用して、入力チェックルールをグループ化する。

Formクラスに以下のプロパティを追加する。

* ID → todoId

.. code-block:: java
  :emphasize-lines: 9-11,13-14,18-20,22-24,27-29,31-33

  package com.example.todo.app.todo;

  import java.io.Serializable;

  import jakarta.validation.constraints.NotNull;
  import jakarta.validation.constraints.Size;

  public class TodoForm implements Serializable {
      // (1)
      public static interface TodoCreate {
      };

      public static interface TodoFinish {
      };

      private static final long serialVersionUID = 1L;

      // (2)
      @NotNull(groups = { TodoFinish.class })
      private String todoId;

      // (3)
      @NotNull(groups = { TodoCreate.class })
      @Size(min = 1, max = 30, groups = { TodoCreate.class })
      private String todoTitle;

      public String getTodoId() {
          return todoId;
      }

      public void setTodoId(String todoId) {
          this.todoId = todoId;
      }

      public String getTodoTitle() {
          return todoTitle;
      }

      public void setTodoTitle(String todoTitle) {
          this.todoTitle = todoTitle;
      }

  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90


  * - 項番
    - 説明
  * - | (1)
    - | 入力チェックルールをグループ化するためのインタフェースを作成する。
      | 入力チェックルールのグループ化については、\ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ を参照されたい。
      |
      | ここでは、新規作成処理用のインタフェースとして\ ``TodoCreate``\ を、完了処理用のインタフェースとして\ ``TodoFinish``\ を作成している。
  * - | (2)
    - | \ ``todoId``\ は完了処理で使用するプロパティである。
      | そのため、\ ``@NotNull``\ アノテーションの\ ``groups``\ 属性には、完了処理用の入力チェックルールである事を示す\ ``TodoFinish``\ インタフェースを指定する。
  * - | (3)
    - | \ ``todoTitle``\ は新規作成処理で使用するプロパティである。
      | そのため、\ ``@NotNull``\ アノテーションと\ ``@Size``\ アノテーションの\ ``groups``\ 属性には、新規作成処理用の入力チェックルールである事を示す\ ``TodoCreate``\ インタフェースを指定する。

|

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理を\ ``TodoController``\ に追加する。

グループ化した入力チェックルールを適用するためには、\ **@Valid アノテーションの代わりに、@Validated アノテーションを使用すること**\ に注意する。

.. code-block:: java
  :emphasize-lines: 6,11,51,73-95

  package com.example.todo.app.todo;

  import java.util.Collection;

  import jakarta.inject.Inject;
  import jakarta.validation.groups.Default;

  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.servlet.mvc.support.RedirectAttributes;
  import org.terasoluna.gfw.common.exception.BusinessException;
  import org.terasoluna.gfw.common.message.ResultMessage;
  import org.terasoluna.gfw.common.message.ResultMessages;

  import com.example.todo.app.todo.TodoForm.TodoCreate;
  import com.example.todo.app.todo.TodoForm.TodoFinish;
  import com.example.todo.domain.model.Todo;
  import com.example.todo.domain.service.todo.TodoService;

  @Controller
  @RequestMapping("todo")
  public class TodoController {

      @Inject
      TodoService todoService;

      @Inject
      TodoMapper beanMapper;

      @ModelAttribute
      public TodoForm setUpForm() {
          TodoForm form = new TodoForm();
          return form;
      }

      @GetMapping("list")
      public String list(Model model) {
          Collection<Todo> todos = todoService.findAll();
          model.addAttribute("todos", todos);
          return "todo/list";
      }

      @PostMapping("create")
      public String create(
              @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (1)
              BindingResult bindingResult, Model model,
              RedirectAttributes attributes) {

          if (bindingResult.hasErrors()) {
              return list(model);
          }

          Todo todo = beanMapper.map(todoForm);

          try {
              todoService.create(todo);
          } catch (BusinessException e) {
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Created successfully!")));
          return "redirect:/todo/list";
      }

      @PostMapping("finish") // (2)
      public String finish(
              @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (3)
              BindingResult bindingResult, Model model,
              RedirectAttributes attributes) {
          // (4)
          if (bindingResult.hasErrors()) {
              return list(model);
          }

          try {
              todoService.finish(form.getTodoId());
          } catch (BusinessException e) {
              // (5)
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          // (6)
          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Finished successfully!")));
          return "redirect:/todo/list";
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | グループ化した入力チェックルールを適用するために、\ ``@Valid``\ アノテーションを\ ``@Validated``\ アノテーションに変更する。

      | \ ``value``\ 属性には、適用する入力チェックルールのグループ(グループインタフェース)を指定する。
      | \ ``Default.class``\ は、グループ化されていない入力チェックルールを適用するために用意されているグループインタフェースである。
  * - | (2)
    - | \ ``/todo/finish``\というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、完了処理用のメソッド(\ ``finish``\ メソッド)が実行されるように\ ``@PostMapping``\ アノテーションを設定する。
  * - | (3)
    - | 適用する入力チェックのグループとして、完了処理用のグループインタフェース(\ ``TodoFinish``\ インタフェース)を指定する。
  * - | (4)
    - | 入力エラーがあった場合、一覧画面に戻る。
  * - | (5)
    - | 業務処理を実行して、\ ``BusinessException``\ が発生した場合は、結果メッセージを\ ``Model``\ に追加して、一覧画面に戻る。
  * - | (6)
    - | 正常に作成が完了した場合は、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。

.. note::

  新規作成処理用と完了処理用を別々のFormクラスとして作成しても良い。別々のFormクラスにした場合、入力チェックルールをグループ化する必要がないため、入力チェックルールの定義はシンプルになる。

  ただし、処理毎にFormクラスを作成した場合、

  * クラス数が増える
  * プロパティが重複するため入力チェックルールを一元管理できない

  ため、仕様変更が発生した場合に修正コストが高くなる可能性があるという点に注意してほしい。

  また、\ ``@ModelAttribute``\ メソッドを使用して複数のFormを初期化した場合、毎回すべてのFormが初期化されるため、不要なインスタンスが生成されることになる。

|

テンプレートHTMLの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理用のformを実装する。

.. code-block:: html
  :emphasize-lines: 28-34

  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Todo List</title>
  <link rel="stylesheet"
      href="../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
  </head>
  <body>
      <h1>Todo List</h1>
      <div id="todoForm">
          <div th:if="${resultMessages} != null" class="alert alert-success" th:class="|alert alert-${resultMessages.type}|">
              <ul>
                  <li th:each="message : ${resultMessages}" th:text="${message.text}">Created successfully!</li>
              </ul>
          </div>
          <form action="/todo/create" th:action="@{/todo/create}" method="post">
              <input type="text" th:field="${todoForm.todoTitle}" />
              <span id="todoTitle.errors" th:errors="${todoForm.todoTitle}" class="text-error">size must be between 1 and 30</span>
              <button>Create Todo</button>
          </form>
      </div>
      <hr />
      <div id="todoList">
          <ul th:remove="all-but-first">
              <li th:each="todo : ${todos}">
                  <span th:class="${todo.finished} ? 'strike'" th:text="${todo.todoTitle}">Send a e-mail</span>
                  <!-- (1) -->
                  <form th:if="${!todo.finished}" action="/todo/finish" th:action="@{/todo/finish}"
                      method="post" class="inline">
                      <!-- (2) -->
                      <input type="hidden" name="todoId" th:value="${todo.todoId}">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span>Have a lunch</span>
                  <form action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span class="strike">Read a book</span>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
          </ul>
      </div>
  </body>
  </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``th:if``\ 属性を使用し、TODOが未完了の場合は、TODOを完了させるためのリクエストを送信するformを表示する。
      | \ ``th:action``\ 属性にはリンクURL式 \ ``@{}``\ を用いて完了処理を実行するためのパス（\ ``/todo/finish``\ ）を指定する。
  * - | (2)
    - | リクエストパラメータとして\ ``todoId``\ を送信する。
      | \ ``th:value``\ 属性を使用して、\ ``todo``\ オブジェクトの\ ``todoId``\ プロパティを値に設定している。

|

TODOを新規作成した後に、「Finish」ボタン押下すると、以下のように打ち消し線が入り、完了したことがわかる。

.. figure:: ./images_TutorialTodo/image075.png
  :width: 40%

.. figure:: ./images_TutorialTodo/image076.png
  :width: 40%

|

Delete TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「Delete」ボタンにTODOを削除するための処理を追加する。

|

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

削除処理用のFormについても、\ ``TodoForm``\ を使用する。

.. code-block:: java
  :emphasize-lines: 15-17,21-22

  package com.example.todo.app.todo;

  import java.io.Serializable;

  import jakarta.validation.constraints.NotNull;
  import jakarta.validation.constraints.Size;

  public class TodoForm implements Serializable {
      public static interface TodoCreate {
      };

      public static interface TodoFinish {
      };

      // (1)
      public static interface TodoDelete {
      }

      private static final long serialVersionUID = 1L;

      // (2)
      @NotNull(groups = { TodoFinish.class, TodoDelete.class })
      private String todoId;

      @NotNull(groups = { TodoCreate.class })
      @Size(min = 1, max = 30, groups = { TodoCreate.class })
      private String todoTitle;

      public String getTodoId() {
          return todoId;
      }

      public void setTodoId(String todoId) {
          this.todoId = todoId;
      }

      public String getTodoTitle() {
          return todoTitle;
      }

      public void setTodoTitle(String todoTitle) {
          this.todoTitle = todoTitle;
      }

  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 削除処理用の入力チェックルールをグループ化するためのインタフェースとして\ ``TodoDelete``\ を作成する。
  * - | (2)
    - | 削除処理では\ ``todoId``\ プロパティを使用する。
      | そのため、\ ``todoId``\ の\ ``@NotNull``\ アノテーションの\ ``groups``\ 属性には、削除処理用の入力チェックルールである事を示す\ ``TodoDelete``\ インタフェースを指定する。

|

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

削除処理を\ ``TodoController``\ に追加する。完了処理とほぼ同じである。

.. code-block:: java
  :emphasize-lines: 95-115

  package com.example.todo.app.todo;

  import java.util.Collection;

  import jakarta.inject.Inject;
  import jakarta.validation.groups.Default;

  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.servlet.mvc.support.RedirectAttributes;
  import org.terasoluna.gfw.common.exception.BusinessException;
  import org.terasoluna.gfw.common.message.ResultMessage;
  import org.terasoluna.gfw.common.message.ResultMessages;

  import com.example.todo.app.todo.TodoForm.TodoDelete;
  import com.example.todo.app.todo.TodoForm.TodoCreate;
  import com.example.todo.app.todo.TodoForm.TodoFinish;
  import com.example.todo.domain.model.Todo;
  import com.example.todo.domain.service.todo.TodoService;

  @Controller
  @RequestMapping("todo")
  public class TodoController {

      @Inject
      TodoService todoService;

      @Inject
      TodoMapper beanMapper;

      @ModelAttribute
      public TodoForm setUpForm() {
          TodoForm form = new TodoForm();
          return form;
      }

      @GetMapping("list")
      public String list(Model model) {
          Collection<Todo> todos = todoService.findAll();
          model.addAttribute("todos", todos);
          return "todo/list";
      }

      @PostMapping("create")
      public String create(
              @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm,
              BindingResult bindingResult, Model model,
              RedirectAttributes attributes) {

          if (bindingResult.hasErrors()) {
              return list(model);
          }

          Todo todo = beanMapper.map(todoForm);

          try {
              todoService.create(todo);
          } catch (BusinessException e) {
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Created successfully!")));
          return "redirect:/todo/list";
      }

      @PostMapping("finish")
      public String finish(
              @Validated({ Default.class, TodoFinish.class }) TodoForm form,
              BindingResult bindingResult, Model model,
              RedirectAttributes attributes) {
          if (bindingResult.hasErrors()) {
              return list(model);
          }

          try {
              todoService.finish(form.getTodoId());
          } catch (BusinessException e) {
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Finished successfully!")));
          return "redirect:/todo/list";
      }

      @PostMapping("delete") // (1)
      public String delete(
              @Validated({ Default.class, TodoDelete.class }) TodoForm form,
              BindingResult bindingResult, Model model,
              RedirectAttributes attributes) {

          if (bindingResult.hasErrors()) {
              return list(model);
          }

          try {
              todoService.delete(form.getTodoId());
          } catch (BusinessException e) {
              model.addAttribute(e.getResultMessages());
              return list(model);
          }

          attributes.addFlashAttribute(ResultMessages.success().add(
                  ResultMessage.fromText("Deleted successfully!")));
          return "redirect:/todo/list";
      }

  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90


  * - 項番
    - 説明
  * - | (1)
    - \ ``/todo/delete``\ というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、削除処理用のメソッド(\ ``delete``\ メソッド)が実行されるように\ ``@PostMapping``\ アノテーションを設定する。

|

テンプレートHTMLの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
削除処理用のformを実装する。

.. code-block:: html
  :emphasize-lines: 33-39

  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Todo List</title>
  <link rel="stylesheet"
      href="../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
  </head>
  <body>
      <h1>Todo List</h1>
      <div id="todoForm">
          <div th:if="${resultMessages} != null" class="alert alert-success" th:class="|alert alert-${resultMessages.type}|">
              <ul>
                  <li th:each="message : ${resultMessages}" th:text="${message.text}">Created successfully!</li>
              </ul>
          </div>
          <form action="/todo/create" th:action="@{/todo/create}" method="post">
              <input type="text" th:field="${todoForm.todoTitle}" />
              <span id="todoTitle.errors" th:errors="${todoForm.todoTitle}" class="text-error">size must be between 1 and 30</span>
              <button>Create Todo</button>
          </form>
      </div>
      <hr />
      <div id="todoList">
          <ul th:remove="all-but-first">
              <li th:each="todo : ${todos}">
                  <span th:class="${todo.finished} ? 'strike'" th:text="${todo.todoTitle}">Send a e-mail</span>
                  <form th:if="${!todo.finished}" action="/todo/finish" th:action="@{/todo/finish}"
                      method="post" class="inline">
                      <input type="hidden" name="todoId" th:value="${todo.todoId}" />
                      <button>Finish</button>
                  </form>
                  <!-- (1) -->
                  <form action="/todo/delete" th:action="@{/todo/delete}" 
                      method="post" class="inline">
                      <!-- (2) -->
                      <input type="hidden" name="todoId" th:value="${todo.todoId}" />
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span>Have a lunch</span>
                  <form action="/todo/finish" method="post" class="inline">
                      <button>Finish</button>
                  </form>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
              <li>
                  <span class="strike">Read a book</span>
                  <form action="/todo/delete" method="post" class="inline">
                      <button>Delete</button>
                  </form>
              </li>
          </ul>
      </div>
  </body>
  </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``th:action``\ 属性にはリンクURL式 \ ``@{}``\ を用いて削除処理を実行するためのパス（\ ``/todo/delete``\ ）を指定する。
  * - | (2)
    - | \ ``type="hidden"``\ 属性を使用して、リクエストパラメータとして\ ``todoId``\ を送信する。

|

未完了状態のTODOの「Delete」ボタンを押下すると、以下のようにTODOが削除される。

.. figure:: ./images_TutorialTodo/image077.png
  :width: 40%

.. figure:: ./images_TutorialTodo/image078.png
  :width: 40%

|

.. _tutorial-todo_infra:

データベースアクセスを伴うインフラストラクチャ層の作成
================================================================================

ここでは、Domainオブジェクトをデータベースに永続化するためのインフラストラクチャ層の実装方法について説明する。

本チュートリアルでは、MyBatis3を使用したインフラストラクチャ層の実装方法について説明する。

|

.. _TutorialCreateORMapperBlankProject:

O/R Mapperに依存したブランクプロジェクトの作成
--------------------------------------------------------------------------------

ここでは、O/R Mapperに依存したブランクプロジェクトの作成を行う。

まず、\ :ref:`TutorialCreateMyBatis3BlankProject`\を参考にプロジェクトを作成し直す。

次に、\ :ref:`tutorial-todo_infra`\ までで作成した\ :file:`src`\ フォルダ以下のうち、\ **TodoRepositoryImplクラス以外のファイルを新規作成したプロジェクトにコピーする**\ 。

\ **ただし、コピーするファイルは新規作成したファイル・変更を加えたファイルに限り、修正を加えていないファイルはコピーしないこと**\ 。

|

.. _Tutorial_Setup_Database:

データベースのセットアップ
--------------------------------------------------------------------------------

ここでは、データベースのセットアップを行う。

本チュートリアルでは、データベースのセットアップの手間を省くため、H2 Databaseを使用する。

|

todo-infra.propertiesの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

APサーバ起動時にH2 Database上にテーブルが作成されるようにするために、\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\ の設定を変更する。

.. code-block:: properties
  :emphasize-lines: 2-3

  database=H2
  # (1)
  database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
  database.username=sa
  database.password=
  database.driverClassName=org.h2.Driver
  # connection pool
  cp.maxActive=96
  cp.maxIdle=16
  cp.minIdle=0
  cp.maxWait=60000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 80

  * - 項番
    - 説明
  * - | (1)
    - | 接続URLのINITパラメータに、テーブルを作成するDDL文を指定する。

.. note::

  INITパラメータに設定しているDDL文をフォーマットすると、以下の様なSQLとなる。

    .. code-block:: sql

      create table if not exists todo (
          todo_id varchar(36) primary key,
          todo_title varchar(30),
          finished boolean,
          created_at timestamp
      )

|

.. _using_MyBatis3:

MyBatis3を使用したインフラストラクチャ層の作成
--------------------------------------------------------------------------------

ここでは、MyBatis3を使用してインフラストラクチャ層のRepositoryImplを作成する方法について説明する。

|

TodoRepositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ ``TodoRepository``\ は、O/R Mapperを使用しない場合と同じ方法で作成する。
| 作成方法は、「:ref:`TutorialTodoCreateRepository`」を参照されたい。
|

TodoRepositoryImplの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| MyBatis3を使用する場合、RepositoryImplはRepositoryインタフェース(Mapperインタフェース)から自動生成される。
| そのため、\ ``TodoRepositoryImpl``\ の作成は不要である。作成した場合は削除すること。
|

Mapperファイルの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``TodoRepository``\ インタフェースのメソッドが呼び出された際に実行するSQLを定義するためのMapperファイルを作成する。

Package Explorer上で右クリック -> New -> File を選択し、「Create New File」ダイアログを表示し、

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Enter or select the parent folder
      - \ ``todo/src/main/resources/com/example/todo/domain/repository/todo``\
    * - 2
      - File name
      - \ ``TodoRepository.xml``\

を入力して「Finish」する。

作成したファイルは以下のディレクトリに格納される。

.. figure:: ./images_TutorialTodo/create-mapper-for-mybatis3.png

\ ``TodoRepository``\ インタフェースに定義したメソッドが呼び出された際に実行するSQLを記述する。

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

  <!-- (1) -->
  <mapper namespace="com.example.todo.domain.repository.todo.TodoRepository">

      <!-- (2) -->
      <resultMap id="todoResultMap" type="Todo">
          <id property="todoId" column="todo_id" />
          <result property="todoTitle" column="todo_title" />
          <result property="finished" column="finished" />
          <result property="createdAt" column="created_at" />
      </resultMap>

      <!-- (3) -->
      <select id="findById" parameterType="String" resultMap="todoResultMap">
      <![CDATA[
          SELECT
              todo_id,
              todo_title,
              finished,
              created_at
          FROM
              todo
          WHERE
              todo_id = #{todoId}
      ]]>
      </select>

      <!-- (4) -->
      <select id="findAll" resultMap="todoResultMap">
      <![CDATA[
          SELECT
              todo_id,
              todo_title,
              finished,
              created_at
          FROM
              todo
      ]]>
      </select>

      <!-- (5) -->
      <insert id="create" parameterType="Todo">
      <![CDATA[
          INSERT INTO todo
          (
              todo_id,
              todo_title,
              finished,
              created_at
          )
          VALUES
          (
              #{todoId},
              #{todoTitle},
              #{finished},
              #{createdAt}
          )
      ]]>
      </insert>

      <!-- (6) -->
      <update id="update" parameterType="Todo">
      <![CDATA[
          UPDATE todo
          SET
              todo_title = #{todoTitle},
              finished = #{finished},
              created_at = #{createdAt}
          WHERE
              todo_id = #{todoId}
      ]]>
      </update>

      <!-- (7) -->
      <delete id="delete" parameterType="Todo">
      <![CDATA[
          DELETE FROM
              todo
          WHERE
              todo_id = #{todoId}
      ]]>
      </delete>

      <!-- (8) -->
      <select id="countByFinished" parameterType="Boolean"
          resultType="Long">
      <![CDATA[
          SELECT
              COUNT(*)
          FROM
              todo
          WHERE
              finished = #{finished}
      ]]>
      </select>

  </mapper>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``mapper``\ 要素の\ ``namespace``\ 属性に、Repositoryインタフェースの完全修飾クラス名(FQCN)を指定する。
  * - | (2)
    - | \ ``<resultMap>``\ 要素に、検索結果(\ ``ResultSet``\ )とJavaBeanのマッピング定義を行う。
      | マッピングファイルの詳細は\ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`\ を参照されたい。
  * - | (3)
    - | \ ``todoId``\ (PK)が一致するレコードを1件取得するSQLを実装する。
      | \ ``<select>``\ 要素の\ ``resultMap``\ 属性には、適用するマッピング定義のIDを指定する。
  * - | (4)
    - | 全レコードを取得するSQLを実装している。
      | \ ``<select>``\ 要素の\ ``resultMap``\ 属性に、適用するマッピング定義のIDを指定する。
      | アプリケーションの要件には記載がないが、最新のTODOが先頭に表示されるようにレコードを並び替えている。
  * - | (5)
    - | 引数に指定されたTodoオブジェクトを挿入するSQLを実装する。
      | \ ``<insert>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
  * - | (6)
    - | 引数に指定されたTodoオブジェクトを更新するSQLを実装する。
      | \ ``<update>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
  * - | (7)
    - | 引数に指定されたTodoオブジェクトを削除するSQLを実装する。
      | \ ``<delete>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
  * - | (8)
    - | 引数に指定された\ ``finished``\ に一致するTodoの件数を取得するSQLを実装する。

|

以上で、MyBatis3を使用したインフラストラクチャ層の作成が完了したので、Service及びアプリケーション層の作成を行う。

Service及びアプリケーション層を作成後にAPサーバーを起動し、Todoの表示を行うと、以下のようなSQLログやトランザクションログが出力される。

.. code-block:: console
  :emphasize-lines: 2-9

  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] TodoController.list(Model)
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Creating new transaction with name [com.example.todo.domain.service.todo.TodoServiceImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Acquired Connection [13059960, URL=jdbc:h2:mem:todo-mybatis3, H2 JDBC Driver] for JDBC transaction
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:c.e.t.d.repository.todo.TodoRepository.findAll  	message:==>  Preparing: SELECT todo_id, todo_title, finished, created_at FROM todo
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:c.e.t.d.repository.todo.TodoRepository.findAll  	message:==> Parameters: 
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:c.e.t.d.repository.todo.TodoRepository.findAll  	message:<==      Total: 0
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Initiating transaction commit
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Committing JDBC transaction on Connection [13059960, URL=jdbc:h2:mem:todo-mybatis3, H2 JDBC Driver]
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Releasing JDBC Connection [13059960, URL=jdbc:h2:mem:todo-mybatis3, H2 JDBC Driver] after transaction
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=com.example.todo.app.todo.TodoForm@4e5a28b2, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
  date:2022-12-01 16:49:29	thread:http-nio-8080-exec-2	X-Track:d3ef4363deef452d8823a00db0fbc71a	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] TodoController.list(Model)-> 205,791,900 ns

|

おわりに
================================================================================
このチュートリアルでは、以下の内容を学習した。


* Macchinetta Server Framework (1.x)による基本的なアプリケーションの開発方法

* MavenおよびSTS(Eclipse)プロジェクトの構築方法

* Macchinetta Server Framework (1.x)のアプリケーションのレイヤ化に従った開発方法

  * POJO(+ Spring)を使用したドメイン層の実装
  * POJO(+ Spring MVC)とThymeleafを使用したアプリケーション層の実装
  * MyBatis3を使用したインフラストラクチャ層の実装
  * O/R Mapperを使用しないインフラストラクチャ層の実装

本チュートリアルで作成したTODO管理アプリケーションには、以下の改善点がある。
アプリケーションの修正を学習課題として、ガイドライン中の該当する説明を参照されたい。

* プロパティ(未完了TODOの上限数)を外部化する → \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`\
* メッセージを外部化する → \ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\
* ページネーション機能を追加する → \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`\
* 例外ハンドリングを加える → \ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\
* 二重送信を防止する(トランザクショントークンチェックを追加する) → \ :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`\
* システム日時の取得元を変更する → \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`\

|

Appendix
================================================================================

.. _TutorialTodoAppendixExpoundConfigurations:

設定ファイルの解説
--------------------------------------------------------------------------------

| アプリケーションを動かすためにどのような設定が必要なのかを理解するために、設定ファイルの解説を行う。
| ここでは、チュートリアルで作成するTodoアプリケーションで使用しない設定については、解説を割愛している箇所がある。
|

web.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :file:`web.xml`\ には、WebアプリケーションとしてTodoアプリをデプロイするための設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/webapp/WEB-INF/web.xml`\ は、以下のような設定となっている。

.. code-block:: xml
  :emphasize-lines: 2, 17, 34, 88, 104, 120

  <?xml version="1.0" encoding="UTF-8"?>
  <!-- (1) -->
  <web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
      version="6.0">

      <context-param>
          <param-name>logbackDisableServletContainerInitializer</param-name>
          <param-value>true</param-value>
      </context-param>

      <listener>
          <listener-class>ch.qos.logback.classic.servlet.LogbackServletContextListener</listener-class>
      </listener>

      <!-- (2) -->
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <!-- Root ApplicationContext -->
          <param-value>
              classpath*:META-INF/spring/applicationContext.xml
              classpath*:META-INF/spring/spring-security.xml
          </param-value>
      </context-param>

      <listener>
          <listener-class>org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener</listener-class>
      </listener>

      <!-- (3) -->
      <filter>
          <filter-name>MDCClearFilter</filter-name>
          <filter-class>org.terasoluna.gfw.web.logging.mdc.MDCClearFilter</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>MDCClearFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

      <filter>
          <filter-name>exceptionLoggingFilter</filter-name>
          <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>exceptionLoggingFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

      <filter>
          <filter-name>XTrackMDCPutFilter</filter-name>
          <filter-class>org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>XTrackMDCPutFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

      <filter>
          <filter-name>CharacterEncodingFilter</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>UTF-8</param-value>
          </init-param>
          <init-param>
              <param-name>forceEncoding</param-name>
              <param-value>true</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>CharacterEncodingFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

      <filter>
          <filter-name>springSecurityFilterChain</filter-name>
          <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>springSecurityFilterChain</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

      <!-- (4) -->
      <servlet>
          <servlet-name>appServlet</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <!-- ApplicationContext for Spring MVC -->
              <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>appServlet</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>

      <!-- (5) -->
      <error-page>
          <error-code>500</error-code>
          <location>/common/error/systemError</location>
      </error-page>

      <error-page>
          <error-code>404</error-code>
          <location>/common/error/resourceNotFoundError</location>
      </error-page>

      <error-page>
          <exception-type>java.lang.Exception</exception-type>
          <location>/WEB-INF/views/common/error/unhandledSystemError.html</location>
      </error-page>

      <!-- (6) -->
      <session-config>
          <!-- 30min -->
          <session-timeout>30</session-timeout>
          <cookie-config>
              <http-only>true</http-only>
              <!-- <secure>true</secure> -->
          </cookie-config>
          <tracking-mode>COOKIE</tracking-mode>
      </session-config>

  </web-app>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | Servlet6.0を使用するための宣言。
  * - | (2)
    - | サーブレットコンテキストリスナーの定義。

      ブランクプロジェクトでは、

      * アプリケーション全体で使用される\ ``ApplicationContext``\ を作成するための\ ``ContextLoaderListener``\
      * HttpSessionに対する操作をログ出力するための\ ``HttpSessionEventLoggingListener``\

      が設定済みである。
  * - | (3)
    - | サーブレットフィルタの定義。

      ブランクプロジェクトでは、

      * 共通ライブラリから提供しているサーブレットフィルタ
      * Spring Frameworkから提供されている文字エンコーディングを指定するための\ ``CharacterEncodingFilter``\
      * Spring Securityから提供されている認証・認可用のサーブレットフィルタ

       が設定済みである。
  * - | (4)
    - | Spring MVCのエントリポイントとなるDispatcherServletの定義。
      |
      | DispatcherServletの中で使用する\ ``ApplicationContext``\ を、(2)で作成した\ ``ApplicationContext``\ の子として作成する。
      | (2)で作成した\ ``ApplicationContext``\ を親にすることで、(2)で読み込まれたコンポーネントも使用することができる。
  * - | (5)
    - | エラーページの定義。

      ブランクプロジェクトでは、

      * サーブレットコンテナにHTTPステータスコードとして、\ ``404``\又は\ ``500``\が応答
      * サーブレットコンテナに例外が通知

      された際の遷移先が定義済みである。
  * - | (6)
    - | セッション管理の定義。

      ブランクプロジェクトでは、

      * セッションタイムアウトとして、30分

      が定義済みである。

|

Bean定義ファイル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作成したブランクプロジェクトには、以下のBean定義ファイルとプロパティファイルが作成される。

* \ :file:`src/main/resources/META-INF/spring/applicationContext.xml`\
* \ :file:`src/main/resources/META-INF/spring/todo-domain.xml`\
* \ :file:`src/main/resources/META-INF/spring/todo-infra.xml`\
* \ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\
* \ :file:`src/main/resources/META-INF/spring/todo-env.xml`\
* \ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\
* \ :file:`src/main/resources/META-INF/spring/spring-security.xml`\

.. note::

  O/R Mapperに依存しないブランクプロジェクトを作成した場合は、\ ``todo-infra.properties``\ と\ ``todo-env.xml``\ は作成されない。

.. note::

  本ガイドラインでは、Bean定義ファイルを役割(層)ごとにファイルを分割することを推奨している。

  これは、どこに何が定義されているか想像しやすく、メンテナンス性が向上するからである。

  今回のチュートリアルのような小さなアプリケーションでは効果はないが、アプリケーションの規模が大きくなるにつれ、効果が大きくなる。

|

applicationContext.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`applicationContext.xml`\ には、Todoアプリ全体に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/applicationContext.xml`\  は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

.. code-block:: xml
  :emphasize-lines: 10-11, 35-37

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
      ">

      <!-- (1) -->
      <import resource="classpath:/META-INF/spring/todo-domain.xml" />

      <bean id="passwordEncoder" class="org.springframework.security.crypto.password.DelegatingPasswordEncoder">
          <constructor-arg name="idForEncode" value="pbkdf2@SpringSecurity_v5_8" />
          <constructor-arg name="idToPasswordEncoder">
              <map>
                  <entry key="pbkdf2@SpringSecurity_v5_8">
                      <bean class="org.springframework.security.crypto.password.Pbkdf2PasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
                  </entry>
                  <entry key="bcrypt">
                      <bean class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />
                  </entry>
                  <!-- When using commented out PasswordEncoders, you need to add bcprov-jdk18on.jar to the dependency.
                  <entry key="argon2@SpringSecurity_v5_8">
                      <bean class="org.springframework.security.crypto.argon2.Argon2PasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
                  </entry>
                  <entry key="scrypt@SpringSecurity_v5_8">
                      <bean class="org.springframework.security.crypto.scrypt.SCryptPasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
                  </entry>
                  -->
              </map>
          </constructor-arg>
      </bean>

      <!-- (2) -->
      <context:property-placeholder
          location="classpath*:/META-INF/spring/*.properties" />

      <!-- Message -->
      <bean id="messageSource"
          class="org.springframework.context.support.ResourceBundleMessageSource">
          <property name="basenames">
              <list>
                  <value>i18n/application-messages</value>
              </list>
          </property>
      </bean>

      <!-- Exception Code Resolver. -->
      <bean id="exceptionCodeResolver"
          class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
          <!-- Setting and Customization by project. -->
          <property name="exceptionMappings">
              <map>
                  <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
                  <entry key="InvalidTransactionTokenException" value="e.xx.fw.7001" />
                  <entry key="BusinessException" value="e.xx.fw.8001" />
                  <entry key=".DataAccessException" value="e.xx.fw.9002" />
              </map>
          </property>
          <property name="defaultExceptionCode" value="e.xx.fw.9001" />
      </bean>

      <!-- Exception Logger. -->
      <bean id="exceptionLogger"
          class="org.terasoluna.gfw.common.exception.ExceptionLogger">
          <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
      </bean>

      <!-- Filter. -->
      <bean id="exceptionLoggingFilter"
          class="org.terasoluna.gfw.web.exception.ExceptionLoggingFilter" >
          <property name="exceptionLogger" ref="exceptionLogger" />
      </bean>

  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | ドメイン層に関するBean定義ファイルをimportする。
  * - | (2)
    - | プロパティファイルの読み込み設定を行う。
      | \ ``src/main/resources/META-INF/spring``\ 直下の任意のプロパティファイルを読み込む。
      | この設定により、プロパティファイルの値をBean定義ファイル内で\ ``${propertyName}``\ 形式で埋め込んだり、Javaクラスに\ ``@Value("${propertyName}")``\ でインジェクションすることができる。

.. tip::

  エディタの「Configure Namespaces」タブにて、以下のようにチェックを入れると、チェックしたXMLスキーマが有効になり、XML編集時にCtrl+Spaceを使用して入力を補完することができる。

  「Namespace Versions」にはバージョンなしのxsdファイルを選択することを推奨する。

  バージョンなしのxsdファイルを選択することで、常にjarに含まれる最新のxsdが使用されるため、Springのバージョンアップを意識する必要がなくなる。

    .. figure:: ./images_TutorialTodo/image021.jpg
      :width: 90%

    .. figure:: ./images_TutorialTodo/image023.png
      :width: 60%

|

todo-domain.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-domain.xml`\ には、Todoアプリのドメイン層に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-domain.xml`\ は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

.. code-block:: xml
  :emphasize-lines: 12-13, 16-17

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="
          http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
      ">

      <!-- (1) -->
      <import resource="classpath:META-INF/spring/todo-infra.xml" />
      <import resource="classpath*:META-INF/spring/**/*-codelist.xml" />

      <!-- (2) -->
      <context:component-scan base-package="com.example.todo.domain" />

      <!-- AOP. -->
      <bean id="resultMessagesLoggingInterceptor"
          class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor">
          <property name="exceptionLogger" ref="exceptionLogger" />
      </bean>
      <aop:config>
          <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
              pointcut="@within(org.springframework.stereotype.Service)" />
      </aop:config>

  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | インフラストラクチャ層に関するBean定義ファイルをimportする。
  * - | (2)
    - | ドメイン層のクラスを管理するcom.example.todo.domainパッケージ配下をcomponent-scan対象とする。
      | これにより、com.example.todo.domainパッケージ配下のクラスに ``@Repository``\ , ``@Service``\ などのアノテーションを付けることで、Spring Framerowkが管理するBeanとして登録される。
      | 登録されたクラス(Bean)は、ControllerやServiceクラスにDIする事で、利用する事が出来る。

.. note::

  O/R Mapperに依存するブランクプロジェクトを作成した場合は、\ ``@Transactional``\アノテーションによるトランザクション管理を有効にするために、\ ``<tx:annotation-driven>``\ タグが設定されている。

    .. code-block:: xml

      <tx:annotation-driven />

|

todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.xml`\ には、Todoアプリのインフラストラクチャ層に関わる設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.xml`\ は、以下のような設定となっている。

| \ :file:`todo-infra.xml`\は、インフラストラクチャ層によって設定が大きく異なるため、ブランクプロジェクト毎に説明を行う。
| 作成したブランクプロジェクト以外の説明は読み飛ばしてもよい。
|

O/R Mapperに依存しないブランクプロジェクトを作成した場合のtodo-infra.xml
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

O/R Mapperに依存しないブランクプロジェクトを作成した場合、以下のように空定義のファイルが作成される。

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
      ">

  </beans>

|

MyBatis3用のブランクプロジェクトを作成した場合のtodo-infra.xml
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

MyBatis3用のブランクプロジェクトを作成した場合、以下のような設定となっている。

.. code-block:: xml
  :emphasize-lines: 10-11, 13-15, 16-17, 18-19, 22-24

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
          http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd
      ">

      <!-- (1) -->
      <import resource="classpath:/META-INF/spring/todo-env.xml" />

      <!-- (2) -->
      <!-- define the SqlSessionFactory -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <!-- (3) -->
          <property name="dataSource" ref="dataSource" />
          <!-- (4) -->
          <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
      </bean>

      <!-- (5) -->
      <!-- scan for Mappers -->
      <mybatis:scan base-package="com.example.todo.domain.repository" />

  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 環境依存するコンポーネント(データソースやトランザクションマネージャなど)を定義するBean定義ファイルをimportする。
  * - | (2)
    - | \ ``SqlSessionFactory``\ を生成するためのコンポーネントとして、\ ``SqlSessionFactoryBean``\ をbean定義する。
  * - | (3)
    - | \ ``dataSource``\ プロパティに、設定済みのデータソースのbeanを指定する。
      |
      | MyBatis3の処理の中でSQLを発行する際は、ここで指定したデータソースからコネクションが取得される。
  * - | (4)
    - | \ ``configLocation``\ プロパティに、MyBatis設定ファイルのパスを指定する。
      |
      | ここで指定したファイルは\ ``SqlSessionFactory``\ を生成する時に読み込まれる。
  * - | (5)
    - | Mapperインタフェースをスキャンするために\ ``<mybatis:scan>``\ を定義し、\ ``base-package``\ 属性には、
      | Mapperインタフェースが格納されている基底パッケージを指定する。
      |
      | 指定されたパッケージ配下に格納されている Mapperインタフェースがスキャンされ、
      | スレッドセーフなMapperオブジェクト(MapperインタフェースのProxyオブジェクト)が自動的に生成される。

.. note::

  \ :file:`mybatis-config.xml`\ は、MyBatis3自体の動作設定を行う設定ファイルである。

  ブランクプロジェクトでは、デフォルトで以下の設定が行われている。

    .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
      <configuration>

          <!-- See https://mybatis.org/mybatis-3/configuration.html#settings -->
          <settings>
              <setting name="mapUnderscoreToCamelCase" value="true" />
              <setting name="lazyLoadingEnabled" value="true" />
              <setting name="defaultFetchSize" value="100" />
      <!--
              <setting name="defaultExecutorType" value="REUSE" />
              <setting name="jdbcTypeForNull" value="NULL" />
              <setting name="localCacheScope" value="STATEMENT" />
      -->
          </settings>

          <typeAliases>
              <package name="com.example.todo.domain.model" />
              <package name="com.example.todo.domain.repository" />
      <!--
              <package name="com.example.todo.infra.mybatis.typehandler" />
      -->
          </typeAliases>

          <typeHandlers>
      <!--
              <package name="com.example.todo.infra.mybatis.typehandler" />
      -->
          </typeHandlers>

      </configuration>

|

|

todo-infra.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.properties`\ には、Todoアプリのインフラストラクチャ層における環境依存値の設定を行う。

O/R Mapperに依存しないブランクプロジェクトを作成した際は、\ :file:`todo-infra.properties`\ は作成されない。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\ は、以下のような設定となっている。

.. code-block:: properties
  :emphasize-lines: 1, 7

  # (1)
  database=H2
  database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1
  database.username=sa
  database.password=
  database.driverClassName=org.h2.Driver
  # (2)
  # connection pool
  cp.maxActive=96
  cp.maxIdle=16
  cp.minIdle=0
  cp.maxWait=60000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90


  * - 項番
    - 説明
  * - | (1)
    - | データベースに関する設定を行う。
      | 本チュートリアルでは、データベースのセットアップの手間を省くため、H2 Databaseを使用する。
  * - | (2)
    - | コネクションプールに関する設定。

.. note::

  これらの設定値は、\ :file:`todo-env.xml`\ から参照されている。

|

todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-env.xml`\ には、デプロイする環境によって設定が異なるコンポーネントの設定を行う。

作成したブランクプロジェクトの\ ``src/main/resources/META-INF/spring/todo-env.xml``\ は、以下のような設定となっている。

| ここでは、MyBatis3用のブランクプロジェクトに格納されるファイルを例に説明する。
| なお、データベースにアクセスしないブランクプロジェクトを作成した際は、\ :file:`todo-env.xml`\ は作成されない。

.. code-block:: xml
  :emphasize-lines: 12, 27, 30, 35

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:jdbc="http://www.springframework.org/schema/jdbc"
      xsi:schemaLocation="
          http://www.springframework.org/schema/jdbc https://www.springframework.org/schema/jdbc/spring-jdbc.xsd
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
      ">

      <bean id="dateFactory" class="org.terasoluna.gfw.common.time.DefaultClockFactory" />

      <!-- (1) -->
      <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
          destroy-method="close">
          <property name="driverClassName" value="${database.driverClassName}" />
          <property name="url" value="${database.url}" />
          <property name="username" value="${database.username}" />
          <property name="password" value="${database.password}" />
          <property name="defaultAutoCommit" value="false" />
          <property name="maxTotal" value="${cp.maxActive}" />
          <property name="maxIdle" value="${cp.maxIdle}" />
          <property name="minIdle" value="${cp.minIdle}" />
          <property name="maxWaitMillis" value="${cp.maxWait}" />
      </bean>


      <!-- (2) -->
      <jdbc:initialize-database data-source="dataSource"
          ignore-failures="ALL">
          <!-- (3) -->
          <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
          <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
      </jdbc:initialize-database>

      <!-- (4) -->
      <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource" />
          <property name="rollbackOnCommitFailure" value="true" />
      </bean>
  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 実データソースの設定。
  * - | (2)
    - | データベース初期化の設定。
      | データベースを初期化するSQLファイルを実行するための設定を行っている。
      |
      | この設定は通常、開発中のみでしか使用しない(環境に依存する設定)ため、\ ``todo-env.xml``\ に定義されている。
  * - | (3)
    - | データベースを初期化するSQLファイルの設定。
      | データベースを初期化するための、DDL文が記載されているSQLファイルとDML文が記載されているSQLファイルを指定している。
      |
      | ブランクプロジェクトの設定では\ ``todo-infra.properties``\ に\ ``database=H2``\ と定義されているため、\ ``H2-schema.sql``\ 及び\ ``H2-dataload.sql``\ が実行される。
  * - | (4)
    - | トランザクションマネージャの設定。
      | id属性には、\ ``transactionManager``\ を指定する。
      | 別の名前を指定する場合は、\ ``<tx:annotation-driven>``\ タグにもトランザクションマネージャ名を指定する必要がある。
      |
      | ブランクプロジェクトでは、JDBCのAPIを使用してトランザクションを制御するクラス(\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\ )が設定されている。

|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`spring-mvc.xml`\ には、Spring MVCに関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\ は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

.. code-block:: xml
  :emphasize-lines: 15, 19, 31, 34, 40, 62, 73

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:mvc="http://www.springframework.org/schema/mvc"
      xmlns:util="http://www.springframework.org/schema/util"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd
          http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
      ">

      <!-- (1) -->
      <context:property-placeholder
          location="classpath*:/META-INF/spring/*.properties" />

      <!-- (2) -->
      <mvc:annotation-driven>
          <mvc:argument-resolvers>
              <bean
                  class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
              <bean
                  class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
          </mvc:argument-resolvers>
      </mvc:annotation-driven>

      <mvc:default-servlet-handler />

      <!-- (3) -->
      <context:component-scan base-package="com.example.todo.app" />

      <!-- (4) -->
      <mvc:resources mapping="/resources/**"
          location="/resources/,classpath:META-INF/resources/"
          cache-period="#{60 * 60}" />

      <mvc:interceptors>
          <!-- (5) -->
          <mvc:interceptor>
              <mvc:mapping path="/**" />
              <mvc:exclude-mapping path="/resources/**" />
              <bean
                  class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
          </mvc:interceptor>
          <mvc:interceptor>
              <mvc:mapping path="/**" />
              <mvc:exclude-mapping path="/resources/**" />
              <bean
                  class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
          </mvc:interceptor>
          <mvc:interceptor>
              <mvc:mapping path="/**" />
              <mvc:exclude-mapping path="/resources/**" />
              <bean class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
                  <property name="codeListIdPattern" value="CL_.+" />
              </bean>
          </mvc:interceptor>
      </mvc:interceptors>

      <!-- (6) -->
      <!-- Settings View Resolver. -->
      <mvc:view-resolvers>
          <bean class="org.thymeleaf.spring6.view.ThymeleafViewResolver">
              <property name="templateEngine" ref="templateEngine" />
              <property name="characterEncoding" value="UTF-8" />
              <property name="forceContentType" value="true" />
              <property name="contentType" value="text/html;charset=UTF-8" />
          </bean>
      </mvc:view-resolvers>

      <!-- (7) -->
      <!-- TemplateResolver. -->
      <bean id="templateResolver"
                class="org.thymeleaf.spring6.templateresolver.SpringResourceTemplateResolver">
          <property name="prefix" value="/WEB-INF/views/" />
          <property name="suffix" value=".html" />
          <property name="templateMode" value="HTML" />
          <property name="characterEncoding" value="UTF-8" />
      </bean>

      <!-- TemplateEngine. -->
      <bean id="templateEngine" class="org.thymeleaf.spring6.SpringTemplateEngine">
          <property name="templateResolver" ref="templateResolver" />
          <property name="enableSpringELCompiler" value="true" />
          <property name="additionalDialects">
              <set>
                  <bean class="org.thymeleaf.extras.springsecurity6.dialect.SpringSecurityDialect" />
              </set>
          </property>
      </bean>

      <bean id="requestDataValueProcessor"
          class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
          <constructor-arg>
              <util:list>
                  <bean
                      class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" />
                  <bean
                      class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
              </util:list>
          </constructor-arg>
      </bean>

      <!-- Setting Exception Handling. -->
      <!-- Exception Resolver. -->
      <bean id="systemExceptionResolver"
          class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
          <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
          <!-- Setting and Customization by project. -->
          <property name="order" value="3" />
          <property name="exceptionMappings">
              <map>
                  <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                  <entry key="BusinessException" value="common/error/businessError" />
                  <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                  <entry key=".DataAccessException" value="common/error/dataAccessError" />
              </map>
          </property>
          <property name="statusCodes">
              <map>
                  <entry key="common/error/resourceNotFoundError" value="404" />
                  <entry key="common/error/businessError" value="409" />
                  <entry key="common/error/transactionTokenError" value="409" />
                  <entry key="common/error/dataAccessError" value="500" />
              </map>
          </property>
          <property name="excludedExceptions">
              <array>
              </array>
          </property>
          <property name="defaultErrorView" value="common/error/systemError" />
          <property name="defaultStatusCode" value="500" />
      </bean>
      <!-- Setting AOP. -->
      <bean id="handlerExceptionResolverLoggingInterceptor"
          class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
          <property name="exceptionLogger" ref="exceptionLogger" />
      </bean>
      <aop:config>
          <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
              pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
      </aop:config>

  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | プロパティファイルの読み込み設定を行う。
      | src/main/resources/META-INF/spring直下の任意のプロパティファイルを読み込む。
      | この設定により、プロパティファイルの値をBean定義ファイル内で\ ``${propertyName}``\ 形式で埋め込んだり、Javaクラスに\ ``@Value("${propertyName}")``\ でインジェクションすることができる。
  * - | (2)
    - | Spring MVCのアノテーションベースのデフォルト設定を行う。
  * - | (3)
    - | アプリケーション層のクラスを管理するcom.example.todo.appパッケージ配下をcomponent-scan対象とする。
  * - | (4)
    - | 静的リソース(css, images, jsなど)アクセスのための設定を行う。

      | \ ``mapping``\ 属性にURLのパスを、\ ``location``\ 属性に物理的なパスの設定を行う。
      | この設定の場合\ ``<contextPath>/resources/app/css/styles.css``\ に対してリクエストが来た場合、\ ``WEB-INF/resources/app/css/styles.css``\ を探し、見つからなければクラスパス上(\ ``src/main/resources``\ やjar内)の\ ``META-INF/resources/app/css/styles.css``\ を探す。
      | どこにも\ ``styles.css``\ が格納されていない場合は、404エラーを返す。

      | ここでは\ ``cache-period``\ 属性で静的リソースのキャッシュ時間(3600秒=60分)も設定している。
      | \ ``cache-period="3600"``\ と設定しても良いが、60分であることを明示するために\ `SpEL <https://docs.spring.io/spring-framework/docs/6.0.3/reference/html/core.html#expressions-beandef-xml-based>`_\ を使用して \ ``cache-period="#{60 * 60}"``\ と書く方が分かりやすい。
  * - | (5)
    - | コントローラ処理のTraceログを出力するインターセプタを設定する。
      | \ ``/resources``\ 配下を除く任意のパスに適用されるように設定する。
  * - | (6)
    - | \ ``ViewResolver``\ の設定を行う。
      | 画面のレンダリングをThymeleafに委譲し、\ ``forceContentType``\属性により\ ``contentType``\属性に指定したコンテンツタイプ（\ ``text/html;charset=UTF-8``\）をレスポンスに設定している。
  * - | (7)
    - | \ ``TemplateResolver``\ の設定を行う。
      | この設定により、例えばコントローラからview名として\ ``hello``\が返却された場合には\ ``/WEB-INF/views/hello.html``\ がテンプレートとして処理される。

|

spring-security.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`spring-security.xml`\ には、Spring Securityに関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/spring-security.xml`\ は、以下のような設定となっている。
| なお、本チュートリアルではSpring Securityの設定ファイルの説明は割愛する。Spring Securityの設定ファイルについては、「\ :doc:`./TutorialSecurity`\ 」を参照されたい。

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:sec="http://www.springframework.org/schema/security"
      xsi:schemaLocation="
          http://www.springframework.org/schema/security https://www.springframework.org/schema/security/spring-security.xsd
          http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
      ">

      <sec:http pattern="/resources/**" request-matcher="ant" security="none"/>
      <sec:http request-matcher="ant">
          <sec:form-login/>
          <sec:logout/>
          <sec:access-denied-handler ref="accessDeniedHandler"/>
          <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
          <sec:session-management />
          <sec:intercept-url pattern="/**" access="permitAll" />
      </sec:http>

      <sec:authentication-manager />

      <!-- CSRF Protection -->
      <bean id="accessDeniedHandler"
          class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
          <constructor-arg index="0">
              <map>
                  <entry
                      key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                      <bean
                          class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                          <property name="errorPage"
                              value="/common/error/invalidCsrfTokenError" />
                      </bean>
                  </entry>
                  <entry
                      key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                      <bean
                          class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                          <property name="errorPage"
                              value="/common/error/missingCsrfTokenError" />
                      </bean>
                  </entry>
              </map>
          </constructor-arg>
          <constructor-arg index="1">
              <bean
                  class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                  <property name="errorPage"
                      value="/common/error/accessDeniedError" />
              </bean>
          </constructor-arg>
      </bean>

      <bean id="webSecurityExpressionHandler" class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler" />

      <!-- Put UserID into MDC -->
      <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
      </bean>

  </beans>

|

logback.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :file:`logback.xml`\ には、ログ出力に関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/logback.xml`\ は、以下のような設定となっている。
| なお、チュートリアルで使用しないログ設定についての説明は割愛する。

.. code-block:: xml
  :emphasize-lines: 5, 37, 50

  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE configuration>
  <configuration>

      <!-- (1) -->
      <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%replace(%msg){'(\r\n|\r|\n)','$1  '}%n%replace(%replace(%xEx){'(\r\n|\r|\n)','$1  '}){'  $',''}%nopex]]></pattern>
          </encoder>
      </appender>

      <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${app.log.dir:-log}/todo-application.log</file>
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/todo-application-%d{yyyyMMdd}.log</fileNamePattern>
              <maxHistory>7</maxHistory>
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%replace(%msg){'(\r\n|\r|\n)','$1  '}%n%replace(%replace(%xEx){'(\r\n|\r|\n)','$1  '}){'  $',''}%nopex]]></pattern>
          </encoder>
      </appender>

      <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${app.log.dir:-log}/todo-monitoring.log</file>
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/todo-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
              <maxHistory>7</maxHistory>
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%replace(%msg){'(\r\n|\r|\n)','$1  '}%n%replace(%replace(%xEx){'(\r\n|\r|\n)','$1  '}){'  $',''}%nopex]]></pattern>
          </encoder>
      </appender>

      <!-- Application Loggers -->
      <!-- (2) -->
      <logger name="com.example.todo">
          <level value="debug" />
      </logger>

      <logger name="com.example.todo.domain.repository">
          <level value="trace" />
      </logger>

      <!-- TERASOLUNA -->
      <logger name="org.terasoluna.gfw">
          <level value="info" />
      </logger>
      <!-- (3) -->
      <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
          <level value="trace" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger">
          <level value="info" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false">
          <level value="error" />
          <appender-ref ref="MONITORING_LOG_FILE" />
      </logger>

      <!-- 3rdparty Loggers -->
      <logger name="org.springframework">
          <level value="warn" />
      </logger>

      <logger name="org.springframework.web.servlet">
          <level value="info" />
      </logger>

      <logger name="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
          <level value="trace" />
      </logger>

      <logger name="org.springframework.jdbc.core.JdbcTemplate">
          <level value="trace" />
      </logger>

      <!--  REMOVE THIS LINE IF YOU USE MyBatis3
      <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <level value="debug" />
      </logger>
            REMOVE THIS LINE IF YOU USE MyBatis3  -->

      <root level="warn">
          <appender-ref ref="STDOUT" />
          <appender-ref ref="APPLICATION_LOG_FILE" />
      </root>

  </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 標準出力でログを出力するアペンダを設定。
  * - | (2)
    - | com.example.todoパッケージ以下はdebugレベル以上を出力するように設定。
  * - | (3)
    - | spring-mvc.xmlに設定した\ ``TraceLoggingInterceptor``\ に出力されるようにtraceレベルで設定。

.. note::

  MyBatis3を使用するブランクプロジェクトを作成した場合は、トランザクション制御関連のログを出力するロガーが有効な状態となっている。

  * MyBatis3用のブランクプロジェクト

    .. code-block:: xml

      <logger name="com.example.todo">
          <level value="debug" />
      </logger>

      <logger name="com.example.todo.domain.repository">
          <level value="trace" />
      </logger>

      <!-- omitted -->

      <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <level value="debug" />
      </logger>

.. raw:: latex

  \newpage
