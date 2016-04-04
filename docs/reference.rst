=====================================
Selenium2Library for Robot Framework 
=====================================

:Library-version: 1.7.4

.. note::

  このドキュメントは、 `Selenium2Library のドキュメント <http://robotframework.org/Selenium2Library/doc/Selenium2Library.html>`_ を和訳したものです。

  オリジナルのドキュメントでは、スクリプトは状況に応じてタブ区切り (TSV) またはパイプ区切り (PSV) で記述されています。
  このドキュメントでは、可能な限りプレーンテキスト・スペース 2 個の区切りで記述しています。
  RobotFramework がサポートする書式については、 `Robot Framework のユーザガイド <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html>`_ を参照してください。


はじめに
=========

Selenium2Library は、 Robot Framework 用の Web テストライブラリです。

このライブラリは内部で Selenium 2 (WebDriver) ライブラリを使って Web ブラウザを操作します。Selenium2 と WebDriver については http://seleniumhq.org/docs/03_webdriver.html を参照してください。


テストを実行する前に
=====================

Selenium2Library ベースのテストケースを実行するには、あらかじめ Selenium2Library を Robot のテストスイートに組み込んでおきます。また、ブラウザを開くには、 `Open Browser`_ キーワードを使います。


.. _`locating elements`:

エレメントの探索と指定
=======================

Selenium2Library のキーワードのうち、ページ上のエレメントを検索する必要があるものは、全て ``locator`` または ``webelement`` という引数をとります。``locator`` には、エレメントの場所を指定するための文字列を指定します。指定方法は種々あります。
``webelement`` には、特定のエレメントを表す ``WebElement`` インスタンスの入った変数を指定します。


``locator`` の使い方
----------------------

デフォルトの動作では、 ``locator`` を指定した場合、特定のエレメントタイプのキーアトリビュートに対してマッチングしてエレメントを特定します。例えば、 ``id`` や ``name`` は全てのエレメントのキーアトリビュートであり、 ``locator`` を簡単に指定するには ``id`` だけを使います。

.. code:: robotframework

  Click Element  my_element

ロケータの文字列にプレフィクスをつけると、 Selenium2Library が ``locator`` を使ってエレメントを探索するときの方法（ロケーションストラテジ）を指定できます。サポートしているストラテジは以下です:

.. table::

  ================= ================================================ =========================================
  ストラテジ        記述例                                           説明
  ================= ================================================ =========================================
  ``identifier``    ``Click Element  identifier=my_element``         @id または @name 属性を指定
  ``id``            ``Click Element  id=my_element``                 @id 属性の指定
  ``name``          ``Click Element  name=my_element``               @name 属性の指定
  ``xpath``         ``Click Element  xpath=//div[@id='my_element']`` 任意の XPath 式
  ``dom``           ``Click Element  dom=document.images[56]``       任意の DOM 式
  ``link``          ``Click Element  link=My Link``                  ``<a>`` 要素のリンクテキスト
  ``partial link``  ``Click Element  partial link=My Link``          ``<a>`` 要素のリンクテキスト（部分一致）
  ``css``           ``Click Element  css=div.my_class``              CSS セレクタ
  ``jquery``        ``Click Element  jquery=div.my_class``           jQuery/sizzle セレクタ
  ``sizzle``        ``Click Element  sizzle=div.my_class``           jQuery/sizzle セレクタ
  ``tag``           ``Click Element  tag=div``                       HTMLタグ名
  ``default`` [1]_  ``Click Link  default=page?a=b``                 タグ中の最初のキー属性の値が一致するもの
  ================= ================================================ =========================================

..  [1] default ストラテジが必要なのは、キー属性を使ったマッチングが必要で、
  かつその属性の値に ``=`` が入っている場合だけです。
  例えば、以下のコードは、 ``page?a`` というストラテジを指定しているように読めてしまうため失敗します:

  .. code:: robotframework

     Click Link  page?a=b

  このような場合に、 default ストラテジを使います:

  .. code:: robotframework

    Click Link  default=page?a=b


``webelements`` の使い方
------------------------

Selenium2Library バージョン 1.7 から、ロケータの代わりに WebElement で指定できるようになりました。WebElement を指定するには、まず `Get WebElements`_ キーワードを使って取得します:

.. code:: robotframework

  ${elem} =  Get WebElement  id=my_element
  Click Element  ${elem}


.. _`locating table`:

テーブル、行、列などの指定方法
-------------------------------

``Table Should Contain`` キーワードのように、テーブルに関係するキーワードは、一風変わった動作をします。デフォルトの動作では、テーブルのロケータが指定されていると、id 属性が一致するテーブルを探します:

.. code:: robotframework

  Table Should Contain  my_table text

この他に、より複雑なテーブル検索ストラテジもサポートしています:

.. table::

  =========== ================================================================ ==============================
  ストラテジ  記述例                                                           説明
  =========== ================================================================ ==============================
  ``css``     ``Table Should Contain  css=table.my_class text``                @id または @name 属性でマッチ
  ``xpath``   ``Table Should Contain  xpath=//table/[@name="my_table"] text``  @id または @name 属性でマッチ
  =========== ================================================================ ==============================


ロケータストラテジを自作する
-----------------------------

組み込みのロケータよりも複雑な検索を実行する必要がある場合は、ロケータストラテジを自作できます。
カスタムロケータを使うには、二つのステップが必要です。まず、 ``Custom Locator Strategy`` という名前で、 WebElement を返すようなキーワードを定義します:

.. code:: robotframework

  Custom Locator Strategy
    [Arguments]  ${browser}  ${criteria}  ${tag}  ${constraints}
    [Documentation]  getElementById() を使ってエレメントを探す
    ${retval} =  Execute Javascript  return window.document.getElementById("${criteria}");
    [Return]  ${retval}

上のキーワード定義は、 id を使ったロケータの再実装です。${browser} には WebDriver インスタンスへの参照が、 ${criteria} にはロケータに指定した文字列 (``=`` の後に指定した文字列全体) が入ります。

定義したロケータを使うには、 `Add Location Strategy`_ を使います。

.. code::

  Add Location Strategy  custom  Custom Locator Strategy

Add Location Strategy の最初の引数には、このストラテジを使う時の名前を (ストラテジ間で重複しないように) 指定します。ストラテジを登録できたら、使い方は他のロケータと同じです。`Add Location Strategy`_ のセクションも参照してください。


.. _`Timeouts`:

タイムアウトの設定
===================

``Wait...`` で始まるキーワードで、引数に ``timeout`` を取るものがあります。``timeout`` の指定は省略でき、省略した際のタイムアウト値は、 `Set Selenium Timeout`_ でグローバルに設定した値に従います。``timeout`` は、 `Execute Async Javascript`_ でも使います。

タイムアウトの値は、秒数を表す数値 (``0.5``, ``42`` など) か、Robot Framework での時間の表記法 (``1.5 seconds``, ``1 min 30 s``) で指定できます。後者の表記法の詳細は http://robotframework.googlecode.com/svn/trunk/doc/userguide/RobotFrameworkUserGuide.html#time-format を参照してください。


Selenium2Library のインポート引数
==================================

``Library Selenium2Library`` でライブラリをインポートするとき、引数を指定できます。

Library Selenium2Library
---------------------------

:Arguments: ``imeout=5.0, implicit_wait=0.0, run_on_failure=Capture Page Screenshot, screenshot_root_directory=None``

``timeout`` はデフォルトのタイムアウト値です。この値は Selenium2Library が待機処理するときのタイムアウト値を一括で設定します。`Set Selenium Timeout`_ でも設定できます。

``implicit wait`` は Selenium がエレメントを探す際の暗黙のタイムアウト (implicit wait) 値です。 `Set Selenium Implicit Wait`_ でも指定できます。WebDriver の implicit wait 値について詳しく知りたければ、 SeleniumHQ のドキュメントで `WebDriver: Advanced Usage`_ セクションを参照してください。

.. _`WebDriver: Advanced Usage`: http://seleniumhq.org/docs/04_webdriver_advanced.html#explicit-and-implicit-waits

``run_on_failure`` には、テスト中に Selenium2Library のキーワードの実行が失敗したときに呼び出されるキーワードを指定します (インポート済みで使えるキーワードを指定できます)。デフォルトの設定では、 `Capture Page Screenshot`_ を使って当該ページのスクリーンショットを保存します。``Nothing`` を指定すれば、この機能を無効にできます。`Register Keyword To Run On Failure`_ も参照してください。

``screenshot_root_directory`` には、スクリーンショットの保存先ディレクトリを指定します。指定がなければ、Robot Framework がログファイルを置く場所と同じになります。

使用例:

