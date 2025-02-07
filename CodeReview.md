## コードレビュー

## コードレビューのポイント
- 規約
    - 規則的にチェックできるものなので、レビューで人がチェックしないようにする
        - PHP_CodeSniffer(PSR-2)
    - 基本はエディタでチェックを行う
    - CICDでもチェックを行う(規約をやぶっている場合はcommitできないなど)
- コメント
    - どれくらいの情報を詰めるかはプロジェクトルール遵守を
    - あえてしない・・・という選択肢もあり→その分、命名に注力すべき
    - ドキュメント生成や構文チェックにつなげられることも・・・
- クラス、メソッド、引数、インデントの数、横幅の長さ
    - メソッド 30行以内 
    - クラス 100行以内
    - 引数 5つ以内
    - インデント 3段階までがMAX
    - ポイントとしては**責務の分離**ができているかないか
- 特定メソッドの推奨
    - パフォーマンスの向上など
    - stringは参照系、StringBuilderは更新系
    ```java
    // Stringを使用した例
    String str = "Hello";
    str += " World"; // 新しいStringオブジェクトが生成される

    // StringBuilderを使用した例
    StringBuilder sb = new StringBuilder("Hello");
    sb.append(" World"); // 既存のStringBuilderオブジェクトが変更される
    String result = sb.toString(); // StringBuilderをStringに変換
    ```

- 命名
    - 役割分担ができているか、必要な情報が込められているか
    - なるべくシンプルかつ統一性のある命名にする
    - 品詞の統一
    - 汎用的になりすぎない
    - メソッドに関して重複をしない

- 責務の分離が保たれているか https://zenn.dev/mpyw/articles/ce7d09eb6d8117
    - Requests: ユーザーやクライアントからの入力データを表すオブジェクトを格納します。バリデーションやサニタイズなど、リクエストデータの前処理もここで行います。
    - Controller: リクエストを受け取り、適切なサービスやユースケースを呼び出してレスポンスを生成する役割を持ちます。ユーザーインターフェースとの仲介役です。
    - Service: ビジネスロジックを実装するためのクラスや関数を格納します。複数のコントローラから再利用されることが多く、アプリケーションの主要な機能を担います。
    - Infra (Infrastructure): データベース接続や外部サービスとの連携など、インフラストラクチャ関連のコードを含みます。リポジトリパターンやAPIクライアントなどがここに配置されます。
    - ValueObject: 一度作成されるとその状態が変更されないオブジェクトを格納します。ドメイン駆動設計（DDD）で使用され、特定の属性や値を持つオブジェクトです。
    - Model: データベースのテーブルに対応するクラスを格納します。データの保存、取得、更新、削除を行うメソッドを含みます。
    - ViewModel: ビュー（UI）に表示するためのデータや状態を管理します。コントローラから受け取ったデータを整形し、ビューに渡す役割を持ちます。
    - UseCase: アプリケーションの特定のユースケースやシナリオを実装するクラスや関数を格納します。ユーザーが実行する特定のアクションに関連するビジネスロジックを含みます。
- 責務の分離→テストしやすさにつながる
- 可読性
    - リテラルは定数やenumを使う・・可読性と値の担保
    - 業務で意味のある値に関してはValueObjectを使用する https://skill-up-engineering.com/2022/12/10/post-6195
    - 構造がわからない値を使わない・・連想配列など
    ```php
    // NG
    $userData = [
        'name' => 'Alice',
        'email' => 'alice@example.com',
        'status' => 'active'
    ];

    $user = new User($userData);

    //OK
    $userData = new UserData('Alice', 'alice@example.com', 'active');
    $user = new User($userData);
    ```
    - まとめすぎない
    ```php
    // NG
    function resist($person_array);

    //OK
    function resist(
        $name,
        $age,
        $email,
        $address,
        $tel
    )
    ```
    - 共通化か可読性か
        - 共通化は一般的なコードの量を減らすことができるが、可読性が下がる <br>https://skill-up-engineering.com/2022/12/10/post-6205

- 型のチェックによる値の担保
  - empty禁止(0,false,null,空文字,空配列が比較できてしまう)、二重イコール禁止(数字と文字の比較ができてしまう)
```php
   // NG
   $userData = [
　　　'gender' => '男',
     'email' => 'yamada@test.com',
     'last_name' => '山田',
     'first_name' => '太郎',
     'tel' => '08012345678',
   ];
   $user = new App\Models\User();
   $user->fill($userData);

   // OK
   $user = new App\Models\User();
   $user->setGender('男');
   $user->setEmail('yamada@test.com');
   $user->setLastName('山田');
   $user->setFirstName('太郎');
   $user->setTel('08012345678');
   $user->save();
```
- 負荷
    - chunk、cursorなどを使用してメモリ消費量を抑える
- エラー処理
    - 適切なエラーハンドリング
       - トランザクションスコープ
       - 業務系のエラーか500系のエラーか
       - 後続処理への影響

## レビューを円滑にすすめるために
- レビュアーとレビュイーの振り分け
    - レビュイーがレビュアーになる
- 細かく分ける
    - 
- 指摘に関して重さを分ける(must:必ず直す、should:直すべし、better:直した方が良い、want:直してほしい)
- 類型化とノウハウの共有
- リスペクトのコミニケーション
- 神学論争的なものは責任者が決める
- 生成AIの活用例
