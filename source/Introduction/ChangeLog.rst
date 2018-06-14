更新履歴
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60
    :class: longtable

    * - 更新日付
      - 更新箇所
      - 更新内容

    * - 20xx-xx-xx
      - \-
      - 1.6.0 RELEASE版公開

    * -
      - Thymeleaf対応
      - 以下のThymeleaf対応章を追加

        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Thymeleaf`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/HealthCheck`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/DateAndTime`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
        * :doc:`../Security/OAuth`
        * :doc:`../Security/SecureLoginDemo`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../Tutorial/TutorialREST`
        * :doc:`../Tutorial/TutorialSession`
        * :doc:`../Tutorial/TutorialSecurity`

    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        記載内容の改善

        記載内容の修正・追加

        * ViewResolverの定義について、Spring 4.0以前からの\ ``<bean>``\要素を使用した定義方法を削除し、Spring 4.1以降の\ ``<mvc:view-resolvers>``\要素を使用した定義方法のみ解説するよう変更

        * \ ``SPRING_SECURITY_LAST_EXCEPTION`` \ が格納されるスコープの説明を修正

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - 構成見直し

        * Overviewを取得データの表示、ページネーションリンクの表示、ページネーション情報の表示の3点について説明するように変更

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - 記載内容の修正

        * 独自カスタマイズしたコードリストのBean定義方法を、コンポーネントスキャンからBean定義ファイルによる定義に変更

    * -
      - :doc:`../Tutorial/TutorialTodo`
      - 記載内容の修正・追加

        * 一覧表示機能作成時に、登録機能の一部を作成していた部分を変更し、一覧表示機能の動作確認できるように、コード例を追加

    * -
      - :doc:`../Tutorial/TutorialREST`
      - 記載内容の修正

        * spring-mvc-rest.xmlを作成する方法の説明を変更

    * - 2018-03-09
      - \-
      - 1.5.1 RELEASE版公開

    * - 
      - :doc:`../Overview/FrameworkStack`
      - CVE-2018-1199への対応のため、利用するOSSのバージョンを更新

        * Spring Frameworkのバージョンを4.3.14に更新
        * Spring Securityのバージョンを4.2.4に更新
    * -
      - :doc:`../Security/OAuth`
      - 記載内容の修正
  
        * 認可サーバのチェックトークンエンドポイントのURL設定が反映されない不具合へのWarningを削除

    * - 2017-12-22
      - \-
      - 1.5.0 RELEASE版公開

.. raw:: latex

   \newpage
