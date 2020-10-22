# JSON

## 01. データ記述言語

### データ記述言語の種類

#### ・JSON：JavaScript Object Notation

一番外側を波括弧で囲う．

```json
{
  "Example": {
    "fruit": ["ばなな", "りんご"],
    "account": 200
  }
}
```

#### ・YAML：YAML Ain't a Markup Language

```yaml
{
  Example:
    fruit:
      - "ばなな"
      - "りんご"
    account: 200
}  
```

#### ・マークアップ言語

マークアップ言語の章を参照せよ．

#### ・CSV：Comma Separated Vector

データ解析の入力ファイルとしてよく使うやつ．

<br>

### JSONの定義方法

#### ・JavaScript

**＊実装例＊**

```javascript
// どんなデータを含むJSONなのかわかりやすい方法
const json = {
    name: null,
    age: null,
    tel: null
}

json.name = "taro";
json.age = 30;
json.tel = "090-0123-4567";
```

**＊実装例＊**

```javascript
const json = {}

// areaというキー名の値を追加
json.prefecture = "Tokyo";

// 以下は．undefined になる．二段階の定義はできない．
//// json.prefecture.area = "Shibuya";
```

**＊実装例＊**

```javascript
const json = {
    name: "taro",
    age: 30,
    tel: "090-0123-4567"
}

// areaというキー名の値を追加
json.prefecture = "Tokyo";

// もしくは，
//// json["prefecture"] = "Tokyo";
```

#### ・PHP

<br>

## 02. 相互パース（シリアライズ＋デシリアライズ）

### シリアライズ，デシリアライズとは

クライアントサイドとサーバサイドの間で，JSON型オブジェクトデータを送受信できるように解析（パース）することを，シリアライズまたはデシリアライズという．

![シリアライズとデシリアライズ](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/シリアライズとデシリアライズ.png)

<br>

### 各形式のオブジェクトの比較

#### ・JavaScript型オブジェクト

PHPでいう連想配列と同じような形をしている．キー名はクオーテーションで囲う必要が無い．

**＊実装例＊**

```javascript
// クラス宣言．
class Example {
    fruit: ["ばなな", "りんご"];
    account: 200;
}

// 外部ファイルから読み込めるようにする．  
module.exports = Example;  
```

#### ・JSON型オブジェクト

**＊実装例＊**

文字列のみ，シングルクオーテーションかダブルクオーテーションで囲う．キー名も値も，クオーテーションで囲う必要が無い．

```json
// 一番外側を波括弧で囲う．
{
  "Example": {
    "fruit": ["ばなな", "りんご"],
    "account": 200
  }
}
```

#### ・PHP型オブジェクト

**＊実装例＊**

```PHP
<?php
class Example
{
    private $fruit;
    
    private $account;
}    
```

<br>

### JavaScriptとJSONの相互パース

#### ・JavaScriptからJSONへのシリアライズの場合

```javascript
// ここに実装例
```

#### ・JSONからJavaScriptへのデシリアライズの場合

```javascript
// ここに実装例
```

#### ・相互パース可能なクラスの場合

**＊実装例＊**

以下でシリアライズとデシリアライズを行うクラスを示す．

```javascript
class StaffParser {

    // デシリアライズによるJavaScript型データを自身に設定
    constructor(properties) {
        this.id   = properties.id;
        this.name = properties.name;
    }
  
  
    //-- デシリアライズ（JSONからJavaScriptへ） --//
    static deserializeStaff(data) {
       // JSON型オブジェクトから値を，JavaScript型オブジェクトに取り出す
       return new StaffParser({
            id: data.id,
            name: data.name
       });
    }
  
  
    //-- シリアライズ（JavaScriptからJSONへ） --//
    static serializeCriteria(criteria) {
        // JavaScript上でのJSON型変数の定義方法
        const query = {}
        // ID
        if (criteria.id) {
          // JSON型オブジェクトが生成される．
          query.id = _.trim(criteria.id);
        }
        // 氏名
        if (criteria.name) {
          query.name = _.trim(criteria.name);
        }
    }
}     
```

<br>

### PHPとJSONの相互パース

#### ・PHPからJSONへのシリアライズの場合

```php
// ここに実装例
```

#### ・JSONからPHPへのデシリアライズの場合

```php
// ここに実装例
```

<br>

## 03. JSONのクエリ言語

### クエリ言語の種類

#### ・JMESPath

**＊実装例＊**

```javascript
// ここに実装例
```

