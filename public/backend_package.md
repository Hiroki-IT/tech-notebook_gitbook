# バックエンドパッケージ

## 01. composerによるパッケージの管理

### composer.jsonファイルの実装

#### ・バージョンを定義

```json
// 個人的に一番おすすめ
// キャレット表記
{
  "require": {
    "foo": "^1.1.1",  // >=1.1.1 and <1.2.0
    "bar": "^1.1",    // >=1.1.0 and <1.2.0
    "hoge": "^0.0.1"  // >=0.0.1 and <0.0.2
  }
}
```

```json
// チルダ表記
{
  "require": {
    "foo": "~1.1.1",  // >=1.1.1 and <2.0.0
    "bar": "~1.1",    // >=1.1.0 and <2.0.0
    "hoge": "~1"      // >=1.1.0 and <2.0.0
  }
}
```

```json
// エックス，アスタリスク表記
{
  "require": {
    "foo": "*",     // どんなバージョンでもOK
    "bar": "1.1.x", // >=1.1.0 and <1.2.0 
    "hoge": "1.X",  // >=1.0.0 and <2.0.0
    "huga": ""      // "*"と同じことになる = どんなバージョンでもOK
  }
}
```

####  ・autoloadの対象に登録した設定を反映

外部ファイルの読み込み時に，```require```メソッドを不要とするファイルを```composer.json```ファイルに登録できる．

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        },
        "classmap": [
            "database/seeds",
            "database/factories"
        ]
    }
}
```

その後，コマンドでこの登録を反映する．

```sh
$ composer dump-autoload
```

<br>

### require

#### ・```composer.json```ファイルにパッケージを追加

パッケージ名を```composer.json```ファイルを書き込む．インストールは行わない．コマンドを使用せずに自分で実装しても良い．

```sh
$ composer require <パッケージ名>:^x.x
```

<br>

### install

#### ・インストール

初めてパッケージをインストールする時，```composer.json```ファイルにあるパッケージを全てインストールする．

```sh
$ composer install 
```

####  ・処理ログを表示する

コマンド処理中のログを表示する

```sh
$ composer install -vvv
```

####  ・--no-devを除いてインストール

require-devタグ内のパッケージは除いてインストール

```sh
$ composer install --no-dev
```

#### ・高速インストール

Composerの配布サイトからインストールする．```prefer-source```オプションを使用するよりも高速でインストールできる．デフォルトでdistを使用するため，実際は宣言しなくても問題ない．

```sh
$ composer install --prefer-dist
```

#### ・開発者用インストール

GitHubのComposerリポジトリからインストールする．Composerの開発者用である．

```sh
$ composer install --prefer-source
```

<br>

### update

#### ・追加インストールと更新

インストールされていないパッケージをインストールする．また，バージョン定義をもとに更新可能なパッケージを更新する．

```sh
$ composer update
```

####  ・処理ログを表示する

コマンド処理中のログを表示する

```sh
$ composer install -vvv
```

####  ・メモリ上限をなくしてインストール

phpのメモリ上限を無しにして，任意のcomposerコマンドを実行する．phpバイナリファイルを使用する．Dockerコンテナ内で実行する場合，設定画面からコンテナのCPUやメモリを増設することもできる．．

```sh
$ COMPOSER_MEMORY_LIMIT=-1 composer update -vvv
```

<br>

### その他のコマンド

#### ・キャッシュを削除

インストール時に生成されたキャッシュを削除する．

```sh
$ composer clear-cache
```

<br>

## 02. アプリケーションによるパッケージの読み込み

### エントリポイントにおける```autoload.php```ファイルの読み込み

パッケージが，```vendor```ディレクトリ下に保存されていると仮定する．パッケージを使用するたびに，各クラスでディレクトリを読み込むことは手間なので，エントリーポイント（```index.php```）あるいは```bootstrap.php```で，最初に読み込んでおき，クラスでは読み込まなくて良いようにする．

**＊実装例＊**

```PHP
<?php
    
require_once realpath(__DIR__ . '/vendor/autoload.php');
```

<br>

## 03. Doctrineパッケージ

### Doctrineとは

RDBの読み込み系／書き込み系の操作を行うパッケージ．他の同様パッケージとして，PDOがある．PDOについては，DBの操作のノートを参照せよ．

<br>

### SQLの定義

#### 1. ```createQueryBuilder```メソッド

https://www.doctrine-project.org/projects/doctrine-dbal/en/2.10/reference/query-builder.html

CRUD処理に必要なSQLを保持し，トランザクションによってSQLを実行する．

**＊実装例＊**

```PHP
<?php
    
// QueryBuilderインスタンスを作成．
$queryBuilder = $this->createQueryBuilder();
```

#### 2. CREATE処理

QueryBuilderクラスにおける```insert```メソッドに，値を設定する．

**＊実装例＊**

```PHP
<?php
    
$queryBuilder
    ->insert('mst_users')
```

#### 3. READ処理

QueryBuilderクラスにおける```select```メソッドに，値を設定する．

**＊実装例＊**

```PHP
<?php
    