.. code:: robotframework

  # デフォルトタイムアウトを15秒にする
  Library  Selenium2Library  15
  # デフォルトタイムアウトは0秒で、 ``implicit_wait`` は5秒
  Library  Selenium2Library  0  5
  # タイムアウトを5秒にして、失敗したら ``Log Source`` を呼び出す
  Library  Selenium2Library  5  run_on_failure=Log Source
  # ``implicit_wait`` を5秒にし、失敗したら ``Log Source`` を呼び出す
  Library  Selenium2Library  implicit_wait=5  run_on_failure=Log Source
  # タイムアウト10秒、失敗しても何もしない
  Library Selenium2Library timeout=10 run_on_failure=Nothing


組み込みキーワード
========================


.. table::

  ============================================= ===================================================================
  キーワード                                    説明
  ============================================= ===================================================================
  `Create Webdriver`_                           WebDriverインスタンスを生成する
  `Open Browser`_                               新しくブラウザウィンドウを開く
  `Go To`_                                      URLを指定する
  `Go Back`_                                    戻るボタンを押す
  `Reload Page`_                                ページをリロードする
  `Select Window`_                              ウィンドウを切り替える
  `Set Window Position`_                        ウィンドウ位置を変更する
  `Set Window Size`_                            ウィンドウサイズを変更する
  `Maximize Browser Window`_                    ブラウザウィンドウを最大化する
  `Select Frame`_                               フレームを切り替える
  `Switch Browser`_                             ブラウザを切り替える
  `Unselect Frame`_                             フレームの選択を解除する
  `List Windows`_                               ウィンドウのリストを取り出す
  `Close Window`_                               ポップアップウィンドウを閉じる
  `Close Browser`_                              現在のブラウザを閉じる
  `Close All Browsers`_                         全てのブラウザを閉じる
  `Get Window Identifiers`_                     開いている全ウィンドウの識別子を調べる
  `Get Window Names`_                           開いている全ウィンドウのウィンドウ名を調べる
  `Get Window Position`_                        ウィンドウの位置を調べる
  `Get Window Size`_                            ウィンドウのサイズを調べる
  `Get Window Titles`_                          ウィンドウのタイトルを調べる
  `Get Location`_                               現在のURLを調べる
  `Location Should Be`_                         URLが指定通りか確認する
  `Location Should Contain`_                    URLに指定の値が含まれるか確認する
  `Get Source`_                                 ページのソースを調べる
  `Get Title`_                                  ページのタイトルを調べる
  `Title Should Be`_                            タイトルが指定通りか確認する
  `Get Text`_                                   エレメントのテキストを調べる
  `Get Value`_                                  エレメントのvalueを調べる
  `Get Webelement`_                             エレメントを WebElement として取り出す
  `Get Webelements`_                            ページの全エレメントを WebElement として取り出す
  `Get Element Attribute`_                      エレメントの属性を調べる
  `Get Vertical Position`_                      エレメントの垂直位置を調べる
  `Get Horizontal Position`_                    エレメントの水平位置を調べる
  `Get All Links`_                              ページ中の全てのリンクを調べる

  `Set Screenshot Directory`_                   スクリーンショットの出力先を変更する
  `Capture Page Screenshot`_                    ページのスクリーンショットを取る

  `Add Cookie`_                                 クッキーを追加する
  `Get Cookie Value`_                           クッキーの値を調べる
  `Get Cookies`_                                クッキーを全て取り出す
  `Delete Cookie`_                              特定のクッキーを削除する
  `Delete All Cookies`_                         全てのクッキーを削除する

  `Page Should Contain`_                        ページが指定文字列を含むか確認する
  `Page Should Contain Button`_                 ページに指定のボタンがあるか確認する
  `Page Should Contain Checkbox`_               ページに指定のチェックボックスがあるか確認する
  `Page Should Contain Element`_                ページに指定のエレメントがあるか確認する
  `Page Should Contain Image`_                  ページに指定の画像があるか確認する
  `Page Should Contain Link`_                   ページに指定のリンクがあるか確認する
  `Page Should Contain List`_                   ページに指定のリストがあるか確認する
  `Page Should Contain Radio Button`_           ページに指定のラジオボタンがあるか確認する
  `Page Should Contain Textfield`_              ページに指定のテキスト入力があるか確認する
  `Page Should Not Contain`_                    ページに指定の文字列がないことを確認する
  `Page Should Not Contain Button`_             ページに指定のボタンがあるか確認する
  `Page Should Not Contain Checkbox`_           ページに指定のチェックボックスがあるか確認する
  `Page Should Not Contain Element`_            ページに指定のエレメントがあるか確認する
  `Page Should Not Contain Image`_              ページに指定の画像があるか確認する
  `Page Should Not Contain Link`_               ページに指定のリンクがあるか確認する
  `Page Should Not Contain List`_               ページに指定のリストがあるか確認する
  `Page Should Not Contain Radio Button`_       ページに指定のラジオボタンがないことを確認する
  `Page Should Not Contain Textfield`_          指定のテキスト入力がないことを確認する
  `Frame Should Contain`_                       フレームに指定文字列があるか確認する
  `Current Frame Contains`_                     現在のフレームに指定文字列があるか確認する
  `Current Frame Should Not Contain`_           現在のフレームが指定文字列を含まないことを確認する
  `Get Matching Xpath Count`_                   指定のXPathにマッチした回数を調べる
  `Xpath Should Match X Times`_                 XPathにマッチするエレメントの個数が指定通りか確認する
  `Locator Should Match X Times`_               エレメントが指定個数入っているか書くにする

  `Element Should Be Disabled`_                 エレメントが無効か確認する
  `Element Should Be Enabled`_                  エレメントが有効か確認する
  `Element Should Be Visible`_                  エレメントが可視か確認する
  `Element Should Contain`_                     エレメントのテキストに指定文字列があるか確認する
  `Element Should Not Be Visible`_              エレメントが不可視であることを確認する
  `Element Should Not Contain`_                 エレメントのテキストが指定文字列が含まないことを確認する
  `Element Text Should Be`_                     エレメントのテキストが指定文字列と一致するか確認する

  `Get Table Cell`_                             テーブルの指定のセルの中身を調べる
  `Table Cell Should Contain`_                  テーブルのセルが指定の文字列を含むか確認する
  `Table Column Should Contain`_                テーブルのカラムが指定の文字列を含むか確認する
  `Table Footer Should Contain`_                テーブルのフッタが指定の文字列を含むか確認する
  `Table Header Should Contain`_                テーブルのヘッダが指定の文字列を含むか確認する
  `Table Row Should Contain`_                   テーブルの行が指定の文字列を含むか確認する
  `Table Should Contain`_                       テーブルが指定の文字列を含むか確認する

  `Click Button`_                               ボタンをクリックする
  `Click Element`_                              任意のエレメントをクリックする
  `Click Element At Coordinates`_               エレメントの指定の場所をクリックする
  `Click Image`_                                画像をクリックする
  `Click Link`_                                 リンクをクリックする
  `Focus`_                                      ウィンドウやフレームをフォーカスする
  `Mouse Down`_                                 エレメント上で左ボタンを押した状態にする
  `Mouse Down On Image`_                        画像上で左ボタンを押した状態にする
  `Mouse Down On Link`_                         リンク上で左ボタンを押した状態にする
  `Mouse Out`_                                  エレメントからマウスカーソルを外す
  `Mouse Over`_                                 エレメントにマウスカーソルを重ねる
  `Mouse Up`_                                   押していた左ボタンをリリースする
  `Double Click Element`_                       任意のエレメントをダブルクリックする
  `Drag And Drop`_                              エレメントを別のエレメントにドラッグ＆ドロップする
  `Drag And Drop By Offset`_                    エレメントを指定の場所にドラッグ＆ドロップする
  `Press Key`_                                  キーを押す
  `Open Context Menu`_                          コンテキストメニューを開く

  `Alert Should Be Present`_                    アラートが表示されたか確認する
  `Choose Ok On Next Confirmation`_             次に表示されるダイアログでOKを押す
  `Choose Cancel On Next Confirmation`_         次に表示されるダイアログでキャンセルを押す
  `Confirm Action`_                             ダイアログのメッセージを取得して閉じる
  `Dismiss Alert`_                              アラートダイアログを閉じて押したボタンを返す
  `Get Alert Message`_                          アラートダイアログのメッセージを調べる
  `Input Text Into Prompt`_                     アラートダイアログにテキストを入力する

  `Input Text`_                                 テキスト入力に入力する
  `Input Password`_                             ログに記録しないでパスワードを入力する
  `Textarea Should Contain`_                    テキストエリアのテキストが指定の文字列を含むか確認する
  `Textarea Value Should Be`_                   テキストエリアの値が指定通りか確認する
  `Textfield Should Contain`_                   テキストフィールドのテキストが指定の文字列を含むか確認する
  `Textfield Value Should Be`_                  テキストフィールドのvalueが指定通りか確認する
  `Clear Element Text`_                         テキスト入力の値をクリアする
  `Select Radio Button`_                        ラジオボタンを選択する
  `Radio Button Should Be Set To`_              指定のラジオボタンが選ばれていることを確認する
  `Radio Button Should Not Be Selected`_        指定のラジオボタンが選ばれていないことを確認する
  `Checkbox Should Be Selected`_                チェックボックスが選択されているか確認する
  `Checkbox Should Not Be Selected`_            チェックボックスが非選択であるか確認する
  `Select All From List`_                       selectの全項目を選択する
  `Select From List`_                           selectの項目を選択する
  `Select From List By Index`_                  selectの項目を選択する
  `Select From List By Label`_                  selectの項目を選択する
  `Select From List By Value`_                  selectの項目を選択する
  `Unselect From List`_                         リストから指定の要素の選択を外す
  `Unselect From List By Index`_                リストから指定の要素の選択を外す
  `Unselect From List By Label`_                リストから指定の要素の選択を外す
  `Unselect From List By Value`_                リストから指定の要素の選択を外す
  `Get List Items`_                             selectの全選択肢を取り出す
  `Get Selected List Label`_                    selectの指定の選択肢のラベルを調べる
  `Get Selected List Labels`_                   selectの全てのラベルを取り出す
  `Get Selected List Value`_                    selectの指定の選択肢のvalueを調べる
  `Get Selected List Values`_                   selectのすべての選択肢のvalueを調べる
  `List Selection Should Be`_                   selectの選択内容が指定通りか確認する
  `List Should Have No Selections`_             selectが非選択状態であることを確認する
  `Choose File`_                                ファイルダイアログにファイルを指定する
  `Select Checkbox`_                            チェックボックスを選択する
  `Unselect Checkbox`_                          チェックボックスの選択を解除する
  `Submit Form`_                                フォームを submit する

  `Log Location`_                               現在のURLをログに記録する
  `Log Source`_                                 ページのソースをログに記録する
  `Log Title`_                                  ページのタイトルをログに記録する

  `Execute Async Javascript`_                   非同期でJavaScriptのコードを実行する
  `Execute Javascript`_                         JavaScriptのコードを実行する
  `Simulate`_                                   イベント発生をシミュレートする
  `Assign Id To Element`_                       エレメントに一時的な id を割り当てる

  `Wait For Condition`_                         指定の条件式が満たされるまで待機する
  `Wait Until Element Contains`_                エレメント内に文字列が現れるまで待機する
  `Wait Until Element Does Not Contain`_        文字列がエレメントからなくなるまで待機する
  `Wait Until Element Is Enabled`_              エレメントが有効状態になるまで待機する
  `Wait Until Element Is Not Visible`_          エレメントが不可視になるまで待機する
  `Wait Until Element Is Visible`_              エレメントが可視になるまで待機する
  `Wait Until Page Contains`_                   文字列がページに現れるまで待機する
  `Wait Until Page Contains Element`_           エレメントがページに現れるまで待機する
  `Wait Until Page Does Not Contain`_           文字列がページからなくなるまで待機する
  `Wait Until Page Does Not Contain Element`_   エレメントがページからなくなるまで待機する
  `Get Selenium Implicit Wait`_                 Selenium の暗黙の待機時間を調べる
  `Get Selenium Speed`_                         Selenium の実行ウェイトを調べる
  `Get Selenium Timeout`_                       Selenium のタイムアウトを調べる
  `Set Browser Implicit Wait`_                  ブラウザ単位で暗黙待機時間を変更する
  `Set Selenium Implicit Wait`_                 Selenium の暗黙待機時間を変更する
  `Set Selenium Speed`_                         Selenium の実行ウェイトを変更する
  `Set Selenium Timeout`_                       Selenium のタイムアウトを変更する

  `Add Location Strategy`_                      自作のエレメント特定方法を追加する
  `Remove Location Strategy`_                   以前登録したエレメントの探索ストラテジを削除する
  `Register Keyword To Run On Failure`_         失敗したときに実行するキーワードを指定する
  ============================================= ===================================================================      
   

