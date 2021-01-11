# ドメイン駆動設計

## 01. MVC

### MVCとは

ドメイン駆動設計が考案される以前，MVCの考え方が主流であった．

![MVCモデル](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/MVCモデル.png)

<br>

### MVCからドメイン駆動設計への発展

#### ・MVCの問題点

しかし，特にModelの役割が抽象的過ぎたため，開発規模が大きくなるにつれて，Modelに役割を集中させ過ぎてしまうことがあった．

#### ・MVCからDDDへの移行

ドメイン駆動設計が登場したことによって，MVCは発展し，M・V・Cそれぞれの役割がより具体的で精密になった．Modelの肥大化は，ModelがもつビジネスロジックをDomain層，またCRUD処理をInfrastructure層として分割することによって，対処された．

<br>

## 02. ドメイン駆動設計とは

### ビジネスルールに対するオブジェクト指向分析と設計

#### ・ドメインエキスパート，ユビキタス言語とは

ドメインエキスパート（現実世界のビジネスルールに詳しく，また実際にシステムを使う人）と，エンジニアが話し合いながら，ビジネスルールに対して，オブジェクト指向分析と設計を行っていく．この時，ドメインエキスパートとエンジニアの話し合いに齟齬が生まれぬように，ユビキタス言語（業務内容について共通の用語）を設定しておく．

#### ・戦略的設計の手順

1. ドメインエキスパートと話し合い，現実世界の業務内容に含まれる『名詞』と『振舞』に着目．
2. 『名詞』と『振舞』を要素として，オブジェクト指向分析と設計を行い，EntityやValue Objectを抽出していく．
3. EntityやValue Objectを用いて，ドメインモデリング（オブジェクト間の関連付け）を行う．

#### ・戦術的設計の手順

戦略的設計を基に，各オブジェクトとオブジェクト間の関連性を実装していく．