$queryBuilder
    ->select('id', 'name')
    ->from('mst_users');
```

#### 4. UPDATE処理

QueryBuilderクラスにおける```update```メソッドに，値を設定する．

**＊実装例＊**

```PHP
<?php
    
$queryBuilder
    ->update('mst_users');
```

#### 5. DELETE処理

QueryBuilderクラスにおける```delete```メソッドに，値を設定する．

**＊実装例＊**

```PHP
<?php
$queryBuilder
    ->delete('mst_users');
```

#### 6. データベースへの接続，SQLの実行 

データベース接続に関わる```getConnection```メソッドを起点として，返り値から繰り返しメソッドを取得し，```fetchAll```メソッドで，テーブルのクエリ名をキーとした連想配列が返される．

**＊実装例＊**

```PHP
<?php
    
// データベースに接続．
$queryBuilder->getConnection()
    // SQLを実行し，レコードを読み出す．
    ->executeQuery($queryBuilder->getSQL(),
          $queryBuilder->getParameters()
    )->fetchAll();
```

<br>

### 読み出し系の操作

#### ・プレースホルダー（プリペアードステートメント）

プリペアードステートメントともいう．SQL中にパラメータを設定し，値をパラメータに渡した上で，SQLとして発行する方法．処理速度が速い．また，パラメータに誤ってSQLが渡されても，これを実行できなくなるため，SQLインジェクションの対策にもなる

**＊実装例＊**

```PHP
<?php
    
use Doctrine\DBAL\Connection;

class DogToyQuery
{
    // READ処理のSQLを定義するメソッド．
    public function read(Value $toyType): Array
    {
        // QueryBuilderインスタンスを作成．
        $queryBuilder = $this->createQueryBuilder();
        
        // SQLの定義
        $queryBuilder->select([
          'dog_toy.type AS dog_toy_type',
          'dog_toy.name AS dog_toy_name',
          'dog_toy.number AS number',
          'dog_toy.price AS dog_toy_price',
          'dog_toy.color_value AS color_value'
        ])
          
          // FROMを設定する．
          ->from('mst_dog_toy', 'dog_toy')
          
          // WHEREを設定する．この時，値はプレースホルダーとしておく．
          ->where('dog_toy.type = :type')
          
          // プレースホルダーに値を設定する．ここでは，引数で渡す『$toyType』とする．
          ->setParameter('type', $toyType);
        
        // データベースに接続．
        return $queryBuilder->getConnection()
          
          // SQLを実行し，レコードを読み出す．
          ->executeQuery($queryBuilder->getSQL(),
            $queryBuilder->getParameters()
          )->fetchAll();
    }
}
```

#### ・データのキャッシュ

読み出し系で取得したデータをキャッシュすることができる．

```PHP
<?php
    
use Doctrine\Common\Cache\FilesystemCache;
use Doctrine\DBAL\Cache\QueryCacheProfile;

class Example
{
    public function find()
    {
        
        // QueryBuilderインスタンスを作成．
        $queryBuilder = $this->createQueryBuilder();
        
        // 何らかのSQLを定義
        $query = $queryBuilder->select()->from()
        
        // キャッシュがある場合，ArrayStatementオブジェクトを格納
        // キャッシュがない場合，ResultCacheStatementを格納
        $statement = $this->connection->executeQuery(
          $query->getSQL(),
          $query->getParameters(),
          $queryParameterTypes(),
          new QueryCacheProfile()
        );
        
        $result = $statement->fetchAll();
        $statement->closeCursor();
        return $result;
    }
}
```

<br>


### 書き込み系の操作

#### ・トランザクション，コミット，ロールバック

![コミットメント制御](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/コミットメント制御.jpg)

RDBの処理用語に相当する```beginTransaction```メソッド，```commit```メソッド，```rollBack```メソッドを用いて，RDBを操作する．

参照：https://www.doctrine-project.org/projects/doctrine-dbal/en/2.10/reference/transactions.html

**＊実装例＊**

```PHP
<?php
    
$conn = new Doctrine\DBAL\Connection

// トランザクションの開始 
$conn->beginTransaction();
try{
    // コミット
    $conn->commit();
} catch (\Exception $e) {
  
    // ロールバック
    $conn->rollBack();
    throw $e;
}
```

<br>

## 04. Carbonパッケージ

### Date型

厳密にはデータ型ではないが，便宜上，データ型とする．タイムスタンプとは，協定世界時(UTC)を基準にした1970年1月1日の0時0分0秒からの経過秒数を表したもの．

| フォーマット         | 実装方法            | 備考                                                         |
| -------------------- | ------------------- | ------------------------------------------------------------ |
| 日付                 | 2019-07-07          | 区切り記号なし、ドット、スラッシュなども可能                 |
| 時間                 | 19:07:07            | 区切り記号なし、も可能                                       |
| 日時                 | 2019-07-07 19:07:07 | 同上                                                         |
| タイムスタンプ（秒） | 1562494027          | 1970年1月1日の0時0分0秒から2019-07-07 19:07:07 までの経過秒数 |

<br>

### ```instance```メソッド 

DateTimeインスタンスを引数として，Carbonインスタンスを作成する．

```PHP
<?php
    