Add Cookie
---------------

:Arguments: name, value, path=None, domain=None, secure=None, expiry=None

現在のセッションに cookie を追加します。``name`` と ``value`` は必須です。
``path``, ``domain``, ``secure`` は省略可です。


Add Location Strategy
--------------------------

:Arguments: strategy_name, strategy_keyword, persist=False

キーワードセクションに定義しておいたカスタムのロケーションストラテジを追加します。
デフォルトの動作では、カスタムのロケーションストラテジは、ストラテジを定義したテストのスコープを抜ける際に自動的に除去されます。
persist を空文字列以外に設定しておくと、テスト全体が終了するまでロケーションストラテジを登録したままにできます。

すでに登録済みのロケーションストラテジ名と同じ名前で追加を試みると失敗します。

カスタムのロケータキーワードの例:

.. code:: robotframework

  # My Custom Locator Strategy を（キーワードセクションで）定義する
  My Custom Locator Strategy  [Arguments]  ${browser}  ${criteria}  ${tag}  ${constraints}
    ${retVal}=  Execute Javascript  return window.document.getElementById('${criteria}');  
    [Return]  ${retVal}

.. code:: robotframework

  # My Custom Locator Strategy をストラテジ名 custom として登録する
  Add Location Strategy  custom  My Custom Locator Strategy
  # custom ストラテジのロケータを使う
  Page Should Contain Element  custom=my_id

カスタムのロケーションストラテジを削除する方法は `Remove Location Strategy`_ を参照してください。


Alert Should Be Present
----------------------------

:Arguments: text=

アラートダイアログが表示されていることを確認し、ダイアログを消します。
``text`` を空文字列以外に指定すると、アラートメッセージの内容がテキストと一致するか調べます。
アラートが表示されていなければ失敗します。
テスト中でアラートダイアログが表示されると、このキーワードか `Get Alert Message`_ でアラートダイアログを消さないかぎり、後続のキーワードは失敗するので注意してください。


Assign Id To Element
-------------------------

:Arguments: locator, id

``locator`` で指定したエレメントに、一時的な id 名 ``id`` を紐付けます。
XPathでロケータを表現するのが難解だったり遅い場合に便利です。紐付けた id はページをリロードすると失効します。

例:

.. code:: robotframework

  # XPath://div[@id="first_div"] で表されるエレメントに my_id という id を与える
  Assign Id To Element  xpath=//div[@id="first_div"]  my_id
  # my_id でエレメントを指定
  Page Should Contain Element  my_id


Capture Page Screenshot
----------------------------

:Arguments: filename=None

現在表示中のページのスクリーンショットを撮り、ログの中に埋め込みます。
``filename`` 引数には、スクリーンショットのファイルに付ける名前を指定します。``filename`` を指定しない場合、ファイル名は ``selenium-screenshot-<通番>.png`` となり、Robot Framework がログを書き出す場所の下に保存されます。
``filename`` を絶対パスで指定しなかった場合は、Robot Framework がログを書き出す場所からの相対パスとみなします。
絶対パス・相対パスのどちらの場合も、書き出し先のディレクトリが存在しなければ自動的に作成します。
スクリーンショットの撮り方を変えるために、css を使えます。通常、ページレイアウトに何か問題があると、背景がはみ出して表示されることがあるので、背景色を変更します。


Checkbox Should Be Selected
--------------------------------

:Arguments: locator

``locator`` で指定したチェックボックスが選択/チェック状態であるか検証します。
チェックボックスのキー属性は ``id`` または ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Checkbox Should Not Be Selected
-------------------------------------

:Arguments: locator

``locator`` で指定したチェックボックスが非選択/チェックされていない状態であるか検証します。
チェックボックスのキー属性は ``id`` または ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Choose Cancel On Next Confirmation
---------------------------------------

ダイアログが表示されたときにキャンセルボタンを選ぶように指定します。


Choose File
----------------

:Arguments: locator, file_path

``locator`` で指定したファイル入力フィールドにファイルパス ``file_path`` を入力します。
ファイルのアップロードフォームでよく使います。
``file_path`` はSelenium Server が稼働しているホストからアクセス可能でなければなりません。

例:

.. code:: robotframework

  Choose File  my_upload_field  /home/user/files/trades.csv


Choose Ok On Next Confirmation
-----------------------------------

``Choose Cancel On Next Confirmation`` の効果を打ち消します。
次の確認パネルが出たときに
Selenium は window.confirm() の動作をオーバライドすることで、自動的に true を返させ、ユーザが手作業で OK ボタンをクリックしたかのように見せかけるので、特に理由がないかぎり、このキーワードをわざわざ使う必要はありません。

確認ダイアログを処理するたびに、Selenium はデフォルトの動作に復帰します。そのため、都度 ``Choose Cancel On Next Confirmation`` を使わないかぎり、次のダイアログでは自動的に true (OKを押した) を返します。

確認ダイアログが出る度に、 ``Get Alert Message`` などでダイアログを処理しないと、それ以降の操作は全て失敗するので注意してください。


Clear Element Text
-----------------------

:Arguments: locator

``locator`` で指定したテキスト入力エレメントの値を消去します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Click Button
-----------------

:Arguments: locator

