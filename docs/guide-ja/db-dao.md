データベースアクセスオブジェクト
================================

[PDO](http://www.php.net/manual/ja/book.pdo.php) の上に構築された Yii DAO (データベースアクセスオブジェクト) は、リレーショナルデータベースにアクセスするためのオブジェクト指向 API を提供するものです。
これは、データベースにアクセスする他のもっと高度な方法、例えば [クエリビルダ](db-query-builder.md) や [アクティブレコード](db-active-record.md) の基礎でもあります。

Yii DAO を使うときは、主として素の SQL と PHP 配列を扱う必要があります。
結果として、Yii DAO はデータベースにアクセスする方法としては最も効率的なものになります。
しかし、SQL の構文はデータベースによってさまざまに異なる場合がありますので、Yii DAO を使用するということは、特定のデータベースに依存しないアプリケーションを作るためには追加の労力が必要になる、ということをも同時に意味します。

Yii は下記の DBMS のサポートを内蔵しています。

- [MySQL](http://www.mysql.com/)
- [MariaDB](https://mariadb.com/)
- [SQLite](http://sqlite.org/)
- [PostgreSQL](http://www.postgresql.org/)
- [CUBRID](http://www.cubrid.org/): バージョン 9.3 以上。
- [Oracle](http://www.oracle.com/us/products/database/overview/index.html)
- [MSSQL](https://www.microsoft.com/en-us/sqlserver/default.aspx): バージョン 2008 以上。


## DB 接続を作成する <a name="creating-db-connections"></a>

データベースにアクセスするために、まずは、データベースに接続するために [[yii\db\Connection]] のインスタンスを作成する必要があります。

```php
$db = new yii\db\Connection([
    'dsn' => 'mysql:host=localhost;dbname=example',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
]);
```

DB 接続は、たいていは、さまざまな場所でアクセスする必要がありますので、次のように、[アプリケーションコンポーネント](structure-application-components.md) の形式で構成するのが通例です。

```php
return [
    // ...
    'components' => [
        // ...
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=example',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
    ],
    // ...
];
```

こうすると `Yii::$app->db` という式で DB 接続にアクセスすることが出来るようになります。

> Tip|ヒント: あなたのアプリケーションが複数のデータベースにアクセスする必要がある場合は、複数の DB アプリケーションコンポーネントを構成することが出来ます。

DB 接続を構成するときは、つねに [[yii\db\Connection::dsn|dsn]] プロパティによってデータソース名 (DNS) を指定しなければなりません。
DSN の形式はデータベースによってさまざまに異なります。
詳細は [PHP マニュアル](http://www.php.net/manual/ja/function.PDO-construct.php) を参照して下さい。
下記にいくつかの例を挙げます。

* MySQL, MariaDB: `mysql:host=localhost;dbname=mydatabase`
* SQLite: `sqlite:/path/to/database/file`
* PostgreSQL: `pgsql:host=localhost;port=5432;dbname=mydatabase`
* CUBRID: `cubrid:dbname=demodb;host=localhost;port=33000`
* MS SQL Server (via sqlsrv driver): `sqlsrv:Server=localhost;Database=mydatabase`
* MS SQL Server (via dblib driver): `dblib:host=localhost;dbname=mydatabase`
* MS SQL Server (via mssql driver): `mssql:host=localhost;dbname=mydatabase`
* Oracle: `oci:dbname=//localhost:1521/mydatabase`

ODBC 経由でデータベースに接続しようとする場合は、[[yii\db\Connection::driverName]] プロパティを構成して、Yii に実際のデータベースのタイプを知らせなければならないことに注意してください。
例えば、

```php
'db' => [
    'class' => 'yii\db\Connection',
    'driverName' => 'mysql',
    'dsn' => 'odbc:Driver={MySQL};Server=localhost;Database=test',
    'username' => 'root',
    'password' => '',
],
```

[[yii\db\Connection::dsn|dsn]] プロパティに加えて、たいていは [[yii\db\Connection::username|username]] と [[yii\db\Connection::password|password]] も構成しなければなりません。
構成可能なプロパティの全てのリストは [[yii\db\Connection]] を参照して下さい。

> Info|情報: DB 接続のインスタンスを作成するとき、実際のデータベース接続は、最初の SQL を実行するか、[[yii\db\Connection::open()|open()]] メソッドを明示的に呼ぶかするまでは確立されません。


## SQL クエリを実行する <a name="executing-sql-queries"></a>

いったんデータベース接続のインスタンスを得てしまえば、次の手順に従って SQL クエリを実行することが出来ます。

1. 素の SQL で [[yii\db\Command]] を作成する。
2. パラメータをバインドする (オプション)。
3. [[yii\db\Command]] の SQL 実行メソッドの一つを呼ぶ。

次に、データベースからデータを読み出すさまざまな方法を例示します。

```php
$db = new yii\db\Connection(...);

// 行のセットを返す。各行は、カラム名と値の連想配列。
// 結果が無い場合は空の配列が返される。
$posts = $db->createCommand('SELECT * FROM post')
            ->queryAll();

// 一つの行 (最初の行) を返す。
// 結果が無い場合は false が返される。
$post = $db->createCommand('SELECT * FROM post WHERE id=1')
           ->queryOne();

// 一つのカラム (最初のカラム) を返す。
// 結果が無い場合は空の配列が返される。
$titles = $db->createCommand('SELECT title FROM post')
             ->queryColumn();

// スカラ値を返す。
// 結果が無い場合は false が返される。
$count = $db->createCommand('SELECT COUNT(*) FROM post')
             ->queryScalar();
```

> Note|注意: 精度を保つために、対応するデータベースカラムの型が数値である場合でも、データベースから取得されたデータは、全て文字列として表現されます。

> Tip|ヒント: 接続を確立した直後に実行したい SQL がある場合 (例えば、タイムゾーンや文字セットを設定したい場合) は、[[yii\db\Connection::EVENT_AFTER_OPEN]] ハンドラの中でそれをすることが出来ます。
> 例えば、
>
```php
return [
    // ...
    'components' => [
        // ...
        'db' => [
            'class' => 'yii\db\Connection',
            // ...
            'on afterOpen' => function($event) {
                // $event->sender は DB 接続を指す
                $event->sender->createCommand("SET time_zone = 'UTC'")->execute();
            }
        ],
    ],
    // ...
];
```


### パラメータをバインドする <a name="binding-parameters"></a>

パラメータを持つ SQL から DB コマンドを作成するときは、SQL インジェクション攻撃を防止するために、ほとんど全ての場合においてパラメータをバインドする手法を用いるべきです。
例えば、

```php
$post = $db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status')
           ->bindValue(':id', $_GET['id'])
           ->bindValue(':status', 1)
           ->queryOne();
```

SQL 文において、一つまたは複数のパラメータプレースホルダ (例えば、上記のサンプルにおける `:id`) を埋め込むことが出来ます。
パラメータプレースホルダは、コロンから始まる文字列でなければなりません。
そして、次に掲げるパラメータをバインドするメソッドの一つを使って、パラメータの値をバインドします。

* [[yii\db\Command::bindValue()|bindValue()]]: 一つのパラメータの値をバインドします。
* [[yii\db\Command::bindValues()|bindValues()]]: 一回の呼び出しで複数のパラメータの値をバインドします。
* [[yii\db\Command::bindParam()|bindParam()]]: [[yii\db\Command::bindValue()|bindValue()]] と似ていますが、パラメータを参照渡しでバインドすることもサポートしています。

次の例はパラメータをバインドする別の方法を示すものです。

```php
$params = [':id' => $_GET['id'], ':status' => 1];

$post = $db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status')
           ->bindValues($params)
           ->queryOne();
           
$post = $db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status', $params)
           ->queryOne();
```

パラメータバインディングは [プリペアドステートメント](http://php.net/manual/ja/mysqli.quickstart.prepared-statements.php) によって実装されています。
パラメータバインディングには、SQL インジェクション攻撃を防止する以外にも、SQL 文を一度だけ準備して異なるパラメータで複数回実行することにより、パフォーマンスを向上させる効果もあります。
例えば、

```php
$command = $db->createCommand('SELECT * FROM post WHERE id=:id');

$post1 = $command->bindValue(':id', 1)->queryOne();
$post2 = $command->bindValue(':id', 2)->queryOne();
```

[[yii\db\Command::bindParam()|bindParam()]] はパラメータを参照渡しでバインドすることをサポートしていますので、上記のコードは次のように書くことも出来ます。

```php
$command = $db->createCommand('SELECT * FROM post WHERE id=:id')
              ->bindParam(':id', $id);

$id = 1;
$post1 = $command->queryOne();

$id = 2;
$post2 = $command->queryOne();
```

クエリの実行の前にプレースホルダを変数 `$id` にバインドし、そして、後に続く各回の実行の前にその変数の値を変更していること (これは、たいてい、ループで行います) に着目してください。
このやり方でクエリを実行すると、パラメータの値が違うごとに新しいクエリを実行するのに比べて、はるかに効率が良くすることが出来ます。


### SELECT しないクエリを実行する <a name="non-select-queries"></a>

今までのセクションで紹介した `queryXyz()` メソッドは、すべて、データベースからデータを取得する SELECT クエリを扱うものでした。
データを返さないクエリのためには、代りに [[yii\db\Command::execute()]] メソッドを呼ばなければなりません。
例えば、

```php
$db->createCommand('UPDATE post SET status=1 WHERE id=1')
   ->execute();
```

[[yii\db\Command::execute()]] メソッドは SQL の実行によって影響を受けた行の数を返します。

INSERT、UPDATE および DELETE クエリのためには、素の SQL を書く代りに、それぞれ、[[yii\db\Command::insert()|insert()]]、[[yii\db\Command::update()|update()]]、[[yii\db\Command::delete()|delete()]] を呼んで、対応する SQL を構築することが出来ます。
これらのメソッドは、テーブルとカラムの名前を適切に引用符で囲み、パラメータの値をバインドします。
例えば、

```php
// INSERT (テーブル名, カラムの値)
$db->createCommand()->insert('user', [
    'name' => 'Sam',
    'age' => 30,
])->execute();

// UPDATE (テーブル名, カラムの値, 条件)
$db->createCommand()->update('user', ['status' => 1], 'age > 30')->execute();

// DELETE (テーブル名, 条件)
$db->createCommand()->delete('user', 'status = 0')->execute();
```

[[yii\db\Command::batchInsert()|batchInsert()]] を呼んで複数の行を一気に挿入することも出来ます。
この方法は、一度に一行を挿入するよりはるかに効率的です。

```php
// テーブル名, カラム名, カラムの値
$db->createCommand()->batchInsert('user', ['name', 'age'], [
    ['Tom', 30],
    ['Jane', 20],
    ['Linda', 25],
])->execute();
```

## テーブルとカラムの名前を引用符で囲む <a name="quoting-table-and-column-names"></a>

特定のデータベースに依存しないコードを書くときには、テーブルとカラムの名前を適切に引用符で囲むことが、たいてい、頭痛の種になります。
データベースによって名前を引用符で囲む規則がさまざまに異なるからです。
この問題を克服するために、次のように、Yii によって導入された引用符の構文を使用することが出来ます。

* `[[カラム名]]`: 引用符で囲まれるカラム名を二重角括弧で包む。
* `{{テーブル名}}`: 引用符で囲まれるテーブル名を二重波括弧で包む。

Yii DAO は、SQL に含まれるこのような構文を、対応する適切な引用符で囲まれたカラム名とテーブル名に自動的に変換します。
例えば、

```php
// MySQL では SELECT COUNT(`id`) FROM `employee` という SQL が実行される
$count = $db->createCommand("SELECT COUNT([[id]]) FROM {{employee}}")
            ->queryScalar();
```

### テーブルプレフィックスを使う <a name="using-table-prefix"></a>

あなたの DB テーブルのほとんどが何か共通のプレフィックスを持っている場合は、Yii DAO によってサポートされているテーブルプレフィックスの機能を使うことが出来ます。

最初に、[[yii\db\Connection::tablePrefix]] プロパティによって、テーブルプレフィックスを指定します。

```php
return [
    // ...
    'components' => [
        // ...
        'db' => [
            // ...
            'tablePrefix' => 'tbl_',
        ],
    ],
];
```

そして、あなたのコードの中で、そのテーブルプレフィックスを名前に持つテーブルを参照しなければならないときには、いつでも `{{%テーブル名}}` という構文を使います。
パーセント記号は DB 接続を構成したときに指定したテーブルプレフィックスに自動的に置き換えられます。
例えば、

```php
// MySQL では SELECT COUNT(`id`) FROM `tbl_employee` という SQL が実行される
$count = $db->createCommand("SELECT COUNT([[id]]) FROM {{%employee}}")
            ->queryScalar();
```

## トランザクションを実行する <a name="performing-transactions"></a>

一続きになった複数の関連するクエリを実行するときは、データの整合性を一貫性を保証するために、一連のクエリをトランザクションで囲む必要がある場合があります。
一連のクエリのどの一つが失敗した場合でも、データベースは、何一つクエリが実行されなかったかのような状態へとロールバックされます。

次のコードはトランザクションの典型的な使用方法を示すものです。

```php
$db->transaction(function($db) {
    $db->createCommand($sql1)->execute();
    $db->createCommand($sql2)->execute();
    // ... その他の SQL 文を実行 ...
});
```

上記のコードは、次のものと等価です。

```php
$transaction = $db->beginTransaction();

try {
    $db->createCommand($sql1)->execute();
    $db->createCommand($sql2)->execute();
    // ... その他の SQL 文を実行 ...

    $transaction->commit();

} catch(\Exception $e) {

    $transaction->rollBack();

    throw $e;
}
```

[[yii\db\Connection::beginTransaction()|beginTransaction()]] メソッドを呼んで、新しいトランザクションを開始します。
トランザクションは、変数 `$transaction` に保存された [[yii\db\Transaction]] オブジェクトとして表現されています。
そして、実行されるクエリを `try...catch...` ブロックで囲みます。
全てのクエリの実行が成功した場合には [[yii\db\Transaction::commit()|commit()]] を呼んでトランザクションをコミットします。
そうでなければ、例外がトリガされてキャッチされ、[[yii\db\Transaction::rollBack()|rollBack()]] が呼ばれて、失敗したクエリに先行するクエリがトランザクションの中で行った変更がロールバックされます。


### 分離レベルを指定する <a name="specifying-isolation-levels"></a>

Yii は、トランザクションの [分離レベル] の設定もサポートしています。
デフォルトでは、新しいトランザクションを開始したときは、データベースシステムによって設定された分離レベルを使用します。
デフォルトの分離レベルは、次のようにしてオーバーライドすることが出来ます。

```php
$isolationLevel = \yii\db\Transaction::REPEATABLE_READ;

$db->transaction(function ($db) {
    ....
}, $isolationLevel);
 
// あるいは

$transaction = $db->beginTransaction($isolationLevel);
```

Yii は、最もよく使われる分離レベルのために、四つの定数を提供しています。

- [[\yii\db\Transaction::READ_UNCOMMITTED]] - 最も弱いレベル。ダーティーリード、非再現リード、ファントムが発生しうる。
- [[\yii\db\Transaction::READ_COMMITTED]] - ダーティーリードを回避。
- [[\yii\db\Transaction::REPEATABLE_READ]] - ダーティーリードと非再現リードを回避。
- [[\yii\db\Transaction::SERIALIZABLE]] - 最も強いレベル。上記の問題を全て回避。
分離レベルを指定するためには、上記の定数を使う以外に、あなたが使っている DBMS によってサポートされている有効な構文の文字列を使うことも出来ます。
例えば、PostreSQL では、`SERIALIZABLE READ ONLY DEFERRABLE` を使うことが出来ます。

DBMS によっては、接続全体に対してのみ分離レベルの設定を許容しているものがあることに注意してください。
その場合、すべての後続のトランザクションは、指定しなくても、同じ分離レベルで実行されます。
従って、この機能を使用するときは、相反する設定を避けるために、全てのトランザクションについて分離レベルを明示的に指定しなければなりません。
このチュートリアルを書いている時点では、これに該当する DBMS は MSSQL と SQLite だけです。

> Note|注意: SQLite は、二つの分離レベルしかサポートしていません。すなわち、`READ UNCOMMITTED` と `SERIALIZABLE` しか使えません。
他のレベルを使おうとすると、例外が投げられます。

> Note|注意: PostgreSQL は、トランザクションを開始する前に分離レベルを指定することを許容していません。
すなわち、トランザクションを開始するときに、分離レベルを直接に指定することは出来ません。
この場合、トランザクションを開始した後に [[yii\db\Transaction::setIsolationLevel()]] を呼び出す必要があります。

[分離レベル]: http://ja.wikipedia.org/wiki/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E5%88%86%E9%9B%A2%E3%83%AC%E3%83%99%E3%83%AB

### トランザクションを入れ子にする <a name="nesting-transactions"></a>

あなたの DBMS が Savepoint をサポートしている場合は、次のように、複数のトランザクションを入れ子にすることが出来ます。

```php
$db->transaction(function ($db) {
    // 外側のトランザクション
    
    $db->transaction(function ($db) {
        // 内側のトランザクション
    });
});
```

あるいは、

```php
$outerTransaction = $db->beginTransaction();
try {
    $db->createCommand($sql1)->execute();

    $innerTransaction = $db->beginTransaction();
    try {
        $db->createCommand($sql2)->execute();
        $innerTransaction->commit();
    } catch (Exception $e) {
        $innerTransaction->rollBack();
    }

    $outerTransaction->commit();
} catch (Exception $e) {
    $outerTransaction->rollBack();
}
```


## レプリケーションと読み書きの分離 <a name="read-write-splitting"></a>

多くの DBMS は、データベースの可用性とサーバのレスポンスタイムを向上させるために、[データベースレプリケーション](http://ja.wikipedia.org/wiki/%E3%83%AC%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3#.E3.83.87.E3.83.BC.E3.82.BF.E3.83.99.E3.83.BC.E3.82.B9) をサポートしています。
データベースレプリケーションによって、データはいわゆる *マスタサーバ* から *スレーブサーバ* に複製されます。
データの書き込みと更新はすべてマスタサーバ上で実行されなければなりませんが、データの読み出しはスレーブサーバ上でも可能です。

データベースレプリケーションを活用して読み書きの分離を達成するために、[[yii\db\Connection]] コンポーネントを下記のように構成することが出来ます。

```php
[
    'class' => 'yii\db\Connection',

    // マスタの構成
    'dsn' => 'マスタサーバの DSN',
    'username' => 'master',
    'password' => '',

    // スレーブの共通の構成
    'slaveConfig' => [
        'username' => 'slave',
        'password' => '',
        'attributes' => [
            // 短かめの接続タイムアウトを使う
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // スレーブの構成のリスト
    'slaves' => [
        ['dsn' => 'スレーブサーバ 1 の DSN'],
        ['dsn' => 'スレーブサーバ 2 の DSN'],
        ['dsn' => 'スレーブサーバ 3 の DSN'],
        ['dsn' => 'スレーブサーバ 4 の DSN'],
    ],
]
```

上記の構成は、一つのマスタと複数のスレーブを指定するものです。
読み出しのクエリを実行するためには、スレーブの一つが接続されて使用され、書き込みのクエリを実行するためには、マスタが使われます。
そのような読み書きの分離が、この構成によって、自動的に達成されます。例えば、

```php
// 上記の構成を使って Connection のインスタンスを作成する
$db = Yii::createObject($config);

// スレーブの一つに対してクエリを実行する
$rows = $db->createCommand('SELECT * FROM user LIMIT 10')->queryAll();

// マスタに対してクエリを実行する
$db->createCommand("UPDATE user SET username='demo' WHERE id=1")->execute();
```

> Info|情報: [[yii\db\Command::execute()]] を呼ぶことで実行されるクエリは、書き込みのクエリと見なされ、[[yii\db\Command]] の "query" メソッドのうちの一つによって実行されるその他すべてのクエリは、読み出しクエリと見なされます。
  現在アクティブなスレーブ接続は `$db->slave` によって取得することが出来ます。

`Connection` コンポーネントは、スレーブ間のロードバランス調整とフェイルオーバーをサポートしています。
読み出しクエリを最初に実行するときに、`Connection` コンポーネントはランダムにスレーブを選んで接続を試みます。
そのスレーブが「死んでいる」ことが分かったときは、他のスレーブを試します。
スレーブが一つも使用できないときは、マスタに接続します。
[[yii\db\Connection::serverStatusCache|サーバステータスキャッシュ]] を構成することによって、「死んでいる」サーバを記憶し、[[yii\db\Connection::serverRetryInterval|一定期間]] はそのサーバへの接続を再試行しないようにすることが出来ます。

> Info|情報: 上記の構成では、すべてのスレーブに対して 10 秒の接続タイムアウトが指定されています。
  これは、10 秒以内に接続できなければ、そのスレーブは「死んでいる」と見なされることを意味します。
  このパラメータは、実際の環境に基づいて調整することが出来ます。


複数のマスタと複数のスレーブという構成にすることも可能です。例えば、

```php
[
    'class' => 'yii\db\Connection',

    // マスタの共通の構成
    'masterConfig' => [
        'username' => 'master',
        'password' => '',
        'attributes' => [
            // 短かめの接続タイムアウトを使う
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // マスタの構成のリスト
    'masters' => [
        ['dsn' => 'マスタサーバ 1 の DSN'],
        ['dsn' => 'マスタサーバ 2 の DSN'],
    ],

    // スレーブの共通の構成
    'slaveConfig' => [
        'username' => 'slave',
        'password' => '',
        'attributes' => [
            // 短かめの接続タイムアウトを使う
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // スレーブの構成のリスト
    'slaves' => [
        ['dsn' => 'スレーブサーバ 1 の DSN'],
        ['dsn' => 'スレーブサーバ 2 の DSN'],
        ['dsn' => 'スレーブサーバ 3 の DSN'],
        ['dsn' => 'スレーブサーバ 4 の DSN'],
    ],
]
```

上記の構成は、二つのマスタと四つのスレーブを指定しています。
`Connection` コンポーネントは、スレーブ間での場合と同じように、マスタ間でのロードバランス調整とフェイルオーバーをサポートしています。
一つ違うのは、マスタが一つも利用できないときは例外が投げられる、という点です。

> Note|注意: [[yii\db\Connection::masters|masters]] プロパティを使って一つまたは複数のマスタを構成する場合は、データベース接続を定義する `Connection` オブジェクト自体のその他のプロパティ (例えば、`dsn`、`username`、`password`) は全て無視されます。

既定では、トランザクションはマスタ接続を使用します。そして、トランザクション内では、全ての DB 操作はマスタ接続を使用します。
例えば、

```php
// トランザクションはマスタ接続で開始される
$transaction = $db->beginTransaction();

try {
    // クエリは両方ともマスタに対して実行される
    $rows = $db->createCommand('SELECT * FROM user LIMIT 10')->queryAll();
    $db->createCommand("UPDATE user SET username='demo' WHERE id=1")->execute();

    $transaction->commit();
} catch(\Exception $e) {
    $transaction->rollBack();
    throw $e;
}
```

スレーブ接続を使ってトランザクションを開始したいときは、次のように、明示的にそうする必要があります。

```php
$transaction = $db->slave->beginTransaction();
```

時として、読み出しクエリの実行にマスタ接続を使うことを強制したい場合があります。
これは、`useMaster()` メソッドを使うによって達成できます。

```php
$rows = $db->useMaster(function ($db) {
    return $db->createCommand('SELECT * FROM user LIMIT 10')->queryAll();
});
```

直接に `$db->enableSlaves` を false に設定して、全てのクエリをマスタ接続に向けることも出来ます。


## データベーススキーマを扱う <a name="database-schema"></a>

Yii DAO は、新しいテーブルを作ったり、テーブルからカラムを削除したりなど、データベーススキーマを操作することを可能にする一揃いのメソッドを提供しています。
以下がそのソッドのリストです。

* [[yii\db\Command::createTable()|createTable()]]: テーブルを作成する
* [[yii\db\Command::renameTable()|renameTable()]]: テーブルの名前を変更する
* [[yii\db\Command::dropTable()|dropTable()]]: テーブルを削除する
* [[yii\db\Command::truncateTable()|truncateTable()]]: テーブルの全ての行を削除する
* [[yii\db\Command::addColumn()|addColumn()]]: カラムを追加する
* [[yii\db\Command::renameColumn()|renameColumn()]]: カラムの名前を変更する
* [[yii\db\Command::dropColumn()|dropColumn()]]: カラムを削除する
* [[yii\db\Command::alterColumn()|alterColumn()]]: カラムを変更する
* [[yii\db\Command::addPrimaryKey()|addPrimaryKey()]]: プライマリキーを追加する
* [[yii\db\Command::dropPrimaryKey()|dropPrimaryKey()]]: プライマリキーを削除する
* [[yii\db\Command::addForeignKey()|addForeignKey()]]: 外部キーを追加する
* [[yii\db\Command::dropForeignKey()|dropForeignKey()]]: 外部キーを削除する
* [[yii\db\Command::createIndex()|createIndex()]]: インデックスを作成する
* [[yii\db\Command::dropIndex()|dropIndex()]]: インデックスを削除する

これらのメソッドは次のようにして使うことが出来ます。

```php
// CREATE TABLE
$db->createCommand()->createTable('post', [
    'id' => 'pk',
    'title' => 'string',
    'text' => 'text',
]);
```

テーブルに関する定義情報を DB 接続の [[yii\db\Connection::getTableSchema()|getTableSchema()]] メソッドによって取得することも出来ます。
例えば、

```php
$table = $db->getTableSchema('post');
```

このメソッドは、テーブルのカラム、プライマリキー、外部キーなどの情報を含む [[yii\db\TableSchema]] オブジェクトを返します。
これらの情報は、主として [クエリビルダ](db-query-builder.md) や [アクティブレコード](db-active-record.md) によって利用されて、特定のデータベースに依存しないコードを書くことを助けてくれます。