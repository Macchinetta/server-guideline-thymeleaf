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

<<<<<<< HEAD
    * - 20xx-xx-xx
=======
    * - 2019-03-08
>>>>>>> Release version 1.6.0.RELEASE
      - \-
      - 1.6.0 RELEASE版公開

    * -
<<<<<<< HEAD
      - Thymeleaf対応
      - 以下のThymeleaf対応章を追加

        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Thymeleaf`
=======
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        記載内容の改善

        記載内容の修正・追加

        * ViewResolverの定義について、Spring 4.0以前からの\ ``<bean>``\要素を使用した定義方法を削除し、Spring 4.1以降の\ ``<mvc:view-resolvers>``\要素を使用した定義方法のみ解説するよう変更

    * -
      - Thymeleaf対応
      - 以下のThymeleaf対応章を追加

>>>>>>> Release version 1.6.0.RELEASE
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

<<<<<<< HEAD
    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        記載内容の改善

        記載内容の修正・追加

        * ViewResolverの定義について、Spring 4.0以前からの\ ``<bean>``\要素を使用した定義方法を削除し、Spring 4.1以降の\ ``<mvc:view-resolvers>``\要素を使用した定義方法のみ解説するよう変更

        * \ ``SPRING_SECURITY_LAST_EXCEPTION`` \ が格納されるスコープの説明を修正

    * -
=======
        記載内容の修正・追加

        * Decoupled Template Logicの適用方法についての記述を追加
        * JavaScriptのテンプレート化についての記述を追加
        * テンプレートHTMLのデバッグについての記述を追加
        * フレームワークスタックに\ ``thymeleaf-extras-java8time``\を追加

    * -
      - :doc:`../Introduction/CriteriaBasedMapping`
      - OWASP Top 10 を2013版から2017版へ変更

        * OWASP(Open Web Application Security Project)による観点の更新

    * -
      - :doc:`../Overview/FrameworkStack`
      - 利用するOSSのバージョンを更新

        * Spring IO PlatformのバージョンをCairo-SR3に更新

         * Spring Framework 5.0.0よりJasperReportsが非サポートとなったことへの対応
         * Spring Framework 5.0.3よりiTextが非サポートとなり、代わりにOpenPDFがサポートされたことへの対応
         * Spring Framework 5.0.0よりクエリ文字列に対するURLエンコーディングの仕様が変更されたことへの対応
         * Spring Security 5より非推奨の\ ``PasswordEncoder``\のパッケージが廃止になったことへの対応
         * Spring Security 5.0.2よりセキュリティヘッダの付与タイミングが変更された旨の説明の追加
         * Spring Security OAuth 2.2.2よりリダイレクトURIのホワイトリストチェックの仕様が変更されたことへの対応
         * Lombok 1.6.22より\ ``@Data``\と\ ``@NoArgsConstructor``\を付与する順序によってコンパイルエラーが発生するようになったことへの対応

        * MyBatisのバージョンを3.4.6に更新

        * Dozerのバージョンを6.4.1に更新

        Spring IO Platformのバージョン更新に伴い利用するOSSのバージョンを更新

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Thymeleaf`
      - Spring Framework 5.0.8対応に伴う修正

        * SpEL評価時におけるnull-safetyの影響についての注意事項を追加
      
    * -
>>>>>>> Release version 1.6.0.RELEASE
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - 構成見直し

        * Overviewを取得データの表示、ページネーションリンクの表示、ページネーション情報の表示の3点について説明するように変更

    * -
<<<<<<< HEAD
=======
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - 記載内容の追加

        * Ajax使用時にトランザクショントークンチェックを適用する方法の説明を追加

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
      - 記載内容の修正

        * \ ``SPRING_SECURITY_LAST_EXCEPTION`` \ が格納されるスコープの誤記を修正

    * -
>>>>>>> Release version 1.6.0.RELEASE
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - 記載内容の修正

        * 独自カスタマイズしたコードリストのBean定義方法を、コンポーネントスキャンからBean定義ファイルによる定義に変更

    * -