``locator`` で指定したボタンをクリックします。
キー属性は ``id``, ``name``, ``value`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Click Element
------------------

:Arguments: locator

``locator`` で指定したエレメントをクリックします。
キー属性は ``id``, ``name``, ``value`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Click Element At Coordinates
---------------------------------

:Arguments: locator, xoffset, yoffset

``locator`` で指定したエレメントから、 ``xoffset, yoffset`` 移動した場所をクリックします。カーソルがエレメントの中心に移動し、x, y はそこからの相対で計算します。
キー属性は ``id``, ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Click Image
----------------

:Arguments: locator

``locator`` で指定した画像をクリックします。
キー属性は ``id``, ``src``, ``alt`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Click Link
---------------

:Arguments: locator

``locator`` で指定したリンクをクリックします。
キー属性は ``id``, ``name``, ``href``, リンクテキストです。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Close All Browsers
-----------------------

開いている全てのブラウザを閉じ、ブラウザキャッシュをリセットします。

このキーワードを使うと、 ``Open Browser`` キーワードを呼び出すごとに返されていた
ブラウザウィンドウのインデクスがリセットされます。
テストやスイートの後始末 (teardown) の際には、このキーワードで全てのブラウザを閉じねばなりません。


Close Browser
------------------

現在のブラウザを閉じます。


Close Window
-----------------

:Arguments: 
  
現在開いているポップアップウィンドウを閉じます。


Confirm Action
-------------------

現在表示されている確認ダイアログを閉じ、表示されていたメッセージを返します。

デフォルトの動作では、このキーワードを使った際、ダイアログは「OK」ボタンで閉じられます。
「キャンセル」ボタンを押す必要がある場合は、事前に ``Cancel On Next Confirmation`` を呼び出しておき、ダイアログが出たときに自動的にキャンセルボタンが押されるようにしておいてください。

例:

.. code::

  # 確認ダイアログを表示する
  Click Button  Send
  # Ok ボタンが押され、メッセージが返る
  ${message}=  Confirm Action
  # 表示されたメッセージが "Are you sure?" だったかどうか確認する
  Should Be Equal  ${message}  Are you sure?
  # 次のダイアログではキャンセルを押す
  Choose Cancel On Next Confirmation
  # 確認ダイアログを表示する
  Click Button  Send
  # キャンセルが押される
  Confirm Action 


Create Webdriver
---------------------

:Arguments: driver_name, alias=None, kwargs={}, \**init_kwargs

WebDriver のインスタンスを生成します。

``Open Browser`` と動作が似ていますが、 WebDriver の ``__init__`` に引数を渡せる点が異なります。
どちらも使える時は、 ``Create Webdriver`` より ``Open Browser`` を使うほうが適切です。

生成したブラウザインスタンスのインデクスを返します。このインデクスは、あとでウィンドウを切り替えるのに使えます。
インデクスは 1 からはじまり、 ``Close All Browsers`` キーワードを使うとリセットされます。
`Switch Browser`_ の例を参照してください。
``driver_name`` には、 selenium.webdriver で定義されている WebDriver の名前をそのまま指定します。
使える WebDriver 名は、例えば Firefox, Chrome, Ie, Opera, Safari, PhantomJS, Remoto です。

キーワード引数 ``init_kwargs`` には WebDriver の ``__init__`` に渡す引数を指定します。この引数の中の値や式は、 WebDriver インスタンスの ``__init__`` が呼び出されるときまで評価されません。バージョン 2.8 以前の Robot Framework では ``init_kwargs`` がサポートされていないので、キーワード引数は全て辞書に入れて ``kwargs`` で渡す必要があります。
指定できるキーワード引数は `Selenium API ドキュメント <http://selenium-python.readthedocs.org/api.html>`_ を参照してください。

例:

.. code:: robotframework

  # プロキシを指定して Firefox インスタンスを生成する
  ${proxy}=  Evaluate  sys.modules['selenium.webdriver'].Proxy()  sys, selenium.webdriver
  ${proxy.http_proxy}=  Set Variable  localhost:8888
  Create Webdriver  Firefox  proxy=${proxy}
  # プロキシを指定して PhantomJS インスタンスを生成する
  ${service args}=  Create List  --proxy=192.168.132.104:8888
  Create Webdriver  PhantomJS  service_args=${service args}
  Example for Robot Framework < 2.8:
  # IE ドライバをデバッグモードで生成する
  ${kwargs}=  Create Dictionary  log_level=DEBUG  log_file=%{HOMEPATH}${/}ie.log
  Create Webdriver  Ie  kwargs=${kwargs}


Current Frame Contains
---------------------------

:Arguments: text, loglevel=INFO

現在のフレームに ``text`` が含まれているか検証します。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。


Current Frame Should Not Contain
-------------------------------------

:Arguments: text, loglevel=INFO

現在のフレームに ``text`` が含まれていないことを検証します。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。


Delete All Cookies
-----------------------

クッキーを全て除去します。


Delete Cookie
------------------

:Arguments: name

``name`` で指定したクッキーを除去します。
指定のクッキーが見つからなくても何もしません。


Dismiss Alert
------------------

:Arguments: accept=True

アラートダイアログで OK したときに true, ダイアログを消したときに false を返します。
アラートが出ていない状態でこのキーワードを使うと失敗します。
このキーワードが失敗すると、それ以降のテストは `Get Alert Message`_ などでアラートダイアログを閉じないかぎり失敗します。


Double Click Element
-------------------------

:Arguments: locator

``locator`` で指定したエレメントをダブルクリックします。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Drag And Drop
------------------

:Arguments: source, target

``source`` で指定したエレメントを、 ``target`` で指定したエレメントまでドラッグして移動し、ドロップします。

例:

.. code:: robotframework

  # elem1 を elem2 に動かす
  Drag And Drop  elem1  elem2


Drag And Drop By Offset
----------------------------

:Arguments: source, xoffset, yoffset

``source`` で指定したエレメントを ``xoffset``, ``yoffset`` の距離だけ移動します。
``xoffset``, ``yoffset`` は正負どちらの値もとりえます。

例:

.. code::

  # myElem を 50px 右, 35px 下に動かす
  Drag And Drop By Offset  myElem  50  -35


Element Should Be Disabled
-------------------------------

:Arguments: locator

``locator`` で指定したエレメントが使用不可 (disabled) であるか検証します。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Element Should Be Enabled
------------------------------

:Arguments: locator

``locator`` で指定したエレメントが使用可 (enabled) であるか検証します。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Element Should Be Visible
------------------------------

:Arguments: locator, message=

``locator`` で指定したエレメントが可視 (visible) であるか検証します。
ここでいう 「可視」とは、「論理的に可視である」ということで、「ブラウザのビューポート上に表示されている」ではありません。
例えば、 ``display:none`` のエレメントは論理的には不可視なので、このキーワードで検証すると失敗します。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Element Should Contain
---------------------------

:Arguments: locator, expected, message=


``locator`` で指定したエレメントがテキスト ``expected`` を含むか検証します。
エレメントのテキストが厳密に一致する (部分文字列の一致ではない) か検証したければ、 `Element Text Should Be`_ を使ってください。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Element Should Not Be Visible
----------------------------------

:Arguments: locator, message=

``locator`` で指定したエレメントが可視 (visible) で **ない** ことを検証します。
このキーワードは、 `Element Should Be Visible`_ の対極です。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Element Should Not Contain
-------------------------------

:Arguments: locator, expected, message=

``locator`` で指定したエレメントがテキスト ``expected`` を含まないことを検証します。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
キー属性は ``id`` と ``name`` です。 `Element Should Contain`_ も参照してください。


Element Text Should Be
---------------------------

:Arguments: locator, expected, message=

``locator`` で指定したエレメントが exactly テキスト ``expected`` を含むか検証します。
`Element Should Contain`_ と違って、このキーワードは部分一致ではなく完全一致かどうかを検証します。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Execute Async Javascript
-----------------------------

:Arguments: \*code

JavaScript コードを非同期に実行します。
複数行のコードを書きたい時は、各行をテストデータのセルに分けて書けます。この場合、実行時には各行をスペースを付加せずに結合してコードを生成し、実行します。
``code`` がファイルの絶対パスになっている場合、ファイルから JavaScript コードを読み出します。パスの区切りは、どのOSでも ``/`` を使います。
JavaScript のコードは、現在選択されているフレームやウィンドウの上で、無名関数の中身として実行されます。

`Execute Javascript`_ と似ていますが、このキーワードを使って実行したスクリプトは、コールバック関数 ``callback`` を使って、実行が終了したことを明にシグナルせねばなりません。
コールバックは、無名関数の最後の引数としてインジェクションされます。
スクリプトがタイムアウトまでに処理を終了できなければ、このキーワードは失敗します。 `タイムアウト <Timeouts>`_ も参照してください。

例:

