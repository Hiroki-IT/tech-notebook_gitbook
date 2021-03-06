# API Gatewayインポート機能

## 01. APIGateway拡張機能

#### ・必要なキー

APIGatewayのインポートに当たり，OpenAPIのYAMLファイルにキーを新たに実装する必要がある．

参考：https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-swagger-extensions.html

<br>

### ```x-amazon-apigateway-integration```キー

#### ・```x-amazon-apigateway-integration```キーとは

該当するHTTPメソッドで統合リクエストや統合レスポンスを定義するために ```x-amazon-apigateway-integration```キー が必要である．各項目の説明は以下を参考にせよ．

参考：https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-swagger-extensions-integration.html

各種パラメータのマッピングも可能である．メソッドリクエストから統合リクエストへのマッピングについては，以下を参考にせよ．

参考：https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-swagger-extensions-integration-requestParameters.html

統合レスポンスからメソッドレスポンスへのマッピングについては，以下を参考にせよ．

参考：https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-swagger-extensions-integration-responseParameters.html

#### ・設定項目（VPCリンク&プロキシ統合）

```yaml
paths:
  /users:
    get:
    
      # ～ 省略 ～
    
      #===========================
      # 統合
      #===========================
      x-amazon-apigateway-integration:
        httpMethod: "GET" # 転送するHTTPメソッド
        uri: "http://<NLBのDNS名>/api/v1/users/" # 転送先のバックエンドURL
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'" # 転送するカスタムヘッダーとAPIキー
          integration.request.querystring.userId: method.request.querystring.userId # マッピングするクエリパラメータ
          # パスパラメータ間のマッピングであれば，integration.request.path.userId: method.request.path.userId
          # 他パラメータからボディへのマッピングであれば，integration.request.header.userId: method.request.body.userId
        connectionType: VPC_LINK # VPCリンクを使用
        connectionId: <VPCリンクID> # VPCリンクのID
        passthroughBehavior: when_no_match # プロキシ統合の場合は設定の変更不可で固定
        type: http_proxy # プロキシ統合を使用      
        responses: # プロキシ統合の場合は設定の変更不可で固定
          default:
            statusCode: 200     
```

#### ・設定項目（VPCリンク&非プロキシ統合の場合）

パススルー条件やresponseキー以下の統合レスポンスを設定できる． 

```yaml       
paths:
  /users:
    post:
     x-amazon-apigateway-integration:
        httpMethod: POST
        uri: "http://<NLBのDNS名>/api/v1/users/"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
        requestTemplates:
          application/json: '{"body" : $input.json("$")}'
        passthroughBehavior: when_no_templates # 統合リクエストのマッピングテンプレートのパススルー条件を選択
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http # 非プロキシ統合     
        responses: # 統合レスポンスを設定
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"body" : $input.json("$")}' # レスポンス統合のマッピングテンプレート
          400:
            statusCode: 400      
          401:
            statusCode: 401
```

#### ・設定項目（モック統合）

パススルー条件を設定できる．モックに処理を定義する必要がある

```yaml
paths:
  /users:
    post:
      x-amazon-apigateway-integration:
        type: mock # モック統合を使用
        requestTemplates:
          application/json: '{"statusCode": 200}' # リクエストの処理
        passthroughBehavior: when_no_templates # 統合リクエストのマッピングテンプレートのパススルー条件を選択
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"id": 1}' # レスポンスの処理
          400:
            statusCode: 400
          401:
            statusCode: 401
```

<br>

### ```x-amazon-apigateway-request-validators```キー

#### ・```x-amazon-apigateway-request-validators```キーとは

メソッドリクエストで各種パラメータのバリデーションを定義するために，```x-amazon-apigateway-request-validators```キーが必要である．実際に定義したものを使用する時は，後述の```x-amazon-apigateway-request-validator```キーが必要である．

#### ・設定項目

