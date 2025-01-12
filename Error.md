- どの情報を告知するか
    - ログレベルによるフィルタリング
        - debug・・SQLログ、リクパラ、レスポンスなど一番詳細な情報。
        - info・・メソッドの開始、終了など。正常系の進捗報告など。
        - warning・・ユーザーの正常な動作として起こり得る準異常系。バリデーションなどはここ
        - error・・意図したExceptionて起こり得るエラー。4XX系。データの不整合などで起こり得る業務エラー系。
        - fatal・・意図しないException。500系。通常起こり得ないエラー(DB不接続、カラム不整合)
- 出すべき情報はなにか(情報の内容と量)
    - 情報が少ないと追えないし、多すぎると埋もれてしまう
    - ログに関するTips
        - クラス名、メソッド、行数、時間入れておくと楽。時間はマスト。
        - 面倒でなければメソッドのInパラメータは吐いた方が良い(追跡が楽)
        - ログにだす情報は改行しないほうがいいかも・・。(特にSQL)不必要に改行を入れると非常にみにくい
        - 多くなりすぎなければ正常系の処理が終わった後につけるのも良い。
        - ログには記号+番号などを入れておくと検索が楽。ログが出た場所をすぐに特定できるために。
        - HTMLデータやパラメータが多すぎるものときなどは文字制限を入れないとログデータ自体が多すぎることがあるので必要に応じて少なくした方が良いかも・・
        - ログに出す詳細情報はInパラメーター、キー系のID情報、エラーメッセージ。
        - エラーメッセージはstackTraceなどがはっきりと出ることを確認しておくこと
        - 共通度が高いメソッドなどはログが大量に出て他の部分を邪魔することもあるので出し分け要注意。
        - 複数のバッチが入るときはチャネルを分けると処理が混ざらないので見やすいかも。
        - 集計することがあるのでキー系の情報がはっきりと拾えるようにしておくこと

- 情報を拾うサービスや連携先
    - 生ログ
        - 比較的小規模〜中規模だったり、レガシーな現場ではいまだに生ログを出力しているというプロジェクトが多いのではないでしょうか。 ただそのまま出しているだけあっていろいろなカスタマイズが可能だったりします。
        - メリット
            - 後述するような設計スキルやエラーレポーティングサービスへの知見が特にいらない
            - 全ての情報を自由自在に出力することができる
        - デメリット
            - レベルにもよるが、膨大なサイズになり、ログの検索が大変
            - 上記のサイズ制限などにより、保存や管理がしにくい
            -レベル分けや入れる情報を精査しないと検索ができない(ログの設計能力がいる)
    - 収集サービス(Cloudwatch、LogAnalyticsなど)
        - ログを抽出して、検索できるようにした各種のサービス。SQLに近いような検索ロジックがつかえることもあります。
        - メリット
            - さまざまな検索クエリなどをつかうことができ、生ログよりも目的の情報を探しやすい
            - 実ログを吐き出しているだけなので、カスタマイズがしやすい
        - デメリット
            - サービスの学習コスト、慣れや検索クエリを覚える手間が若干必要
            - 生ログをはいているだけなので、ログ設計指針をしっかりしておかないと検索が大変
    - エラーレポーティングサービス(Sentryなど)
        - ログを見る機会は特にエラー時に集中しているため、エラー発生時に告知するようなサービス(それゆえSlackやメールなどの告知と連携していることも多い)
        - メリット
            - エラーの場合のみ、レポートが送られてくるので、ログよりは見る情報が必然的に少なくて済む
            - 情報が細かくカテゴライズされており、検索や抽出が容易
        - デメリット
            - エラーが吐かれない異常の場合、動きを追えない
            - 構築の段階でどのエラーハンドリングをもれなくサービスに連携させる設計が必要
            - 全ての情報が見れるわけではないので、ある程度推測が必要な部分がでてくる

- エラー時に告知する情報の切り分け
    - 環境が何か
        - 開発・・詳細な記録まで残す(debug)
        - 本番・・一定以上のエラーのみ(error)
    - 対象が誰か
        - 一般ユーザー
            - stackTraceなどの細かい情報は秘匿性の観点からみせない。
            - 見せる情報は「その情報で何が原因かがわかること」(例:在庫が不足しています。)
            - 問い合わせる際に、どのようなエラーだったかをわかるようにエラーコードがついていると検索が容易。
        - システム管理者
            - なるべく詳細なエラー情報が残すべし。
            - できるだけそのエラーだけで必要な情報がわかると尚よい(リクパラなどの再現に必要な情報)
- エラー報告時に必要な情報
    - バグ管理ツールなどに記載する必要があるため、なるべく詳細な情報を載せる必要がある。
        - 時間
        - 再現動作(スクショや動画)
        - キーとなる情報(伝票番号、作業者)
        - リクエストパラメータ(入力情報)
        - その他特定度の高いデータ、特異度の高い情報

- HTTPステータスコード
    - 200・・一般的なリクエスト成功
    - 201・・リソースが作成された場合の成功(200で代用することもあるかと思います。)
    - 400(422)・・リクエスト不正。バリデーションなどリクエストパラメータ不正の場合などが一般的。 
        - 400:リクエスト自体が無効。(jsonが壊れているなど)
        - 422:一般的なバリデーションエラー
    - 401・・認証エラー。主にBasic認証などAPIに到達までにNGになるパターン。
    - 403・・リクエストによる操作権限がない(許可されていない)ときにつかわれる。 認証(401)と認可(403)の違いに注意！   
        - 認証・・そもそもユーザーとしてログインできないなど、ユーザー自体の確認などに使われる 
        - 認可・・特定動作の権限があるかないかのケースで使われる
    - 404・・NotFound。リソースがない。
    - 500・・異常