.. code:: robotframework

  # ワンライナーで指定
  Execute Async JavaScript  var callback = arguments[arguments.length - 1];  window.setTimeout(callback, 2000);
  # スクリプトファイルで指定
  Execute Async JavaScript  ${CURDIR}/async_js_to_execute.js
  # 直接定義する
  ${retval}=  Execute Async JavaScript
  ...  var callback = arguments[arguments.length - 1];
  ...  function answer(){callback("text");};
  ...  window.setTimeout(answer, 2000);
  Should Be Equal  ${retval}  text


Execute Javascript
-----------------------

:Arguments: \*code

JavaScript コードを実行します。
複数行のコードを書きたい時は、各行をテストデータのセルに分けて書けます。この場合、実行時には各行をスペースを付加せずに結合してコードを生成し、実行します。
``code`` がファイルの絶対パスになっている場合、ファイルから JavaScript コードを読み出します。パスの区切りは、どのOSでも ``/`` を使います。
JavaScript のコードは、現在選択されているフレームやウィンドウの上で、無名関数の中身として実行されます。
現在のウィンドウを参照するために変数 ``window`` 、ドキュメントを参照するために ``document`` を使えます (例: ``document.getElementById('foo')``)。
このキーワードは、 JavaScript 中に ``return`` 文がない場合には ``None`` を返します。戻り値は適切な Python のデータ型や WebElements 型に変換されます。

例:

.. code:: robotframework

  Execute JavaScript  window.my_js_function('arg1', 'arg2')
  Execute JavaScript  ${CURDIR}/js_to_execute.js
  ${sum}=  Execute JavaScript  return 1 + 1;
  Should Be Equal  ${sum}  ${2}


Focus
----------

:Arguments: locator

``locator`` で指定したエレメントをフォーカスします。


Frame Should Contain
-------------------------

:Arguments: locator, text, loglevel=INFO

``locator`` で指定したフレームが ``text`` を含むか検証します。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Alert Message
----------------------

:Arguments: dismiss=True

現在表示されている JavaScript アラートのメッセージ本文を返します。
デフォルトの動作では、表示されたアラートダイアログは OK で閉じられます。
このキーワードは、アラートが表示されていなければ失敗になります。
アラートが出ている時、このキーワードなどでダイアログを閉じないと、以降のテストが失敗になるので注意してください。


Get All Links
------------------

現在のページに表示されている全てのリンクの id の入ったリストを返します。
リンクタグに id が入っていなければ、リスト中には空文字列が入ります。


Get Cookie Value
---------------------

:Arguments: name

クッキー名 ``name`` のクッキーの値を返します。
該当するクッキーがなければ、このキーワードは失敗します。

Get Cookies
----------------

現在のページの全てのクッキーを返します。


Get Element Attribute
--------------------------

:Arguments: attribute_locator

エレメントの属性の値を返します。
``attribute_locator`` の形式は、 ``element_id@class`` のように、ロケータの後に ``@`` と属性名をつけたものです。


Get Horizontal Position
----------------------------

:Arguments: locator

``locator`` で指定したエレメントの水平位置を返します。
値はページの左端からの距離で、ピクセル単位の整数です。
エレメントが存在しなければ失敗します。

.. seealso:: `Get Vertical Position`_


Get List Items
-------------------

:Arguments: locator

``locator`` で指定した select リストの全選択肢の値を返します。
リストとコンボボックスのどちらにも使えます。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Location
-----------------

:Arguments: 
  
現在の URL を返します。


Get Matching Xpath Count
-----------------------------

:Arguments: xpath

``xpath`` に一致するエレメントの数を返します。
XPath 前提なので、ロケータに ``xpath=`` プレフィクスをつけてはなりません。

.. code:: robotframework

  # 正しい:
  count =  Get Matching Xpath Count  //div[@id='sales-pop']
  # 誤り:
  count =  Get Matching Xpath Count  xpath=//div[@id='sales-pop']

マッチしたエレメントの数を検証したければ、 `Xpath Should Match X Times`_ を使ってください。


Get Selected List Label
----------------------------

:Arguments: locator

``locator`` で指定した選択リストで、現在選択されている要素の表示ラベルを返します。
リストとコンボボックスのどちらにも使えます。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Selected List Labels
-----------------------------

:Arguments: locator

``locator`` で指定した選択リストで、現在選択されている要素の表示ラベルをリストとして返します。
選択されている要素がないときは失敗します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Selected List Value
----------------------------

:Arguments: locator

``locator`` で指定した選択リストで、現在選択されている要素の値 (``value``) を返します。
戻り値は、選択されているええ面との ``value`` 属性の値です。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Selected List Values
-----------------------------

:Arguments: locator

``locator`` で指定した選択リストで、現在選択されている要素の値 (``value``) をリストとして返します。
選択されている要素がないときは失敗します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Selenium Implicit Wait
-------------------------------

Selenium が待機する時間の設定値を秒で返します。
値の意味は `Set Selenium Implicit Wait`_ を参照してください。


Get Selenium Speed
-----------------------

Selenium にコマンドを出したあとに待機する時間を返します。
値の意味は `Set Selenium Speed`_ を参照してください。


Get Selenium Timeout
-------------------------

様々なキーワードで使われているタイムアウトの長さを秒で返します。
値の意味は `Set Selenium Timeout`_ を参照してください。


Get Source
---------------

現在のページやフレームの HTML ソースを返します。


Get Table Cell
-------------------

:Arguments: table_locator, row, column, loglevel=INFO

テーブルセルの内容を返します。
``row`` と ``column`` の番号は、 1 から開始します。ヘッダやフッタの行やカラムも数に入ります。
``row`` や ``column`` を負の数で指定すると、末尾 (-1) からの行数指定になります。
ヘッダやフッタの行の内容も、このキーワードで取得できます。
テーブルの指定方法は  `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。


Get Text
-------------

:Arguments: locator

``locator`` で指定したエレメントの text 値を返します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Title
--------------

現在のページのタイトルを返します。


Get Value
--------------

:Arguments: locator

``locator`` で指定したエレメントの ``value`` 属性の値を返します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Vertical Position
--------------------------

:Arguments: locator

``locator`` で指定したエレメントの垂直方向の位置を返します。
位置は、ページの先頭からの距離をピクセル単位で表した整数です。
エレメントが見つからない時は失敗になります。

.. seealso:: `Get Horizontal Position`_


Get Webelement
-------------------

:Arguments: locator

``locater`` にマッチする WebElement を返します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Webelements
--------------------

:Arguments: locator

``locater`` にマッチする WebElement のリストを返します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Get Window Identifiers
---------------------------

ブラウザが把握している全てのウィンドウの id 属性をログに出力して返します。


Get Window Names
---------------------

ブラウザが把握している全てのウィンドウの name 属性をログに出力して返します。


Get Window Position
------------------------

現在のウィンドウの位置を x, y の順で返します。

例:

.. code:: robotframework

  ${x}  ${y}=  Get Window Position


Get Window Size
--------------------

現在のウィンドウサイズを width, height の順で返します。

例:

.. code:: robotframework

  ${width}  ${height}=  Get Window Size


Get Window Titles
----------------------

ブラウザが把握している全てのウィンドウのタイトルをログに出力して返します。
  

Go Back
------------

ブラウザの「戻る」ボタンをユーザがクリックした時の動作をシミュレートします。


Go To
----------

:Arguments: url

アクティブなブラウザインスタンスを使って、指定の URL に移動します。


Input Password
-------------------

:Arguments: locator, text

``locator`` で指定したテキストフィールドに、パスワードをタイプ入力します。
``Input Text`` キーワードとの違いは、指定したパスワードがログに残らないところです。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Input Text
---------------

:Arguments: locator, text

``locator`` で指定したテキストフィールドに、 ``text`` をタイプ入力します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Input Text Into Prompt
---------------------------

:Arguments: text

アラートボックスに ``text`` をタイプ入力します。


List Selection Should Be
-----------------------------

:Arguments: locator, \*items

``locator`` で指定した選択肢リストで選択されている項目が ``*items`` と完全一致するか検証します。
何も選択されていないことを検証したければ、単に ``*items`` の指定をなくしてください。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


List Should Have No Selections
-----------------------------------

:Arguments: locator

``locator`` で指定した選択肢リストで何も選択されていないことを検証します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


List Windows
-----------------

現在の全てのウィンドウハンドルをリストで返します。


Location Should Be
-----------------------

:Arguments: url

現在の URL が ``url`` と完全一致することを検証します。


Location Should Contain
----------------------------

:Arguments: expected

現在の URL に ``expected`` が含まれているか検証します。


Locator Should Match X Times
---------------------------------

:Arguments: locator, expected_locator_count, message=, loglevel=INFO

ページ内に ``locator`` で指定したエレメントが ``expected_locator_count`` 個あるか検証します。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Log Location
-----------------

現在の URL をログに記録して返します。


Log Source
---------------

:Arguments: loglevel=INFO

現在のページやフレームの HTML ソース全体をログに記録して返します。
``loglevel`` はログの記録に使うログレベルで、 ``WARN``, ``INFO`` (デフォルト), ``DEBUG``, ``NONE`` (ログに記録しない) のいずれかです。


Log Title
--------------

現在のページのタイトルをログに記録して返します。


Maximize Browser Window
----------------------------

現在のブラウザウィンドウを最大化します。


Mouse Down
---------------

:Arguments: locator

``locator`` で指定したエレメントの上でマウスの左ボタンを押下する操作をシミュレートします。
マウスボタンをリリースしない限り、エレメントはプレス状態になります。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。

特定エレメント向けの `Mouse Down On Image`_ や `Mouse Down On Link`_ も参照してください。


Mouse Down On Image
------------------------

:Arguments: locator

画像上でのマウスボタン押下をシミュレートします。
キー属性は ``id``, ``src``, ``alt`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Mouse Down On Link
-----------------------