各種パラメータのいずれをバリデーションの対象とするかを指定したうえで，エイリアス名を定義する．ルートで定義する．

```yaml
paths:
  /users:
    
    # ～ 省略 ～
    
#===========================
# バリデーションセットの定義
#===========================  
x-amazon-apigateway-request-validators:
  本文、クエリ文字列パラメータ、およびヘッダーの検証:
    validateRequestParameters: true # クエリパラメータとヘッダー
    validateRequestBody: true # ボディ
  クエリ文字列パラメータおよびヘッダーの検証:
    validateRequestParameters: true
    validateRequestBody: false
```

<br>

### ```x-amazon-apigateway-request-validator```キー

#### ・```x-amazon-apigateway-request-validator```キーとは

メソッドリクエストで各種パラメータのバリデーションを実行するために，```x-amazon-apigateway-request-validator```キーが必要である．

#### ・設定項目

事前に定義した```x-amazon-apigateway-request-validators```キーの中から，使用するバリデーションのエイリアス名を宣言する．

```yaml
paths:
  /users:
    post:
    
    # ～ 省略 ～
    
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証 # エイリアス名を宣言
      
    # ～ 省略 ～
    
#===========================
# バリデーションセットの定義
#===========================  
x-amazon-apigateway-request-validators:
  本文、クエリ文字列パラメータ、およびヘッダーの検証:
    validateRequestParameters: true # クエリパラメータとヘッダー
    validateRequestBody: true # ボディ
  クエリ文字列パラメータおよびヘッダーの検証:
    validateRequestParameters: true
    validateRequestBody: false
```
<br>

## 02. サンプルYAML

### サンプルについて

#### ・注意点

インポートにあたり，以下に注意する．Swagger EditorでAPIの仕様書のHTMLファイルを確認できる．

参考：https://editor.swagger.io/

 - OpenAPI仕様のバージョン2.0と3.0に対応している．
 - ```x-amazon-apigateway-integration```キーを各HTTPメソッドに定義する．
 - API Gatewayが```security```キーのルート定義に非対応のため，冗長ではあるが，各HTTPメソッドに個別に定義する．
 - リクエストメソッドで受信するAPIキーのヘッダー名は，小文字で「```x-api-key```」以外は設定できない．ただし，統合リクエストで転送する時に付与するヘッダー名は「```X-API-Key```」と設定できる．
 - 統合リクエストでバックエンドに転送するAPIキーは，シングルクオートで囲う必要がある．
 - APIキーの作成は手動で行う必要がある．
- ステージの作成は手動で行う必要がある．
- ```servers```キーの実装はインポートしても反映できない．
- マッピングテンプレートはVTLによる定義が可能である．

#### ・その他非対応な記法

その他の非対応の記述については，以下を参考にせよ．

参考：https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-known-issues.html#api-gateway-known-issues-rest-apis

<br>

### VPCリンク＆プロキシ統合

**実装例**