![DDDの概念](https://user-images.githubusercontent.com/42175286/61179612-d305c800-a640-11e9-8c4a-3d31225af633.jpg)

<br>

### ドメイン駆動設計の派生型

現在までに，ドメイン駆動設計の派生型がいくつか提唱されている．

#### ・Layeredアーキテクチャ

#### ・Hexagonalアーキテクチャ

#### ・Onionアーキテクチャ

  <br>

## 02-02. Layeredアーキテクチャ型ドメイン駆動設計

### 責務の分担方法

![ドメイン駆動設計](https://user-images.githubusercontent.com/42175286/58724663-2ec11c80-8418-11e9-96e9-bfc6848e9374.png)

Layeredアーキテクチャ型ドメイン駆動設計において，MVCは，以下の4層に再編成できる．

#### ・User Interface層

#### ・Application層

| デザインパターン    | 説明 | 備考 |
| ------------------- | ---- | ---- |
| Controller          |      |      |
| Application Service |      |      |
| Validation          |      |      |
| Converter           |      |      |

#### ・Domain層（ビジネロジックをコード化）

| デザインパターン | 説明 | 備考 |
| ---------------- | ---- | ---- |
| Entity           |      |      |
| Specification    |      |      |
| Value Object     |      |      |
| Type Code        |      |      |
| Domain Service   |      |      |

#### ・Infrastructure層（DBとマッピング）

| デザインパターン | 説明 | 備考 |
| ---------------- | ---- | ---- |
| Repository       |      |      |
| Factory          |      |      |

<br>

### DIPに基づくドメイン駆動設計

#### ・依存性を逆転させる方法

1. Repositoryの抽象クラスを，より上位のドメイン層に配置する．
2. Repositoryの実装クラスを，より下位のInfrastructure層に配置する．
3. 両方のクラスに対して，バインディング（関連付け）を行い，抽象クラスをコールした時に，実際には実装クラスがコールされるようにする．
4. これらにより，依存性が逆転する．依存性逆転の原則に基づくことによって，ドメイン層への影響なく，Repositoryの交換が可能になる．

![ドメイン駆動設計_逆転依存性の原則](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_依存性逆転の原則.jpg)

#### ・Domain層とInfrastructure層のバインディング

```php
<?php

// ここに実装例
```

<br>


## 03. Application層（UseCase層）

### Controller

#### ・責務

ビジネスの仕組みを表しているわけではなく，以外のロジックである．CRUDのReadの場合，以下のような処理手順を組み合わせて，Use case（使用事例）を実装する．

1. 最初に，リクエストによるJSON型データ送信を受け取る．
2. JSON型データを連想配列にパースする．
3. Repositoryからメソッドをコールし，連想配列を渡してDBからオブジェクトデータをReadする．
4. Readしたオブジェクトデータを連想配列に変換する．
5. 最後に，連想配列をJSON型データにパースし，JavaScriptに送信する．

![シリアライズとデシリアライズ](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/シリアライズとデシリアライズ.png)

**＊実装例＊**

```php
<?php

class AcceptOrdersController
{
    // 単なるメソッドではなく，Use caseとなるようなメソッド
    public function acceptOrders()
    {
    
    }

}  
```

**＊ユースケース例＊**

オンラインショッピングにおけるUse case

![ユースケース図](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ユースケース図.png)

<br>

### Application Service

#### ・責務

Application層の中で，ドメイン層のオブジェクトを使用する汎用的なメソッドが切り分けられたもの．Controllerにメソッドを提供する．Domain層のDomain Serviceとは異なるので注意する．

**＊実装例＊**

```php
<?php

class SlackNotificationService
{
    public function notify(SlackMessage $message)
    {
       // SlackのAPIにメッセージを送信する処理
    }
}
```

<br>

### 入力データに対するフォーマットのValidationパターン

#### ・責務

デザインパターンの一つ．フロントエンドからサーバサイドに送信されてきたデータのフォーマットを検証する責務を持つ．例えば，UserInterface層からApplication層に送信されてきたJSON形式データのフォーマットを検証する．

**＊実装例＊**

```php
<?php

use Respect\Validation\Validator; // Validationのライブラリ

class FormatValidator
{
    // 日時データのフォーマットを検証する．
    public function validateFormat($dateTime)
    {
        if(empty($dateTime)) {
            return false;
        }  
      
        if(!Validator::date(\DateTime::ATOM)->validate($dateTime)) {
            return false;
        }
      
        return true;
    }
}  
```

<br>

### Converterパターン

#### ・責務

デザインパターンの一つ．データ構造を変換する責務を持つ．例えば，Application層からUserInterface層へのデータのレスポンス時に，送信するオブジェクトデータ（Route Entity）を連想配列に変換する．

**＊実装例＊**

```php
<?php

class Converter
{
    // オブジェクトを連想配列に詰め替える処理．
    public function convertToArray(XxxEntity $xxxEntity)
    {
        $xxxArray['id'] = $xxxEntity->id;
        $xxxArray['name'] = $xxxEntity->name;
        $xxxArray['email'] = $xxxEntity->email;
    }
}  
```

<br>


## 04. Domain層

### Repositoryパターン（インターフェース）

![Repository](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/Repository.png)

#### ・責務

デザインパターンの一つ．データベースにアクセスする責務を持つ．リクエストによるデータ送信が行われる．Controllerは，Domain層の抽象メソッドをコールし，DBにおけるデータのCRUDを行う．DIPに基づくドメイン駆動設計の場合，Repositoryのインターフェースを配置する．

**＊実装例＊**

```php
<?php

interface GetDogToyEntityRepository
{

    // 対応する具象メソッドはInfrastructure層のRepositoryに実装．
    function arrayDogToyEntities();

}
```

<br>

### Entity

#### ・責務

以降の説明を参照．

<br>

### Specificationパターン

#### ・責務

デザインパターンの一つ．ビジネスルールの検証，検索条件オブジェクトの生成は、EntitiyやValue Objectに持たせた場合，肥大化の原因となり，可読性と保守性が悪い．そこで，こういったビジネスルールをSpecificationオブジェクトにまとめておく．

#### ・入力データに対するビジネスルールのValidation

真偽値メソッド（```isXxxx```メソッド）のように，オブジェクトのデータを検証して、仕様を要求を満たしているか、何らかの目的のための用意ができているかを調べる処理する．

**＊実装例＊**

```php
<?php

class XxxSpecification
{
    public function isSatisfiedBy($XxxEntity)
    {
       // ビジネスルールのバリデーション処理．
    }
} 
```

#### ・検索条件オブジェクトの生成

リクエストのパスパラメータとクエリパラメータを引数として，検索条件のオブジェクトを生成する．ビジネスルールのValidationを行うSpecificationクラスと区別するために，Criteriaオブジェクトという名前としても用いられる．

**＊実装例＊**

```php
<?php

class XxxCriteria
{
    private $id;
  
    private $name;
  
    private $email;
  
    // 検索条件のオブジェクトを生成．
    public function build(array $array)
    {
        // 自身をインスタンス化．
        $criteria = new static();
        
        if(isset($array['id'])) {
            $criteria->id = $array['id'];
        }
      
        if(isset($array['name'])) {
            $criteria->id = $array['name'];
        }
      
        if(isset($array['email'])) {
            $criteria->id = $array['email'];
        }
      
        return $criteria;
    }  

}
```

<br>

### Value Object

#### ・責務

以降の説明を参照．

<br>

### Type Code（標準型）

#### ・責務

以降の説明を参照．

<br>

### Domain Service

#### ・責務

以降の説明を参照．

<br>

## 04-02. Domain層｜Entity の責務

### Entityの具体例

![ドメイン駆動設計_エンティティ](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_エンティティ.jpg)

<br>

### 保持するデータの値が一定でない

状態を変化させる必要があるデータをもつ

<br>

### データの値が同じでも区別できる

オブジェクトにアイデンティティがあり，他のオブジェクトと同じ属性をもっていても，区別される．

**＊実装例＊**

```php
<?php

class ToyOrderEntity
{
    // 犬用おもちゃ
    private $dogToyEntity;
    
    // 猫用おもちゃ
    private $catToyEntity;
    
    // Setterを実装
    public function __construct
    (
        DogToyEntity $dogToyEntity,
        CatToyEntity $catToyEntity
    )
    {
        $this->dogToyEntity = $dogToyEntity;
        $this->catToyEntity = $catToyEntity;
    }
  
    // 	Getterを実装
    public function getXXX()
    {
        //  Read処理;
    }  
  
}
```

<br>

### Route Entityとは

![ドメイン駆動設計_集約関係](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_集約関係.jpg)

 EntityやValue Objectからなる集約の中で，最終的にアプリケーション層へレスポンスされる集約を，『RouteEntity』という．

**＊実装例＊**

```php
<?php

class DogToyEntity
{
    // おもちゃタイプ
    private $toyType;
    
    // おもちゃ商品名
    private $toyName;
    
    // 数量
    private $number;
    
    // 価格VO
    private $priceVO;
    
    // 色VO
    private $colorVO;
    
    
    // Setterを実装
    public function __construct
    (
        int $toyType,
        string $toyName,
        int $number,
        priceVO $priceVO,
        ColorVO $colorVO
    )
    {
        $this->toyType = $toyType;
        $this->toyName = $toyName;
        $this->number = $number;
        $this->priceVO = $priceVO;
        $this->colorVO = $colorVO;
    }
        
    // Getterを実装
    public function toyNameWithColor()
    {
        return sprintf(
            '%s（%s）',
            $this->toyName,
            $this->colorVO->colorName()
        );
    }
    
}
```

<br>

## 04-03. Domain｜Value Object の責務

### Value Objectの具体例

金額，数字，電話番号，文字列，日付，氏名，色などのユビキタス言語に関するデータと，これを扱うメソッドを実装する場合，一意で識別できるデータ（例えば，```$id```データ）をもたないオブジェクトとして，これらの実装をまとめておくべきである．このオブジェクトを，Value Objectという．

![ドメイン駆動設計_バリューオブジェクト](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_バリューオブジェクト.jpg)

**＊実装例＊**

```php
<?php

/**
 * 支払情報オブジェクト
 */
class PaymentInfoVO
{
    // 予め実装したImmutableObjectトレイトを用いて，データの不変性を実現
    use ImmutableObject;

    /**
     * 支払いID
     *
     * @var PaymentId
     */
    private $PaymentId;

    /**
     * 支払い方法
     *
     * @var PaymentType
     */
    private $PaymentType;
    
    /**
     * 連絡先メールアドレス
     *
     * @var string|null
     */
    private $contactMail;
    
    /**
     * 金額
     *
     * @var Money
     */
    private $price;    
}
```

<br>

### 一意に識別できるデータをもたず，対象のユビキタス言語に関するデータをメソッドを持つ

#### ・金額データと計算

金額データの計算をController内処理やEntity内メソッドで行うのではなく，金額計算を行うValue Objectのメソッドとして分割する．

#### ・所要時間データと計算

所要時間データの計算をController内処理やEntity内メソッドで行うのではなく，所要時間計算を行うValue Objectのメソッドとして分割する．

#### ・住所データと処理

郵便番号データとその処理をValue Objectとして分割する．

<br>

### オブジェクトの持つデータの不変性

#### ・不変性に関するベストプラクティス

EntityとValue Objectのどちらとして，オブジェクトをモデリング／実装すべきなのかについて考える．そもそも，大前提として，『オブジェクトの持つデータはできるだけ不変にすべき』というベストプラクティスがあり，その結果，Value Objectというが生まれたと考えられる．実は，Value Objectを使わずに全てEntityとしてモデリング／実装することは可能である．しかし，不変にしてもよいところも可変になり，可読性や信頼性を下げてしまう可能性がある．

#### ・普遍性をコードで実現する方法

インスタンス化時に自動的に呼び出される```__construct()```を用いる．インスタンス化時に実行したい処理を記述できる．Setterを持たせずに，```__construct()```でのみ値の設定を行えば，Value Objectのような，『Immutable』なオブジェクトを実現できる．

**＊実装例＊**

```php
<?php

class ExampleVO
{
    
    private $propertyA;
    
    private $propertyB;
    
    private $propertyC;
    
    // コンストラクタで$propertyに値を設定
    public function __construct($param)
    {
        $this->propertyA = $param['a'];
        $this->propertyB = $param['b'];
        $this->propertyC = $param['c'];
    }   
}
```

#### ・『Immutable』を実現できる理由

Test01クラスインスタンスの```$property01```に値を設定するためには，インスタンスからSetterを呼び出す．Setterは何度でも呼び出せ，その度にデータの値を上書きできてしまう．

**＊実装例＊**

```php
<?php

$test01 = new Test01;

$test01->setProperty01("データ01の値");

$test01->setProperty01("新しいデータ01の値");
```

一方で，Test02クラスインスタンスの```$property02```に値を設定するためには，インスタンスを作り直さなければならない．つまり，以前に作ったインスタンスの```$property02```の値は上書きできない．Setterを持たせずに，```__construct()```だけを持たせれば，『Immutable』なオブジェクトとなる．

**＊実装例＊**

```php
<?php

$test02 = new Test02("データ02の値");

$test02 = new Test02("新しいデータ02の値");
```

<br>

### 概念的な統一体

```php
<?php

// ここに実装例
```

<br>

### オブジェクトの交換可能性

オブジェクトが新しくインスタンス化された場合，以前に同一オブジェクトから生成されたインスタンスから新しく置き換える必要がある．

<br>

### オブジェクト間の等価性

全てのデータの値が他のVOと同じ場合，同一のVOと見なされる．

 <br>

### メソッドによってオブジェクトの状態が変わらない

**＊実装例＊**

```php
<?php

// （1）ドメイン層の氏名を扱うVO
class NameVO
{
        // （2）予め実装したImmutableObjectトレイトを用いて，データの不変性を実現
    use ImmutableObject;

    // 苗字データ
    private $lastName;
    
    // 名前データ
    private $firstName;
    
    // （6） メソッドによってオブジェクトの状態が変わらない
    public function fullName(): string
    {
        return $this->lastName . $this->firstName;
    }
    
    // 
    protected static function computedPropertyNames()
    {
        return [
            'fullName'
        ];
    }
}
```

**＊実装例＊**

同様に，Immutableトレイトを基に，VOを生成する．

```php
<?php

// ドメイン層の金額を扱うVO
class Money
{

}
```

<br>

## 04-04. Domain層｜Type Code（標準型）の責務

### 区分や種類のデータを保持する

Type Codeは概念的な呼び名で，実際は，標準的なライブラリとして利用できるEnumクラスに相当する．一意に識別する必要がないユビキタス言語の中でも，特に『区分』や『種類』などは，Value Objectとしてではなく，Enumクラスとしてモデリング／実装する．

<br>

### Enumクラスを用いたType Codeの実装

**＊実装例＊**

```php
<?php

class ColorVO extends Enum
{
    const RED = '1';
    const BLUE = '2';


    // 『self::定数名』で，定義の値へアクセスする．
    private $defs = [
        self::RED => ['color_name' => 'レッド'],
        self::BLUE => ['color_name' => 'ブルー']
    ];


    // 色値データ
    private $colorValue;
    
    
    // 色名データ．
    private $colorName;
    
    
    // インスタンス化の時に，『色の区分値』を受け取る．
    public function __construct
    (
        String $value
    )
    {
        // $kbnValueに応じて，色名をcolornameデータにセットする．
        $this->colorValue = $value;
        $this->colorname = $this->defs[$value]['color_name'];
    }
    
    
    // constructによってセットされた色値を返すメソッド．
    public function colorValue() :int
    {
        return $this->colorValue;
    } 


    // constructによってセットされた色名を返すメソッド．
    public function colorName() :string
    {
        return $this->colorName;
    } 
    
}
```

<br>

## 04-05. Domain Service

要勉強．

<br>


## 05. Infrastructure層

### Repositoryパターン（実装クラス）

#### ・DBに対する書き込み責務（Create，Update，Delete）

DBに対する書き込み操作を行う．

1. GETまたはPOSTによって，アプリケーション層から値が送信される．

2. Factoryによって，送信された値からEntityやValue Objectを構成する．さらに，それらから集約を構成する．

3. Repositoryによって，最終的な集約を構成する．

4. Repositoryによって，集約を連想配列に分解する．

5. ```add()```によって，Repositoryクラスのデータに，集約を格納する．

6. ```store()```によって，Transactionクラスのデータに，Repositoryを格納する．

7. DBに対して，書き込みを行う．

   ![ドメイン駆動設計_リポジトリ_データ更新](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_リポジトリ_データ更新.png)

**＊実装例＊**

```php
<?php

// 集約の構成とデータ追加を行う．
class setDogToyEntityRepository
{
    
    // 接続先したいデータベースが設定されたデータ
    private $dbs;
  
    public function setDataSet(Request $request)
    {
  
        $dogToyEntity = new DogToyEntity;
  
  
        // 送信された値を取り出して格納し，集約を生成．
        $dogToyEntity->toyType = $request->xxx();
        $dogToyEntity->toyName	= $request->xxx();
        $dogToyEntity->number = $request->xxx();
        $dogToyEntity->priceVO = $request->xxx(new PriceVO());
        $dogToyEntity->colorVO = $request->xxx(new ColorVO());
  
  
        // 集約を連想配列に分解する．
        $data = [
            'type' => $dogToyEntity->toyType,
            'name' => $dogToyEntity->toyName,
            'number' => $dogToyEntity->number,
            'price' => $dogToyEntity->priceVO->price(),
            'color_value' => $dogToyEntity->colorVO->value(),
        ];
  
  
        // データベースのテーブルに挿入する．
        $this->dbs['app']->insert(dog_toy_table, $data);
    
    }
}
```

#### ・DBに対する読み出し責務（Read）

DBに対する書き込み操作を行う．

1. アプリケーション層から集約がリクエストされる．
2. DBに対して，読み出しを行う．
3. Factoryによって，送信された値からEntityやValue Objectを構成する．さらに，それらから集約を構成する．
4. Repositoryによって，最終的な集約を構成する．
5. 再構成された集約をアプリケーション層にレスポンス．

![ドメイン駆動設計_リポジトリ_データ取得](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ドメイン駆動設計_リポジトリ_データ取得.jpg)

**＊実装例＊**

```php
<?php

// データのReadと集約再構成を行う．
class getDogToyEntityRepository
{
  // 接続先したいデータベースが設定されたデータ
    private $dbs;


    // 連想配列データから『RouteEntity』の集約を構成し，レスポンスする．
    public function arrayDogToyEntities(): DogToyEntities
    {
        $dogToyEntities = [];
        foreach($this->fetchDataSet() as $fetchedData){
            $dogToyEntities[] = $this->aggregateDogToyEntity($fetchedData)
        }
        
        return $dogToyEntities;
    }


    // データベースからデータをReadする．
    private function fetchDataSet()
    {
        $select = [
            'dog_toy.type AS dog_type',
            'dog_toy.name AS dog_toy_name',
            'dog_toy.number AS number',
            'dog_toy.price AS dog_toy_price',
            'dog_toy.color_value AS color_value'
        ];
        
        $query = $this->getFecthQuery($select);
        return $query->getConnection()->executeQuery()->fetchAll(); 
    }


    // 集約を行うメソッド
    private function aggregateDogToyEntity(Array $fetchedData)
    {
        $dogToyEntity = new DogToyEntity;
        $dogToyEntity->toyType = $fetchedData['dog_toy_type'];
        $dogToyEntity->toyName = $fetchedData['dog_toy_name'];
        $dogToyEntity->number = $fetchedData['number'];
        $dogToyEntity->priceVO = new PriceVO($fetchedData['dog_toy_price']);
        $dogToyEntity->colorVO = new ColorVO($fetchedData['color_value']);
    }
  
}
```

<br>

### Factory

#### ・責務

責務として，構成した集約関係を加工して新たな集約を再構成する．

**＊実装例＊**

```php
<?php

// 構成した集約関係を加工して新たな集約を再構成する
class Factory
{
  
    private $factory;

    public function __construct(Factory $factory)
    {
        $this->$factory = $factory;
    }
  
    public function factoryToyOrderEntity()
    {
        if(isset($this->factory)){
            $toyOrderEntity = ;//なんらかの集約処理;
            }
    
        return $toyOrderEntity;
    }
    
}
```