:Arguments: locator

リンク上でのマウスボタン押下をシミュレートします。
キー属性は ``id``, ``name``, ``href``, リンクテキストです。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Mouse Out
--------------

:Arguments: locator

``locator`` で指定したエレメントからマウスカーソルを外す操作をシミュレートします。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Mouse Over
---------------

:Arguments: locator

``locator`` で指定したエレメントの上にマウスカーソルをホバーする操作をシミュレートします。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Mouse Up
-------------

:Arguments: locator

``locator`` で指定したエレメントで、マウスの左ボタンをリリースする操作をシミュレートします。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Open Browser
-----------------

:Arguments: url, browser=firefox, alias=None, remote_url=False, desired_capabilities=None, ff_profile_dir=None

新しいブラウザインスタンスを開き、指定の URL に行きます。
生成したブラウザインスタンスのインデクスを返します。このインデクスは、あとでウィンドウを切り替えるのに使えます。
インデクスは 1 からはじまり、 ``Close All Browsers`` キーワードを使うとリセットされます。
`Switch Browser`_ の例を参照してください。
オプションの ``alias`` はブラウザインスタンスの別名で、ブラウザを切り替える時に (インデクスで切り替えるのと同じ感覚で) 使えます。詳しくは `Switch Browser`_ を参照してください。

``browser`` に使える値は以下の通りです:

.. table:: 

  ===================== ===============================
  シンボル              ブラウザ
  ===================== ===============================
  ``firefox``           FireFox
  ``ff``                FireFox
  ``internetexplorer``  Internet Explorer
  ``ie``                Internet Explorer
  ``googlechrome``      Google Chrome
  ``gc``                Google Chrome
  ``chrome``            Google Chrome
  ``opera``             Opera
  ``phantomjs``         PhantomJS
  ``htmlunit``          HTMLUnit
  ``htmlunitwithjs``    HTMLUnit (JavaScript サポート)
  ``android``           Android
  ``iphone``            Iphone
  ``safari``            Safari
  ===================== ===============================

Internet Explorer のブラウザインスタンスが複数あると、動作がおかしなことになるので注意してください。
IE ブラウザが一つしか起動していないときだけ、 `Switch Browser`_ が動作するのも同じ理由です。
詳しくは http://selenium-grid.seleniumhq.org/faq.html#i_get_some_strange_errors_when_i_run_multiple_internet_explorer_instances_on_the_same_machine を参照してください。
オプションの ``remote_url`` は、 ``http://127.0.0.1/wd/hub`` のような遠隔の Selenium サーバの URL です。
この値を指定する場合、 ``desired_capabilities`` でリモートサーバのケイパビリティを設定できます。
設定は ``key1:val1,key2:val2`` のような文字列形式にします。
このオプションは、 IE でプロキシサーバを指定するときや、 saucelabs.com のようなサービスでブラウザと OS を指定するときに便利です。
``desired_capabilities`` には (`Create Dictionary`_ で作成した) 辞書も指定でき、より複雑なコンフィギュレーションを扱えます。
オプションの ``ff_profile_dir`` は firefox のプロファイルで、デフォルトプロファイルをオーバライドするときに使います。

.. _`Create Dictionary`: http://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Create%20Dictionary


Open Context Menu
----------------------

:Arguments: locator

``locator`` で指定したエレメントのコンテキストメニューを開きます。


Page Should Contain
------------------------

:Arguments: text, loglevel=INFO

現在のページに ``text`` が含まれているか検証します。
このキーワードが失敗すると、自動的にページのソースを ``loglevel`` に指定したログレベルで記録します。
指定できるログレベルは ``DEBUG``, ``INFO`` (デフォルト), ``WARN``, ``NONE`` です。
ログレベルに ``NONE`` または現在のログレベルより低いレベルを指定すると、ページソースをログに出力しません。


Page Should Contain Button
-------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したボタンが含まれているか検証します。
このキーワードは、 ``<input>`` または ``<button>`` タグで作られたボタンを探します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``name``, ``value`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Checkbox
---------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したチェックボックスが含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Element
--------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したエレメントが含まれているか検証します。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Image
------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定した画像が含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``src``, ``alt`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Link
-----------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したリンクが含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id`, ``name``, ``href``, リンクテキストです。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain List
-----------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定した選択リストが含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性はは ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Radio Button
-------------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したラジオボタンが含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``name``, ``value`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Contain Textfield
----------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したテキストフィールドが含まれているか検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain
----------------------------

:Arguments: text, loglevel=INFO

現在のページに ``text`` が含まれていないことを検証します。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。