```yaml
openapi: 3.0.0

info:
  title: example-api-with-proxy-integration # API名
  description: The API for example with proxy integration # APIの説明
  termsOfService: https://www.example.com/terms/ # 利用規約
  contact:
    name: API support # 連絡先名
    url: https://www.example.com/support # 連絡先に関するURL
    email: support@example.com # メールアドレス
  license:
    name: Apache 2.0 # ライセンス
    url: https://www.apache.org/licenses/LICENSE-2.0.html # URL
  version: 1.0.0 # APIドキュメントのバージョン

servers:
  - url: https://{env}.example.com/api/v1
    description: |
    variables:
      env:
        default: stg
        description: API environment
        enum:
          - stg
          - www

paths:
  #===========================
  # pathsオブジェクト
  #===========================
  /users:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: ユーザ情報取得
      description: 全ユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: query # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                Users:
                  User:
                    userId: 1
                    name: Hiroki
              schema:
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                messages: [
                    "不正なリクエストです．"
                ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．     
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        httpMethod: GET
        uri: "http://<NLBのDNS名>/api/v1/users/"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.querystring.userId: method.request.querystring.userId # マッピングするクエリパラメータ
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http_proxy
        passthroughBehavior: when_no_match
        responses:
          default:
            statusCode: 200
    #===========================
    # path itemオブジェクト
    #===========================
    post:
      tags:
        - ユーザ情報作成エンドポイント
      summary: ユーザ情報作成
      description: ユーザ情報を作成する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters: [ ]
      requestBody: # リクエストボディにパラメータを割り当てる．
        description: ユーザID
        content:
          application/json: # MIME type
            example: # リクエストボディ例
              userId: 1
            schema: # スキーマ
              $ref: "#/components/schemas/user" # Userモデルを参照する．
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                userId: 1
              schema:
                $ref: "#/components/schemas/normal" # 正常系モデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．              
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        httpMethod: POST
        uri: "http://<NLBのDNS名>/api/v1/users/"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http_proxy
        passthroughBehavior: when_no_match        
        responses:
          default:
            statusCode: 200
  #===========================
  # pathsオブジェクト
  #===========================
  /users/{userId}:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: 指定ユーザ情報取得
      description: 指定したユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
      #===========================
      # 統合
      #===========================                
      x-amazon-apigateway-integration:
        httpMethod: GET
        uri: "http://<NLBのDNS名>/api/v1/users/{userId}"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.path.userId: method.request.path.userId
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http_proxy
        passthroughBehavior: when_no_match        
        responses:
          default:
            statusCode: 200
    #===========================
    # path itemオブジェクト
    #===========================                
    put:
      tags:
        - ユーザ情報更新エンドポイント
      summary: 指定ユーザ更新
      description: 指定したユーザ情報を更新する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．      
      #===========================
      # 統合
      #===========================               
      x-amazon-apigateway-integration:
        httpMethod: PUT
        uri: "http://<NLBのDNS名>/api/v1/users/{userId}"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.path.userId: method.request.path.userId
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http_proxy
        passthroughBehavior: when_no_match        
        responses:
          default:
            statusCode: 200
#===========================
# バリデーションセットの定義
#===========================  
x-amazon-apigateway-request-validators:
  本文、クエリ文字列パラメータ、およびヘッダーの検証:
    validateRequestParameters: true # クエリパラメータとヘッダー
    validateRequestBody: true # ボディ
  クエリ文字列パラメータおよびヘッダーの検証:
    validateRequestParameters: true
    validateRequestBody: false

components:
  #===========================
  # callbackキーの共通化
  #===========================
  callbacks: { }
  #===========================
  # linkキーの共通化
  #===========================
  links: { }
  #===========================
  # responseキーの共通化
  #===========================
  responses:
    unauthorized:
      description: Unauthorized レスポンス
      content:
        application/json: # MIME type
          example: # ボディ例
            status: 401
            title: Unauthorized
            errors:
              messages: [
                  "認証に失敗しました．"
              ]
          schema:
            $ref: "#/components/schemas/error" # 異常系モデルを参照する．                 
  #===========================
  # schemaキーの共通化
  #===========================
  schemas:
    # ユーザ
    user:
      type: object
      properties:
        userId:
          type: string
        name:
          type: string
    # 正常系
    normal:
      type: object
      properties:
        userId:
          type: string
    # 異常系      
    error:
      type: object
      properties:
        messages:
          type: array
          items:
            type: string
  #===========================
  # securityフィールドの共通化
  #===========================
  securitySchemes:
    # APIキー認証
    apiKeyAuth:
      description: APIキー認証
      type: apiKey
      name: x-api-key # カスタムヘッダー名
      in: header
```
<br>

### VPCリンク＆非プロキシ統合

**実装例**

