[OCI] Oracle Functionsから Autonomous Databaseに接続
https://blogs.oracle.com/developers/oracle-functions-connecting-to-an-atp-database-revisited

事前準備
- Autonomous Dataabseの作成
    オプション）sample.sqlで表、データを作成
- 接続ウォレットのダウンロード
- Object Storage バケットの作成
- 入手したウォレットzipファイルを解凍し、すべてのファイルをObject Storageにアップロード
- Functions 実行環境の整備（Cloud Shellでも可）

ステップ.1 Oracle Functions用のリソースプリンシパルの構成
- 動的グループの作成
    Functionsに使用するコンパートメント用に動的グループを作成
    一致ルール
    ALL{resource.type='fnfunc', resource.compartment.id='ocid1.compartment.oc1..aaaaaaaa ..."}
- 動的グループにポリシーの割り当て
    ウォレットファイルが保存されているバケットを読み取れるように、動的グループにポリシーを割り当て
    allow dynamic-group XXX to manage objects in compartment YYY where target.bucket.name='wallet'

ステップ.2 Functionの作成・動作確認
- アプリケーションの作成
    fn create app oci-adb-jdbc-java-app --annotation oracle.com/oci/subnetIds='["ocid1.subnet.oc1.phx..."]'
- アプリケーションに構成を追加
    Object Storageのネームスペースとバケット名
        fn config app oci-adb-jdbc-java-app NAMESPACE [name]
        fn config app oci-adb-jdbc-java-app BUCKET_NAME [name]
    ADBへの接続用のDBユーザ名・パスワード・接続文字列
        fn config app oci-adb-jdbc-java-app DB_USER [user]
        fn config app oci-adb-jdbc-java-app DB_PASSWORD [password]
        fn config app oci-adb-jdbc-java-app DB_URL jdbc:oracle:thin:\@[tns name]\?TNS_ADMIN=/tmp/wallet
    実行するSQL文
        fn config app ziptodd SQL_TEXT "select * from employees"
- ファンクションの作成/テストディレクトリの削除
    fn init --runtime java oci-adb-jdbc-java
    cd oci-adb-jdbc-java
    rm -r src/test/

    ファンクションの作成
    - pom.xmlファイル
    - func.yamlファイル
    - HelloFunction.javaファイル
    の編集（本ファイルに置き換え）
- ファンクションのデプロイと呼び出し
    fn deploy --app oci-adb-jdbc-java-app
    fn invoke oci-adb-jdbc-java-app oci-adb-jdbc-java