<<<<<<< HEAD
=======
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
      - Spring Framework 5.0.8対応に伴う修正

        * JasperReportsが非サポートとなったため、JasperReportsに言及している記載を修正
        * iTextの代わりにOpenPDFがサポートされるようになった旨の説明を追加し、実装例を修正

    * -
      - | :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
      - OWASP Top 10 2017対応に伴う修正

        * A8:2017に関連する、デシリアライズ時のWarningを追加
        * Macchinetta Server Framework (1.x)ではXXE対策済みのSpring MVCを使用しているため、
          XXE対策についてのWarningをNoteへ変更し、spring-oxmによる対策方法の記述を削除

    * -
      - | :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - OWASP Top 10 2017対応に伴う修正

        * Macchinetta Server Framework (1.x)ではXXE対策済みのSpring MVCを使用しているため、
          XXE対策についてのWarningをNoteへ変更し、spring-oxmによる対策方法の記述を削除

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - 記載内容の追加

        * \ ``Pageable`` \ を利用した検索結果のソートについての説明を追加

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`
      - Dozer 6.4.1対応に伴う修正

        * Dozer のバージョンアップ対応に伴い、ガイドラインに記載されているコード例を修正
        * Dozer 6.2.0において、単方向マッピングの挙動が仕様と異なっていたバグが修正されたことの説明を追加
        * Dozer 6.3.0よりJAXBがデフォルト利用されるようになったため、挙動の変更の注意点をWARNINGに追加
        
        記載内容の削除

        * 現バージョン（Dozer5.5.0以降）ではCollection<T>を使用したBean間のマッピングも可能であるため、マッピングが失敗する旨を記述したTodoを削除

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/JMS`
      - OWASP Top 10 2017対応に伴う修正

        * A8:2017に関連する、デシリアライズ時のWarningを追加


    * -
      - :doc:`../Security/Authentication`
      - OWASP Top 10 2017対応に伴う修正

        * A10:2017に関連する、ログイン認証時のログについてのTipを追加

        記載内容の修正

        * Spring Security 5より非推奨の\ ``PasswordEncoder``\のパッケージが廃止されたことに伴い、\ ``MessageDigestPasswordEncoder``\を使用する方法に記載を修正

        記載内容の改善

        * ブランクプロジェクトで定義する\ ``PasswordEncoder``\を\ ``BCryptPasswordEncoder``\から\ ``DelegatingPasswordEncoder``\に変更したことに伴う記載内容の変更

        記載内容の追加

        * \ ``SPRING_SECURITY_LAST_EXCEPTION`` \ が格納されるスコープの説明を追加

    * -
      - :doc:`../Security/CSRF`
      - OWASP Top 10 2017対応に伴う修正

        * OWASP Top 10 2013版へのリンクをOWASP Cheat Sheetへのリンクへ変更

    * -
      - :doc:`../Security/LinkageWithBrowser`
      - Spring Security 5.0.7対応に伴う修正

        * Spring Securityが付与するセキュリティヘッダの付与タイミングに関する仕様変更についての説明及び注意事項を追加

    * -
      - :doc:`../Security/OAuth`
      - Spring Security OAuth 2.2.2対応に伴う修正

        * Spring Security OAuthのバージョン更新に伴いリダイレクトURI情報を保持するテーブルへの説明にWarningを追加

    * -
>>>>>>> Release version 1.6.0.RELEASE
      - :doc:`../Tutorial/TutorialTodo`
      - 記載内容の修正・追加

        * 一覧表示機能作成時に、登録機能の一部を作成していた部分を変更し、一覧表示機能の動作確認できるように、コード例を追加

    * -
      - :doc:`../Tutorial/TutorialREST`
      - 記載内容の修正

        * spring-mvc-rest.xmlを作成する方法の説明を変更

<<<<<<< HEAD
=======
    * -
      - :doc:`../Tutorial/TutorialSecurity`
      - 記載内容の修正

        * \ ``SPRING_SECURITY_LAST_EXCEPTION`` \ が格納されるスコープの誤記を修正

    * -
      - :doc:`../Appendix/Lombok`
      - Spring IO Platform Cairo-SR3対応に伴う修正

        * Lombok 1.6.22における\ ``@Data``\と\ ``@NoArgsConstructor``\を付与する順序についてのWarningを追加

>>>>>>> Release version 1.6.0.RELEASE
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