```yaml
openapi: 3.0.0

info:
  title: example-api-with-non-proxy-integration # API名
  description: The API for example with non-proxy integration. # APIの説明
  termsOfService: https://www.example.com/terms/ # 利用規約
  contact:
    name: API support # 連絡先名
    url: https://www.example.com/support # 連絡先に関するURL
    email: support@example.com # メールアドレス
  license:
    name: Apache 2.0 # ライセンス
    url: https://www.apache.org/licenses/LICENSE-2.0.html # URL
  version: 1.0.0 # APIドキュメントのバージョン

servers:
  - url: https://{env}.example.com/api/v1
    description: |
    variables:
      env:
        default: stg
        description: API environment
        enum:
          - stg
          - www

paths:
  #===========================
  # pathsオブジェクト
  #===========================
  /users:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: ユーザ情報取得
      description: 全ユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: query # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                Users:
                  User:
                    userId: 1
                    name: Hiroki
              schema:
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                messages: [
                    "不正なリクエストです．"
                ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．     
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        httpMethod: GET
        uri: "http://<NLBのDNS名>/api/v1/users/"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.querystring.userId: method.request.querystring.userId # マッピングするクエリパラメータ
        passthroughBehavior: when_no_templates
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"body" : $input.json("$")}'
          400:
            statusCode: 400
          401:
            statusCode: 401
    #===========================
    # path itemオブジェクト
    #===========================
    post:
      tags:
        - ユーザ情報作成エンドポイント
      summary: ユーザ情報作成
      description: ユーザ情報を作成する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters: [ ]
      requestBody: # リクエストボディにパラメータを割り当てる．
        description: ユーザID
        content:
          application/json: # MIME type
            example: # リクエストボディ例
              userId: 1
            schema: # スキーマ
              $ref: "#/components/schemas/user" # Userモデルを参照する．
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                userId: 1
              schema:
                $ref: "#/components/schemas/normal" # 正常系モデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．              
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        httpMethod: POST
        uri: "http://<NLBのDNS名>/api/v1/users/"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
        requestTemplates:
          application/json: '{"body" : $input.json("$")}'
        passthroughBehavior: when_no_templates
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"body" : $input.json("$")}'
          400:
            statusCode: 400
          401:
            statusCode: 401
        # type: http
  #===========================
  # pathsオブジェクト
  #===========================
  /users/{userId}:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: 指定ユーザ情報取得
      description: 指定したユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
      #===========================
      # 統合
      #===========================                
      x-amazon-apigateway-integration:
        httpMethod: GET
        uri: "http://<NLBのDNS名>/api/v1/users/{userId}"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.path.userId: method.request.path.userId
        passthroughBehavior: when_no_templates
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"body" : $input.json("$")}'
          400:
            statusCode: 400
          401:
            statusCode: 401
          404:
            statusCode: 404
    #===========================
    # path itemオブジェクト
    #===========================                
    put:
      tags:
        - ユーザ情報更新エンドポイント
      summary: 指定ユーザ更新
      description: 指定したユーザ情報を更新する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．      
      #===========================
      # 統合
      #===========================               
      x-amazon-apigateway-integration:
        httpMethod: PUT
        uri: "http://<NLBのDNS名>/api/v1/users/{userId}"
        requestParameters:
          integration.request.header.X-API-Key: "'XXXXX'"
          integration.request.path.userId: method.request.path.userId
        passthroughBehavior: when_no_templates
        connectionType: VPC_LINK
        connectionId: <VPCリンクID>
        type: http
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"body" : $input.json("$")}'
          400:
            statusCode: 400
          401:
            statusCode: 401
#===========================
# バリデーションセットの定義
#===========================  
x-amazon-apigateway-request-validators:
  本文、クエリ文字列パラメータ、およびヘッダーの検証:
    validateRequestParameters: true # クエリパラメータとヘッダー
    validateRequestBody: true # ボディ
  クエリ文字列パラメータおよびヘッダーの検証:
    validateRequestParameters: true
    validateRequestBody: false

components:
  #===========================
  # callbackキーの共通化
  #===========================
  callbacks: { }
  #===========================
  # linkキーの共通化
  #===========================
  links: { }
  #===========================
  # responseキーの共通化
  #===========================
  responses:
    unauthorized:
      description: Unauthorized レスポンス
      content:
        application/json: # MIME type
          example: # ボディ例
            status: 401
            title: Unauthorized
            errors:
              messages: [
                  "認証に失敗しました．"
              ]
          schema:
            $ref: "#/components/schemas/error" # 異常系モデルを参照する．                 
  #===========================
  # schemaキーの共通化
  #===========================
  schemas:
    # ユーザ
    user:
      type: object
      properties:
        userId:
          type: string
        name:
          type: string
    # 正常系
    normal:
      type: object
      properties:
        userId:
          type: string
    # 異常系      
    error:
      type: object
      properties:
        messages:
          type: array
          items:
            type: string
  #===========================
  # securityフィールドの共通化
  #===========================
  securitySchemes:
    # APIキー認証
    apiKeyAuth:
      description: APIキー認証
      type: apiKey
      name: x-api-key # カスタムヘッダー名
      in: header
```