Page Should Not Contain Button
-----------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したボタンが含まれていないことを検証します。
このキーワードは ``<input>`` または ``<button>`` タグで作ったボタンを検索します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は id, name and value です。 エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Checkbox
-------------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したチェックボックスが含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Element
------------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したエレメントが含まれていないことを検証します。
``message`` は、デフォルトのエラーメッセージをオーバライドしたいときに使います。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。
キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Image
----------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定した画像が含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``src``, ``alt`` です。 エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Link
---------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したリンクが含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``href``, ``alt``, リンクテキストです。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain List
---------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定した選択リストが含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Radio Button
-----------------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したラジオボタンが含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性は ``id``, ``name``, ``value`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Page Should Not Contain Textfield
--------------------------------------

:Arguments: locator, message=, loglevel=INFO

現在のページに ``locator`` で指定したテキストフィールドが含まれていないことを検証します。
``message`` と ``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。
キー属性 ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Press Key
--------------

:Arguments: locator, key

``locator`` で指定したエレメントでキーを押す操作をシミュレートします。
``key`` は単一の文字、文字列、または ``\\`` でプレフィクスした ASCII コード表現にできます。

例:

.. code:: robotframework

  Press Key  text_field  q
  Press Key  text_field  abcde
  # Enter キーを ASCII コードで記述
  Press Key  login_button  \\13


Radio Button Should Be Set To
----------------------------------

:Arguments: group_name, value

``group_name`` で指定したラジオボタンの選択値が ``value`` であるか検証します。
検証します。
ラジオボタンの指定方法は `Select Radio Button`_ を参照してください。


Radio Button Should Not Be Selected
----------------------------------------

:Arguments: group_name

``group_name`` で指定したラジオボタンがどれも選択されていないことを検証します。
ラジオボタンの指定方法は `Select Radio Button`_ を参照してください。


Register Keyword To Run On Failure
---------------------------------------

:Arguments: keyword

Selenium2Library 上のキーワードが失敗したときに実行される run-on-failure キーワードを設定します。
``keyword`` は、テスト中に Selenium2Library のキーワードの実行が失敗したときに呼び出されるキーワードです (インポート済みで使えるキーワードを指定できます)。
引数を伴うキーワードは指定できません。 ``Nothing`` を指定すれば、この機能を無効にできます。
run-on-failure キーワードの初期値は Selenium2Library のインポート時に指定できます。
デフォルトの設定は `Capture Page Screenshot`_ です。
エラーが起きたとき当該ページのスクリーンショットを撮れるのはとても便利なのですが、実行は遅くなることがあります。
このキーワードは、設定前の run-on-failure キーワードを返すので、元の値を退避して、あとで復帰できます。

例:

.. code:: robotframework

  # 失敗したら Log Source を実行する
  Register Keyword To Run On Failure  Log Source
  # 現在の run-on-failure を ${previous kw} に退避して、run-on-failure を無効にする
  ${previous kw}=  Register Keyword To Run On Failure  Nothing
  # 退避した run-on-failure キーワードを復帰する
  Register Keyword To Run On Failure  ${previous kw}

run-on-failure 機構は、 Python または Jython 2.4 以降でテストを実行している時のみ使えます。
IronPython では使えません。


Reload Page
----------------

ページのリロード操作をシミュレートします。


Remove Location Strategy
-----------------------------

:Arguments: strategy_name

以前追加したカスタムのロケーションストラテジを除去します。
デフォルトのストラテジを指定すると失敗します。
カスタムのロケーションストラテジの追加は `Add Location Strategy`_ を参照してください。


Select All From List
-------------------------

:Arguments: locator

``locator`` で指定したマルチセレクトのできるリストで、全ての要素を選択します。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select Checkbox
-----------------------------------

:Arguments: locator

``locator`` で指定したチェックボックスを選択します。
チェックボックスが選択済みなら何もしません。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select Frame
-----------------

:Arguments: locator

``locator`` で指定したフレームを現在のフレームにします。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select From List
---------------------

:Arguments: locator, \*items

``locator`` で指定したリストで、 ``*items`` に一致する要素を選択します。
複数項目を同時選択できないリストに対して複数の値を指定した場合には、最後に指定した値が選択されます。
複数項目を選択できるリストで、 ``*items`` が空のリストの場合、 **全ての項目** が選択されます。
``*items`` の照合にはまず ``value`` に対する一致を試し、つぎにラベルに対する一致を試します。
``... By Index/Value/Label`` のキーワードを使うほうが高速です。
複数項目を同時選択できないリストで、最後に指定した要素がリスト上にない場合、例外が送出され、 ``*items`` のうちリストにない要素全てについて警告が出ます。
複数項目を選択できるリストでは、 ``*items`` のいずれか一つの要素でもリスト上にない場合に例外が送出されます。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select From List By Index
------------------------------

:Arguments: locator, \*indexes

``locator`` で指定したリストで、インデクスが ``*indexes`` である要素を選択します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select From List By Label
------------------------------

:Arguments: locator, \*labels

``locator`` で指定したリストで、ラベルが ``*labels`` である要素を選択します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select From List By Value
------------------------------

:Arguments: locator, \*values

``locator`` で指定したリストで、 ``value`` が ``*values`` である要素を選択します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Select Radio Button
------------------------

:Arguments: group_name, value

``group_name`` で指定したラジオボタンの選択値を ``value`` にします。
``group_name`` でラジオボタンを指定する方法は二つあります:

* ``group_name`` は ``radio`` インプットの ``name`` と照合します。
* ``value`` は ``value`` または ``id`` 属性と照合します。

従って、条件に一致するラジオボタンを検索するための XPath は
``//input[@type='radio' and @name='group_name' and (@value='value' or @id='value')]`` のようになります。

例:

.. code:: robotframework

  # <input type="radio" name="size" value="XL">XL</input> であるような HTML に一致
  Select Radio Button  size  XL
  # <input type="radio" name="size" value="XL" id="sizeXL">XL</input> であるような HTML に一致
  Select Radio Button  size  sizeXL


Select Window
------------------

:Arguments: locator=None

``locator`` に一致するウィンドウを選択し、以前のウィンドウのウィンドウハンドルを返します。
``locator`` に指定できるのは、ウィンドウの名前、タイトル、 URL, 除外リスト (excluded handle's list), 特定のウィンドウを表すキーワード (special words) です。
選択中のウィンドウから切り替わった場合はそのウィンドウのハンドル値、選択中のウィンドウがない状態から選択した場合には None を返します。
ウィンドウが見つかった場合、次にこのキーワードで別のウィンドウを選択するまで、以後のコマンドはすべて新たに選択したウィンドウに対して実行されます。
ウィンドウが見つからなかった場合は失敗になります。
デフォルトの動作では、 ``locator`` を指定した場合、ウィンドウのタイトルと、 javascript 上のウィンドウ名を使って一致するウィンドウを探します。複数のウィンドウが一致した場合には、最初に見つかったウィンドウを選択します。
特定のウィンドウを表すロケータとして、以下があります。

.. table::

  ==================== =======================================================================================
  キーワード
  ==================== =======================================================================================
  ``main`` (文字列)    メインウィンドウを返す
  ``self`` (文字列)    現在のウィンドウ (結果的に現在のウィンドウハンドルを返す)
  ``new`` (文字列)     ウィンドウのインデクスの値が最も大きいものを最新として返す
  除外リスト (リスト)  `List Windows`_ などで取得したウィンドウハンドルリストに含まれないもののうち最初のもの
  ==================== =======================================================================================
 
Selenium2Library のアプローチでロケータストラテジを指定してウィンドウを探すこともできます:

.. table::

  =========== ========================================= =========================================
  ストラテジ  例                                        説明
  =========== ========================================= =========================================
  ``title``   ``Select Window  title=My Document``      ウィンドウタイトルで一致
  ``name``    ``Select Window  name=${name}``           ウィンドウの Javascript 上の名前で一致
  ``url``     ``Select Window  url=http://google.com``  ウィンドウの現在の URL で一致
  =========== ========================================= =========================================

例:

.. code:: robotframework

  Click Link  popup_link  # opens new window
  Select Window  popupName  
  Title Should Be  Popup Title  
  # Chooses the main window again
  Select Window


Set Browser Implicit Wait
------------------------------

:Arguments: seconds

現在のブラウザの暗黙の待機時間 (implicit wait) を秒で設定します。
Selenium2 の対応する関数の説明には、「エレメントを取得したり、コマンドを実行し終わるまで待機するためのタイムアウトで、設定は持続する。セッション中で一度だけ呼び出せばよい ('Sets a sticky timeout to implicitly wait for an element to be found, or a command to complete. This method only needs to be called one time per session.')」とあります。

例:

.. code:: robotframework

  Set Browser Implicit Wait  10 seconds

.. seealso:: `Set Selenium Implicit Wait`_


Set Screenshot Directory
-----------------------------

:Arguments: path, persist=False

キャプチャした画像置き場となるディレクトリを設定します。
``path`` にはスクリーンショットを保存する先の絶対パスを指定します。
指定したパスが存在しなければ、作成されます。
``persist`` を指定すると、テスト全体の実行が終わるまでパスの設定が維持されます。
それ以外の場合は、現在のテスト実行スコープが終わった時点で、以前の値に復帰します。


Set Selenium Implicit Wait
-------------------------------

:Arguments: seconds

Selenium 2 のデフォルトの暗黙の待機時間 (implicit wait) を秒で指定し、開いている全てのブラウザに適用します。
Selenium2 の対応する関数の説明には、「エレメントを取得したり、コマンドを実行し終わるまで待機するためのタイムアウトで、設定は持続する。セッション中で一度だけ呼び出せばよい ('Sets a sticky timeout to implicitly wait for an element to be found, or a command to complete. This method only needs to be called one time per session.')」とあります。

例:

.. code:: robotframework

  ${orig wait} =  Set Selenium Implicit Wait  10 seconds
  # (何か時間のかかる Ajax 処理などを実行)
  Set Selenium Implicit Wait  ${orig wait}


Set Selenium Speed
-----------------------

:Arguments: seconds

Selenium がコマンドを実行した後に待機する時間を設定します。
このコマンドは、テスト実行を遅くして、実行の様子を目で確認したいときに便利です。
``seconds`` は Robot Framework の時間表現の形式を使えます。
設定前のスピード値を返します。

例:

.. code:: robotframework

  Set Selenium Speed  .5 seconds


Set Selenium Timeout
-------------------------

:Arguments: seconds

様々なキーワードで使われているタイムアウトを秒で指定します。
Selenium2Library では、 ``timeout`` を引数にとる ``Wait ...`` 系のキーワードがいくつかあります。
これらの ``timeout`` はいずれも省略可能で、このキーワードで設定値をグローバルに変更できます。
詳しくは `タイムアウトの設定 <Timeouts>`_ を参照してください。
このキーワードは設定前の ``timeout`` を返すので、後で以前の値を復帰するときに使えます。
デフォルトのタイムアウトは 5 秒ですが、 Selenium2Library のインポート時に変更されることもあります。

例:

.. code:: robotframework

  ${orig timeout} =  Set Selenium Timeout  15 seconds
  # (表示の遅いページをロードする)
  Set Selenium Timeout  ${orig timeout}


Set Window Position
------------------------

:Arguments: x, y

現在のウィンドウの位置を ``x, y`` に設定します。

例:

.. code:: robotframework

  Set Window Size  ${1000}  ${0}
  ${x}  ${y}=  Get Window Position
  Should Be Equal  ${x}  ${1000}
  Should Be Equal  ${y}  ${0}


Set Window Size
--------------------

:Arguments: width, height

現在のウィンドウの幅と高さを ``width, height`` に設定します。

例:

.. code:: robotframework

  Set Window Size  ${800}  ${600}
  ${width}  ${height}=  Get Window Size
  Should Be Equal  ${width}  ${800}
  Should Be Equal  ${height}  ${600}


Simulate
-------------

:Arguments: locator, event

``locator`` で指定したエレメントで ``event`` の発生をシミュレートします。
このキーワードはエレメントに OnEvent 系のハンドラが実装されていて、それを呼び出したいときに便利です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Submit Form
----------------

:Arguments: locator=None

``locator`` で指定したフォームを submit します。
``locator`` が空の場合、ページの最初のフォームを submit します。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Switch Browser
-------------------

:Arguments: index_or_alias

``index_or_alias`` を使ってアクティブなブラウザを切り替えます。
インデクスは `Open Browser`_ が返す値で、エイリアスは `Open Browser`_ の呼び出し時に付与できます。

例e:

.. code:: robotframework

  Open Browser  http://google.com  ff
  Location Should Be  http://google.com
  Open Browser  http://yahoo.com  ie  2nd conn
  Location Should Be  http://yahoo.com
  Switch Browser  1  # インデクス指定
  Page Should Contain  I'm feeling lucky
  Switch Browser  2nd conn  # エイリアス指定
  Page Should Contain  More Yahoo!
  Close All Browsers

上の例では、インデクスに ``1`` を使っていて、最初の ``Open Browser`` でブラウザを開くよりも前はブラウザウィンドウは存在しなかったという前提になっています。
今のブラウザインデクスが分からなければ、以下のようにして取得できます。

.. code:: robotframework

  ${id} =  Open Browser  http://google.com  *firefox
  # Do something ...    
  Switch Browser  ${id}  


Table Cell Should Contain
------------------------------

:Arguments: table_locator, row, column, expected, loglevel=INFO

テーブル中の特定のセルに ``expected`` が含まれているか検証します。
行およびカラム番号は 1 から始めます。このキーワードがパスするのは、指定のセルに指定のコンテンツが含まれている場合です。
セルの内容を厳密一致で調べたい場合や、セルの内容が特定のテキストから始まっているかを調べたい場合などは、 `Get Table Cell`_ と、Robot Framework 組み込みの  ``Should Be Equal`` や ``Should Start Width`` を使ってください。
テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain`_ を参照してください。