$datetime = new \DateTime('2019-07-07 19:07:07');
$carbon = Carbon::instance($datetime);

echo $carbon; // 2019-07-07 19:07:07
```

<br>

### ```create```メソッド

日時の文字列からCarbonインスタンスを作成する．

**＊実装例＊**

```PHP
<?php
    
$carbon = Carbon::create(2019, 07, 07, 19, 07, 07);

echo $carbon; // 2019-07-07 19:07:07
```

<br>

### ```createFromXXX```メソッド

指定の文字列から，Carbonインスタンスを作成する．

#### ・日時数字から

**＊実装例＊**

```PHP
<?php
    
// 日時数字から，Carbonインスタンスを作成する．
$carbonFromeDate = Carbon::createFromDate(2019, 07, 07);

echo $carbonFromeDate; // 2019-07-07
```

#### ・時間数字から

**＊実装例＊**

```PHP
<?php
    
// 時間数字から，Carbonインスタンスを作成する．
$carbonFromTime = Carbon::createFromTime(19, 07, 07);

echo $carbonFromTime; // 19:07:07
```

#### ・日付，時間，日時フォーマットから

第一引数でフォーマットを指定する必要がある．

**＊実装例＊**

```PHP
<?php
    
// 日付，時間，日時フォーマットから，Carbonインスタンスを作成する．
// 第一引数でフォーマットを指定する必要がある．
$carbonFromFormat = Carbon::createFromFormat('Y-m-d H:m:s', '2019-07-07 19:07:07');

echo $carbonFromFormat; // 2019-07-07 19:07:07
```

#### ・タイムスタンプフォーマットから

**＊実装例＊**

```PHP
<?php
    
// タイムスタンプフォーマットから，Carbonインスタンスを作成する．
$carbonFromTimestamp = Carbon::createFromTimestamp(1562494027);

echo $carbonFromTimestamp; // 2019-07-07 19:07:07
```

<br>

### ```parse```メソッド

日付，時間，日時フォーマットから，Carbonインスタンスを作成する．```createFromFormat```メソッドとは異なり，フォーマットを指定する必要がない．

**＊実装例＊**

```PHP
<?php
    
$carbon = Carbon::parse('2019-07-07 19:07:07')
```

<br>

## 05. Pinqパッケージ

### Pinqとは：Php Integrated Query

配列データやオブジェクトデータに対して，クエリを実行できるようになる．他の同様パッケージとして，Linqがある．

<br>

### ```Traversable::from```メソッド

SQLの```SELECT```や```WHERE```といった単語を用いて，```foreach```のように，配列データやオブジェクトデータの各要素に対して，処理を行える．

**＊実装例＊**

```PHP
<?php
    
use Pinq\Traversable;

class Example
{
    
    public function getData(array $entities)
    {
        
        return [
          'data' => Traversable::from($entities)
            // 一つずつ要素を取り出し，関数に渡す．
            ->select(
              function ($entity) {
                  return $this->convertToArray($entity);
              })
            // indexからなる配列として返却．
            ->asArray(),
        ];
    }
}
```

<br>

## 06. Guzzleパッケージ

### Guzzleパッケージとは

通常，リクエストメッセージの送受信は，クライアントからサーバに対して，Postmanやcurl関数などを使用して行う．しかし，GuzzleパッケージのClientを使えば，サーバから他サーバ（外部のAPIなど）に対して，リクエストメッセージの送受信ができる．

<br>

### Clientインスタンス

#### ・リクエストメッセージをGET送信

**＊実装例＊**

```PHP
<?php
    
$client = new Client();

// GET送信
$response = $client->request("GET", <アクセスしたいURL>);
```

#### ・レスポンスメッセージからボディを取得

**＊実装例＊**

```PHP
<?php
    
$client = new Client();

// POST送信
$response = $client->request("POST", <アクセスしたいURL>);

// レスポンスメッセージからボディのみを取得
$body = json_decode($response->getBody(), true);
```

<br>

## 07. Knp/Snappyパッケージ

###  Knp/Snappyとは

ローカルまたは指定したURLのhtmlファイルから，PDFや画像のファイルを生成するパッケージ．

### ・```generateFromHtml```メソッド

htmlファイルを元にして，ローカルディレクトリにPDFファイルを作成する．

**＊実装例＊**

```PHP
<?php
    
$snappy = new Pdf('/usr/local/bin/wkhtmltopdf');

$snappy->generateFromHtml('example.html', '.../example.pdf');
```

<br>

## 08. Respect/Validationパッケージ

### Respect/Validationとは

リクエストされたデータが正しいかを，サーバサイド側で検証する．フロントエンドからリクエストされるデータに関しては，JavaScriptとPHPの両方によるバリデーションが必要である．

```PHP
<?php
    
// ここに実装例
```