<br>

### モック統合

API Gatewayのエンドポイントに対して，以下のパラメータでリクエストを送信すると，レスポンスを確認できる．

```http
GET https://xxxxx.execute-api.ap-northeast-1.amazonaws.com/dev/users/?userId=1

X-API-Key：XXXXX
```

**実装例**

```yaml
openapi: 3.0.0

info:
  title: example-api-with-mock-integration # API名
  description: The API for example with mock integration # APIの説明
  termsOfService: https://www.example.com/terms/ # 利用規約
  contact:
    name: API support # 連絡先名
    url: https://www.example.com/support # 連絡先に関するURL
    email: support@example.com # メールアドレス
  license:
    name: Apache 2.0 # ライセンス
    url: https://www.apache.org/licenses/LICENSE-2.0.html # URL
  version: 1.0.0 # APIドキュメントのバージョン

servers:
  - url: https://{env}.example.com/api/v1
    description: |
    variables:
      env:
        default: stg
        description: API environment
        enum:
          - stg
          - www

paths:
  #===========================
  # pathsオブジェクト
  #===========================
  /users:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: ユーザ情報取得
      description: 全ユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: query # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                Users:
                  User:
                    userId: 1
                    name: Hiroki
              schema:
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                messages: [
                    "不正なリクエストです．"
                ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．     
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        requestTemplates:
          application/json: '{"statusCode": 200}'
        passthroughBehavior: when_no_templates
        type: mock
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{[{"id": 1,"name": test}]}'
          400:
            statusCode: 400
          401:
            statusCode: 401
    #===========================
    # path itemオブジェクト
    #===========================
    post:
      tags:
        - ユーザ情報作成エンドポイント
      summary: ユーザ情報作成
      description: ユーザ情報を作成する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters: [ ]
      requestBody: # リクエストボディにパラメータを割り当てる．
        description: ユーザID
        content:
          application/json: # MIME type
            example: # リクエストボディ例
              userId: 1
            schema: # スキーマ
              $ref: "#/components/schemas/user" # Userモデルを参照する．
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                userId: 1
              schema:
                $ref: "#/components/schemas/normal" # 正常系モデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # レスポンスボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．              
      #===========================
      # 統合
      #===========================          
      x-amazon-apigateway-integration:
        requestTemplates:
          application/json: '{"statusCode": 200}'
        passthroughBehavior: when_no_templates
        type: mock
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"id": 1}'
          400:
            statusCode: 400
          401:
            statusCode: 401
        # type: mock
  #===========================
  # pathsオブジェクト
  #===========================
  /users/{userId}:
    #===========================
    # path itemオブジェクト
    #===========================
    get:
      tags:
        - ユーザ情報取得エンドポイント
      summary: 指定ユーザ情報取得
      description: 指定したユーザ情報を取得する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: クエリ文字列パラメータおよびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
      #===========================
      # 統合
      #===========================                
      x-amazon-apigateway-integration:
        requestTemplates:
          application/json: '{"statusCode": 200}'
        passthroughBehavior: when_no_templates
        type: mock
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{[{"id": 1,"name": test}]}'
          400:
            statusCode: 400
          401:
            statusCode: 401
          404:
            statusCode: 404
    #===========================
    # path itemオブジェクト
    #===========================                
    put:
      tags:
        - ユーザ情報更新エンドポイント
      summary: 指定ユーザ更新
      description: 指定したユーザ情報を更新する．
      #===========================
      # メソッドリクエスト
      #===========================
      x-amazon-apigateway-request-validator: 本文、クエリ文字列パラメータ、およびヘッダーの検証
      security:
        - apiKeyAuth: [ ] # APIキーの必須化
      parameters:
        - in: path # パスにパラメータを割り当てる．
          name: userId
          required: true
          description: ユーザID
          schema:
            type: string
            example: # パスパラメータ例
              userId=1
      #===========================
      # メソッドレスポンス
      #===========================
      responses:
        '200':
          description: OK レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                userId: 1
                name: Hiroki
              schema: # スキーマ
                $ref: "#/components/schemas/user" # Userモデルを参照する．
        '400':
          description: Bad Request レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 400
                title: Bad Request
                errors:
                  messages: [
                      "ユーザIDは必ず指定してください．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．
        '401':
          $ref: "#/components/responses/unauthorized" # 認証エラーを参照する．
        '404':
          description: Not Found レスポンス
          content:
            application/json: # MIME type
              example: # ボディ例
                status: 404
                title: Not Found
                errors:
                  messages: [
                      "対象のユーザが見つかりませんでした．"
                  ]
              schema:
                $ref: "#/components/schemas/error" # 異常系モデルを参照する．      
      #===========================
      # 統合
      #===========================               
      x-amazon-apigateway-integration:
        requestTemplates:
          application/json: '{"statusCode": 200}'
        passthroughBehavior: when_no_templates
        type: mock
        responses:
          200:
            statusCode: 200
            responseTemplates:
              application/json: '{"id": 1}'
          400:
            statusCode: 400
          401:
            statusCode: 401
#===========================
# バリデーションセットの定義
#===========================  
x-amazon-apigateway-request-validators:
  本文、クエリ文字列パラメータ、およびヘッダーの検証:
    validateRequestParameters: true # クエリパラメータとヘッダー
    validateRequestBody: true # ボディ
  クエリ文字列パラメータおよびヘッダーの検証:
    validateRequestParameters: true
    validateRequestBody: false

components:
  #===========================
  # callbackキーの共通化
  #===========================
  callbacks: { }
  #===========================
  # linkキーの共通化
  #===========================
  links: { }
  #===========================
  # responseキーの共通化
  #===========================
  responses:
    unauthorized:
      description: Unauthorized レスポンス
      content:
        application/json: # MIME type
          example: # ボディ例
            status: 401
            title: Unauthorized
            errors:
              messages: [
                  "認証に失敗しました．"
              ]
          schema:
            $ref: "#/components/schemas/error" # 異常系モデルを参照する．                 
  #===========================
  # schemaキーの共通化
  #===========================
  schemas:
    # ユーザ
    user:
      type: object
      properties:
        userId:
          type: string
        name:
          type: string
    # 正常系
    normal:
      type: object
      properties:
        userId:
          type: string
    # 異常系      
    error:
      type: object
      properties:
        messages:
          type: array
          items:
            type: string
  #===========================
  # securityフィールドの共通化
  #===========================
  securitySchemes:
    # APIキー認証
    apiKeyAuth:
      description: APIキー認証
      type: apiKey
      name: x-api-key # カスタムヘッダー名
      in: header
```