Table Column Should Contain
--------------------------------

:Arguments: table_locator, col, expected, loglevel=INFO

特定のテーブルカラムに ``expected`` が含まれているか検証します。
最左端のカラムのカラム番号を 1 とします。
負のカラム番号は、行の末尾のカラム (末尾: -1) から数えた番号として使えます。
テーブルに複数カラムにわたるセルがある場合、結合されているセル一つ一つもカラムとして数えます。
例えば、ある行のカラム A と B が colspan="2" で結合されていて、論理的な 3 番目のカラムに ``C`` が入っている場合、以下のテストはいずれも動作します。

例:

.. code:: robotframework

  Table Column Should Contain  tableId  3  C
  Table Column Should Contain  tableId  2  C

テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Table Footer Should Contain
--------------------------------

:Arguments: table_locator, expected, loglevel=INFO

テーブルフッタに ``expected`` が含まれているか検証します。
テーブルフッタは、 ``<tfoot>`` 要素の子要素である ``<td>`` 要素です。
テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Table Header Should Contain
--------------------------------

:Arguments: table_locator, expected, loglevel=INFO

テーブルヘッダ、つまりいずれかの ``<th>...</th>`` エレメントに ``expected`` が含まれているか検証します。
テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Table Row Should Contain
-----------------------------

:Arguments: table_locator, row, expected, loglevel=INFO

特定のテーブル行に ``expected`` が含まれているか検証します。
先頭の行の行番号は 1 です。行番号を負の数で指定すると、テーブルの末尾から数えた (末尾: -1) 指定になります。
``<thead>``, ``<tbody>``, ``<tfoot>`` からなるテーブルの場合、 ``<tbody>`` セクションだけが検索対象です。
ヘッダやフッタのコンテンツを検索したいときは、 `Table Header Should Contain`_ や `Table Footer Should Contain`_ を使ってください。
複数行にまたがるセルが存在する場合、結合したセルの最も上のセルだけがマッチします。
テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Table Should Contain
-------------------------

:Arguments: table_locator, expected, loglevel=INFO

テーブル中のどこかに ``expected`` が含まれているか検証します。
テーブルの指定方法は、 `テーブル、行、列などの指定方法 <locating table>`_ を参照してください。
``loglevel`` の説明は `Page Should Contain Element`_ を参照してください。


Textarea Should Contain
----------------------------

:Arguments: locator, expected, message=

``locator`` で指定したテキストエリアがテキスト ``expected`` を含むか検証します。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Textarea Value Should Be
-----------------------------

:Arguments: locator, expected, message=

``locator`` で指定したテキストエリアの値が ``expected`` と厳密一致するか検証します。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Textfield Should Contain
-----------------------------

:Arguments: locator, expected, message=

``locator`` で指定したテキストフィールドがテキスト ``expected`` を含むか検証します。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Textfield Value Should Be
------------------------------

:Arguments: locator, expected, message=

``locator`` で指定したテキストフィールドの値が ``expected`` と厳密一致するか検証します。
``message`` はデフォルトのエラーメッセージをオーバライドするのに使います。
キー属性は ``id`` と ``name`` です。. エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Title Should Be
--------------------

:Arguments: title

現在のページのタイトルが ``title`` と一致するか検証します。


Unselect Checkbox
----------------------

:Arguments: locator

``locator`` で指定したチェックボックスの選択を解除します。
チェックボックスが非選択なら何もしません。
キー属性は ``id`` と ``name`` です。
エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Unselect Frame
------------------- 

トップフレームを現在のフレームにします。


Unselect From List
-----------------------

:Arguments: locator, \*items

``locator`` で指定した選択リストで、 ``*items`` に指定した値の選択を解除します。
引数の特殊なケースとして ``*items`` が空のリストのときには **全ての選択を解除** します。
``*items`` の一致は、 ``value`` と ``label`` の **両方で** 試みます。
``... By Index/Value/Label`` のキーワードを使うほうが高速です。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Unselect From List By Index
--------------------------------

:Arguments: locator, \*indexes

``locator`` で指定したリストから ``*indexes`` で指定したインデクスの要素の選択を解除します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Unselect From List By Label
---------------------------------

:Arguments: locator, \*labels

``locator`` で指定したリストから ``*labels`` で指定したラベルの要素の選択を解除します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。


Unselect From List By Value
--------------------------------

:Arguments: locator, \*values

``locator`` で指定したリストから ``*values`` で指定した値の要素の選択を解除します。
リストとコンボボックスのどちらにも使えます。キー属性は ``id`` と ``name`` です。エレメントの指定方法は  `エレメントの探索と指定 <locating elements>`_ を参照してください。

Wait For Condition
-----------------------

:Arguments: condition, timeout=None, error=None

``condition`` が true になるか、タイムアウトになるまで待機します。
``condition`` には任意の JavaScript の式を指定できますが、必ず末尾に (値を返す) ``return`` 文がなければなりません。
JavaScript からウィンドウのコンテンツにアクセスする方法は `Execute JavaScript`_ を参照してください。`
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Element Contains
--------------------------------

:Arguments: locator, text, timeout=None, error=None

``locator`` で指定したエレメントが ``text`` が含んだ状態になるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Element Does Not Contain
----------------------------------------

:Arguments: locator, text, timeout=None, error=None

``locator`` で指定したエレメントが ``text`` を含まない状態になるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Element Is Enabled
----------------------------------

:Arguments: locator, timeout=None, error=None

``locator`` で指定したエレメントが操作可能 (enabled) になるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Element Is Not Visible
--------------------------------------

:Arguments: locator, timeout=None, error=None

``locator`` で指定したエレメントが不可視 (not visible) になるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Element Is Visible
----------------------------------

:Arguments: locator, timeout=None, error=None

``locator`` で指定したエレメントが可視 (visible) になるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Page Contains
-----------------------------

:Arguments: text, timeout=None, error=None

現在のページに ``text`` が出現するまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains Element`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Page Contains Element
-------------------------------------

:Arguments: locator, timeout=None, error=None

現在のページに ``locator`` で示したエレメントが出現するまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Wait Until Page Does Not Contain
-------------------------------------

:Arguments: text, timeout=None, error=None

現在のページから ``text`` がなくなるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)



Wait Until Page Does Not Contain Element
---------------------------------------------

:Arguments: locator, timeout=None, error=None

現在のページから ``locator`` で示したエレメントがなくなるまで待機します。
``timeout`` が経過してしまうと失敗になります。
``timeout`` の詳細とデフォルト値については `タイムアウトの設定 <Timeouts>`_ を参照してください。
``error`` はデフォルトのエラーメッセージをオーバライドするときに使います。

.. seealso:: `Wait Until Page Contains`_, `Wait For Condition`_, `Wait Until Element Is Visible`_, `Wait Until Keyword Succeeds`_ (Robot Framework 組み込みキーワード)


Xpath Should Match X Times
-------------------------------

:Arguments: xpath, expected_xpath_count, message=, loglevel=INFO

ページ中に、 ``xpath`` で指定したエレメントが指定個数あるかを検証します。

xpath を指定するときに、 ``xpath=`` を含めてはなりません。
使える表記は XPath です。

::
   # 正しい
   Xpath Should Match X Times  //div[@id='sales-pop']  1

   #誤り 
   Xpath Should Match X Times  xpath=//div[@id='sales-pop']  1

``message`` と ``loglevel`` 引数は `Page Should Contain Element`_  を参照
してください。


.. _`Wait Until Keyword Succeeds`: http://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Wait%20Until%20Keyword%20Succeeds

