# Laravel

## Facade

### 静的プロキシ

#### ・Facadeとは

Facadeに登録されたクラス（Facadeクラス）とServiceContainerを繋ぐ静的プロキシとして働く．メソッドをコールできるようになる．

#### ・Facadeを使用しない場合

new演算子でインスタンスを作成する．

**＊実装例＊**

```php
<?php
  
namespace 
    
class Example
{
    public function method()
    {
        return "example";
    }
}
```
```php
<?php
    
// 通常
$example = new Example();
$example->method();
```

#### ・Facadeの静的プロキシ機能を使用する場合

静的メソッドの記法でコールできる．ただし，自作クラスをFacadeの機能を使用してインスタンス化すると，スパゲッティな「Composition（合成）」の依存関係を生じさせてしまう．例えば，Facadeの中でも，```Route```のような，代替するよりもFacadeを使ったほうが断然便利である部分以外は，使用しないほうがよい．

**＊実装例＊**

Facadeとして使用したいクラスを定義する．

```php
<?php

namespace App\Domain\Entity;

class Example
{
    public function method()
    {
        return "example";
    }
}
```

```config/app.php```の```aliases```配列に，エイリアス名とクラスの名前空間を登録すると，そのエイリアス名でインスタンス化とメソッドコールを行えるようになる．

```php
<?php
  
'aliases' => [
    'Example' => App\Domain\Entity\Example::class,
]
```

インスタンス化とメソッドコールを行う．

```php
<?php

use Illuminate\Support\Facades\Example;
    
// Facade利用
$result = Example::method();
```

#### ・標準登録されたFacadeクラスの種類

以下のクラスは，デフォルトで登録されているFacadeである．

| エイリアス名         | クラス名                                                     | サービスコンテナ結合キー |
| :------------------- | :----------------------------------------------------------- | :----------------------- |
| App                  | [Illuminate\Foundation\Application](https://laravel.com/api/6.x/Illuminate/Foundation/Application.html) | `app`                    |
| Artisan              | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/6.x/Illuminate/Contracts/Console/Kernel.html) | `artisan`                |
| Auth                 | [Illuminate\Auth\AuthManager](https://laravel.com/api/6.x/Illuminate/Auth/AuthManager.html) | `auth`                   |
| Auth (Instance)      | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/6.x/Illuminate/Contracts/Auth/Guard.html) | `auth.driver`            |
| Blade                | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/6.x/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler`         |
| Broadcast            | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/6.x/Illuminate/Contracts/Broadcasting/Factory.html) |                          |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/6.x/Illuminate/Contracts/Broadcasting/Broadcaster.html) |                          |
| Bus                  | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/6.x/Illuminate/Contracts/Bus/Dispatcher.html) |                          |
| Cache                | [Illuminate\Cache\CacheManager](https://laravel.com/api/6.x/Illuminate/Cache/CacheManager.html) | `cache`                  |
| Cache (Instance)     | [Illuminate\Cache\Repository](https://laravel.com/api/6.x/Illuminate/Cache/Repository.html) | `cache.store`            |
| Config               | [Illuminate\Config\Repository](https://laravel.com/api/6.x/Illuminate/Config/Repository.html) | `config`                 |
| Cookie               | [Illuminate\Cookie\CookieJar](https://laravel.com/api/6.x/Illuminate/Cookie/CookieJar.html) | `cookie`                 |
| Crypt                | [Illuminate\Encryption\Encrypter](https://laravel.com/api/6.x/Illuminate/Encryption/Encrypter.html) | `encrypter`              |
| DB                   | [Illuminate\Database\DatabaseManager](https://laravel.com/api/6.x/Illuminate/Database/DatabaseManager.html) | `db`                     |
| DB (Instance)        | [Illuminate\Database\Connection](https://laravel.com/api/6.x/Illuminate/Database/Connection.html) | `db.connection`          |
| Event                | [Illuminate\Events\Dispatcher](https://laravel.com/api/6.x/Illuminate/Events/Dispatcher.html) | `events`                 |
| File                 | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/6.x/Illuminate/Filesystem/Filesystem.html) | `files`                  |
| Gate                 | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/6.x/Illuminate/Contracts/Auth/Access/Gate.html) |                          |
| Hash                 | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/6.x/Illuminate/Contracts/Hashing/Hasher.html) | `hash`                   |
| Lang                 | [Illuminate\Translation\Translator](https://laravel.com/api/6.x/Illuminate/Translation/Translator.html) | `translator`             |
| Log                  | [Illuminate\Log\LogManager](https://laravel.com/api/6.x/Illuminate/Log/LogManager.html) | `log`                    |
| Mail                 | [Illuminate\Mail\Mailer](https://laravel.com/api/6.x/Illuminate/Mail/Mailer.html) | `mailer`                 |
| Notification         | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/6.x/Illuminate/Notifications/ChannelManager.html) |                          |
| Password             | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/6.x/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`          |
| Password (Instance)  | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/6.x/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker`   |
| Queue                | [Illuminate\Queue\QueueManager](https://laravel.com/api/6.x/Illuminate/Queue/QueueManager.html) | `queue`                  |
| Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/6.x/Illuminate/Contracts/Queue/Queue.html) | `queue.connection`       |
| Queue (Base Class)   | [Illuminate\Queue\Queue](https://laravel.com/api/6.x/Illuminate/Queue/Queue.html) |                          |
| Redirect             | [Illuminate\Routing\Redirector](https://laravel.com/api/6.x/Illuminate/Routing/Redirector.html) | `redirect`               |
| Redis                | [Illuminate\Redis\RedisManager](https://laravel.com/api/6.x/Illuminate/Redis/RedisManager.html) | `redis`                  |
| Redis (Instance)     | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/6.x/Illuminate/Redis/Connections/Connection.html) | `redis.connection`       |
| Request              | [Illuminate\Http\Request](https://laravel.com/api/6.x/Illuminate/Http/Request.html) | `request`                |
| Response             | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/6.x/Illuminate/Contracts/Routing/ResponseFactory.html) |                          |
| Response (Instance)  | [Illuminate\Http\Response](https://laravel.com/api/6.x/Illuminate/Http/Response.html) |                          |
| Route                | [Illuminate\Routing\Router](https://laravel.com/api/6.x/Illuminate/Routing/Router.html) | `router`                 |
| Schema               | [Illuminate\Database\Schema\Builder](https://laravel.com/api/6.x/Illuminate/Database/Schema/Builder.html) |                          |
| Session              | [Illuminate\Session\SessionManager](https://laravel.com/api/6.x/Illuminate/Session/SessionManager.html) | `session`                |
| Session (Instance)   | [Illuminate\Session\Store](https://laravel.com/api/6.x/Illuminate/Session/Store.html) | `session.store`          |
| Storage              | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/6.x/Illuminate/Filesystem/FilesystemManager.html) | `filesystem`             |
| Storage (Instance)   | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/6.x/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk`        |
| URL                  | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/6.x/Illuminate/Routing/UrlGenerator.html) | `url`                    |
| Validator            | [Illuminate\Validation\Factory](https://laravel.com/api/6.x/Illuminate/Validation/Factory.html) | `validator`              |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/6.x/Illuminate/Validation/Validator.html) |                          |
| View                 | [Illuminate\View\Factory](https://laravel.com/api/6.x/Illuminate/View/Factory.html) | `view`                   |
| View (Instance)      | [Illuminate\View\View](https://laravel.com/api/6.x/Illuminate/View/View.html) |                          |

<br>

## ServiceProvider

### artisanによる操作

#### ・クラスの自動生成

```bash
$ php artisan make:provider {クラス名}
```

<br>

### ServiceProvider

#### ・ServiceProviderの用途

| 用途の種類                                         | 説明                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| AppServiceProvider                                 | ・ServiceContainerへのクラスのバインド（登録）<br>・ServiceContainerからのインスタンスのリゾルブ（生成） |
| MacroServiceProvider                               | ServiceContainerへのメソッドのバインド（登録）               |
| RouteServiceProvider<br>（app.php，web.phpも使用） | ルーティングとコントローラの対応関係の定義                   |
| EventServiceProvider                               | EventListenerとEventhandler関数の対応関係の定義              |

#### ・ServiceProviderのコール

```config/app.php```の```providers```配列に，クラスの名前空間を登録すると，アプリケーションの起動時にServiceProviderをコールできるため，ServiceContainerへのクラスのバインドが自動的に完了する．

**＊実装例＊**

```php
<?php

'providers' => [
    
    // 複数のServiceProviderが実装されている

    App\Providers\ComposerServiceProvider::class,
],
```

<br>

### AppServiceProvider

#### ・ServiceContainer，バインド，リゾルブ

クラスをバインド（登録）しただけで新しいインスタンスをリゾルブ（生成）してくれるオブジェクトを『ServiceContainer』という．

```php
<?php

namespace Illuminate\Contracts\Container;

use Closure;
use Psr\Container\ContainerInterface;

interface Container extends ContainerInterface
{
    /**
     * 通常のバインディングとして，自身にバインドする．
     * 第二引数は，クロージャー，もしくはクラス名前空間
     *
     * @param  string  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false);
    
    /**
     * singletonとして，自身にバインドする．
     *
     * @param  string  $abstract
     * @param  \Closure|string|null  $concrete
     * @return void
     */
    public function singleton($abstract, $concrete = null);
}
```

#### ・具象クラスをバインド

AppSeriveProviderにて，ServiceContainerにクラスをバインドすることによって，ServiceContainerがインスタンスを生成できるようになる．Laravelでは，具象クラスはServiceContainerに自動的にバインドされており，以下の実装を行う必要はない．

**＊実装例＊**

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Domain\Entity\Example;

class ExampleServiceProvider extends ServiceProvider
{
    /**
     * コンテナにバインド
     *
     * @return void
     */
    public function register()
    {
        $this->app->bind(Example::class, function ($app) {
            return new Example1(new Example2, new Example3);
        });
    }
}
```

#### ・複数の具象クラスをバインド

**＊実装例＊**

```php
<?php

namespace App\Providers;

use App\Domain\Entity\Example1;
use App\Domain\Entity\Example2;
use App\Domain\Entity\Example3;
use Illuminate\Support\ServiceProvider;

class ExamplesServiceProvider extends ServiceProvider
{
     /**
     * 各registerメソッドをコール
     *
     * @return void
     */
    public function register()
    {
        $this->registerExample1();
        $this->registerExample2();
        $this->registerExample3();
    }
    
    /**
    * 一つ目のクラスをバインド
    */
    private function registerExample1()
    {
        $this->app->bind(Example1::class, function ($app) {
            return new Example1();
        });
    }
    
    /**
    * 二つ目のクラスをバインド
    */    
    private function registerExample2()
    {
        $this->app->bind(Example2::class, function ($app) {
            return new Example2();
        });
    }
    
    /**
    * 三つ目のクラスをバインド
    */
    private function registerExample3()
    {
        $this->app->bind(Example3::class, function ($app) {
            return new Example3();
        });
    }
}
```

#### ・インターフェース（または抽象クラス）をバインド

具象クラスは自動的にバインドされるが，インターフェース（抽象クラス）は，手動でバインドする必要がある．このバインドによって，インターフェースをコールすると実装インスタンスを生成できるようになる．

**＊実装例＊**

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Domain\ExampleRepository as ExampleRepositoryInterface;
use App\Infrastructure\ExampleRepository;

class ExampleServiceProvider extends ServiceProvider
{
    /**
     * コンテナにバインド
     *
     * @return void
     */
    public function register()
    {
        // Domain層とInfrastructure層のリポジトリの結合をバインド
        $this->app->bind(
            'App\Domain\Repositories\ArticleRepository',
            'App\Infrastructure\Repositories\ArticleRepository'
        );
    }
}
```

#### ・ServiceContainerからのリゾルブ

コンストラクタの引数にクラスを渡しておく．Appファサードを使用してクラスを宣言することで，クラスのリゾルブを自動的に行う．

**＊実装例＊**

```php
<?php

namespace App\Domain\Entity;

class Example extends Entity
{
    /**
     * サブクラス
     */
    protected $subExample;

    /**
     * コンストラクタインジェクション
     *
     * @param SubExmaple $subExample
     */
    public function __construct(SubExmaple $subExample)
    {
        $this->example = $subExample;
    }
}
```

```php
<?php

class Example
{
    public function method()
    {
        return "example";
    }
}

// Exampleクラスをリゾルブし，そのままmethodをコール
$result = app()->make(Example::class)->method();

// Exampleクラスをリゾルブ
$example = App::make(Example::class);
$result = $example->method();
```

<br>

### MacroServiceProvider

```php
<?php

declare(strict_types=1);

namespace App\Providers;

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\ServiceProvider;

/**
 * マイグレーションマクロサービスプロバイダクラス
 */
class MigrationMacroServiceProvider extends ServiceProvider
{
    /**
     * サービスコンテナにマイグレーションメソッドをバインドします．
     *
     * @return void
     */
    public function register()
    {
        Blueprint::macro('systemColumns', function () {
            $this->string('created_by')
                ->comment('レコードの作成者')
                ->nullable();
            $this->string('updated_by')
                ->comment('レコードの最終更新者')
                ->nullable();;
            $this->string('created_at')
                ->comment('レコードの作成日')
                ->nullable();;
            $this->string('updated_at')
                ->comment('レコードの最終更新日')
                ->nullable();;
        });
    }
}
```

<br>

### RouteServiceProvider

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\RouteServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Route;

class RouteServiceProvider extends ServiceProvider
{
    /**
     * コントローラの場所を定義します．
     */
    protected $namespace = 'App\Http\Controllers';

    public const HOME = '/home';

    /**
     * パターンフィルタを定義します．
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

    public function map()
    {
        $this->mapApiRoutes();

        $this->mapWebRoutes();
    }

    /**
     * Webルーティングファイルのパスを定義します
     *
     * @return void
     */  
    protected function mapWebRoutes()
    {
        Route::middleware('web')
             ->namespace($this->namespace)
             ->group(base_path('routes/web.php'));
    }

    /**
     * Apiルーティングファイルのパスを定義します．
     *
     * @return void
     */  
    protected function mapApiRoutes()
    {
        Route::prefix('api')
             ->middleware('api')
             ->namespace($this->namespace)
             ->group(base_path('routes/api.php'));
    }
}

```

<br>

### EventServiceProvider

<br>

## Routes

### artisanによる操作

#### ・ルーティング一覧

```bash
# ルーティングを一覧で表示
$ php artisan route:list
```

#### ・キャッシュ削除

```bash
# ルーティングのキャッシュを削除
$ php artisan route:clear

# 全てのキャッシュを削除
$ php artisan optimize:clear
```

<br>

### 種類

#### ・api.php

RESTfulAPIとして扱うエンドポイントを実装する

#### ・web.php

API以外の場合，こちらにルーティング処理を実装する．第一引数にURL，第二引数に実行するメソッドを定義する．

<br>

### ルーティング

#### ・ルーティング定義

**＊実装例＊**

```php
<?php

Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

```{コントローラ名}@{メソッド名}```で，コントローラに定義してあるメソッドをコールできる．

**＊実装例＊**

```php
Route::get('/user', 'UserController@index');
```

#### ・namespace

```group```メソッドと組み合わせて使用する．コントローラをコールする時に，グループ内で同じ名前空間を指定できる．

**＊実装例＊**

```php
<?php

Route::namespace('Admin')->group(function () {
    // "App\Http\Controllers\Admin\" 下にあるコントローラ
    Route::get('/user', 'UserController@index');
    Route::post('/user/{userId}', 'UserController@createUser');
});
```

#### ・where

パスパラメータの形式の制約を，正規表現で設定できる．

**＊実装例＊**

```php
<?php

Route::namespace('Admin')->group(function () {

    Route::get('/user', 'UserController@index')
    
    // userIdの形式を「0〜9が一つ以上」に設定
    Route::post('/user/{userId}', 'UserController@createUser')
        ->where('id', '[0-9]+');
});
```

RouteServiceProviderの```boot```メソッドにて，```pattern```メソッドで制約を設定することによって，ルーティング時にwhereを使用する必要がなくなる．

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\RouteServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Route;

class RouteServiceProvider extends ServiceProvider
{
    /**
     * ルートモデル結合、パターンフィルタなどの定義
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('articleId', '[0-9]+');

        parent::boot();
    }
}

```

<br>

### セキュリティ

#### ・CSRF対策

Laravelでは，CSRF対策のため，POST，PUT，DELETEメソッドを使用するルーティングでCSRFトークンによる照合を行う必要がある．そのために，View側でCSRFトークンフィールドを実装する．この実装により，セッションごとに，登録ユーザにCSRFトークンを付与できる．

**＊実装例＊**

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

#### ・XSS対策

#### ・常時HTTPS化

<br>


## Migration

### artisanによる操作

#### ・マイグレーションファイルを作成

```bash
$ php artisan make:migrate create_{テーブル名}_table
```

#### ・テーブル作成

マイグレーションファイルを元にテーブルを作成する．

```bash
$ php artisan migrate
```

#### ・マイグレーションの結果を確認

```bash
$ php artisan migrate:status
```

#### ・指定した履歴数だけテーブルを元に戻す

指定した履歴数だけ，ロールバックを行う

```bash
$ php artisan migrate:rollback --step={ロールバック数}
```

#### ・初期の状態までテーブルを元に戻す

初期の状態までロールバック

```bash
$ php artisan migrate:reset
```

#### ・テーブルを元に戻してから再作成

```migrate:reset```と```migrate```を実行する．

```bash
$ php artisan migrate:refresh
```
#### ・テーブルを削除してから再作成

全てのテーブルを削除と```migrate```を実行する．マイグレーションファイルの構文チェックを行わずに，強制的に実行される．

```bash
$ php artisan migrate:fresh
```

<br>

### テーブルの作成と削除

#### ・up，down

コマンドによるマイグレーション時にコールされる．```up```メソッドでテーブル，カラム，インデックスのCREATEを行う．```down```メソッドでCREATEのロールバックを行う．

**＊実装例＊**

```php
<?php
  
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateExampleTable extends Migration
{
    /**
     * マイグレート
     *
     * @return void
     */
    public function up()
    {
        Schema::create('example', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * ロールバック
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('example');
    }
}
```

#### ・インデックスタイプ

```

```

<br>

## Factory，Seeder

### artisanによる操作

#### ・Factoryの生成

```bash
# --model={モデル名}
$ php artisan make:factory ExampleFactory --model=Example
```
#### ・個別Seederの生成
```bash
$ php artisan make:seeder UserSeeder
```

#### ・DatabaseSeederの実行

```bash
# 事前に，Composerのオートローラを再生成
$ composer dump-autoload

# DatabaseSeederを指定して実行
$ php artisan db:seed --class=DatabaseSeeder
```

<br>


### テストデータ

#### ・Factoryによるデータ定義

**＊実装例＊**

```php
<?php

use App\User;
use Faker\Generator as Faker;
use Illuminate\Database\Eloquent\Factory;
use Illuminate\Support\Str;

/**
 * @var Factory $factory
 */

$factory->define(User::class, function (Faker $faker) {
    return [
        'name'              => $faker->name,
        'email'             => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password'          => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token'    => Str::random(10),
    ];
});
```

#### ・Seederによるデータ作成

Factoryにおける定義を基にして，指定した数だけデータを作成する．

**＊実装例＊**

```php
<?php

use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create();
    }
}
```

DatabaseSeederにて，Seederをまとめて実行する．

**＊実装例＊**

```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * データベース初期値設定の実行
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```



## Eloquent｜Model

### artisanによる操作

#### ・クラスの自動生成

```bash
$ php artisan make:model Example
```

<br>

### Active Recordパターン

#### ・Active Recordパターンとは

![ActiveRecord](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/ActiveRecord.png)

#### ・他の類似するデザインパターンとの比較

| デザインパターン | Modelとテーブルの関連度合い                                 | 採用ライブラリ例 |
| ---------------- | ----------------------------------------------------------- | ---------------- |
| Active Record    | 非常に強い．基本的には，Modelとテーブルが一対一対応になる． | Eloquent         |
| Data Mapper      |                                                             | Doctrine         |
| Repository       | 各エンティティの粒度次第で強くも弱くもなる．                |                  |
| なし             | 非常に弱い                                                  | DBファサード     |

#### ・メリットとデメリット

| 項目   | メリット                                                     | デメリット                                                   |      |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| 保守性 |                                                              |                                                              |      |
| 可読性 | ・複雑なデータアクセスロジックを実装する必要が無い．<br>・Modelにおけるデータアクセスロジックがどのテーブルに対して行われるのか推測しやすい．<br>・リレーションを理解する必要があまりなく，複数のテーブルに対して無秩序にSQLを発行するような設計実装になりにくい． | データアクセスロジックが複数のテーブルを跨ぐためには，関連付くテーブルを軸としたリレーションを行う必要がある．この時，カラムの不要な取得を行ってしまうことがある． |      |
| 拡張性 |                                                              | Modelの構成がテーブル構造に強く依存してしまうため，Modelのドメインロジックを柔軟に実装できなくなってしまう．そのため，ドメイン駆動設計との相性は悪い． |      |

<br>

### Modelとテーブルの対応

#### ・Modelの継承

Modelクラスを継承したクラスは，```INSERT```文や```UPDATE```文などのデータアクセスロジックを使用できるようになる．

````php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    // クラスチェーンによって，データアクセスロジックをコール
}
````


#### ・テーブルとの関連付け

Eloquentは，Model自身の```table```プロパティに代入されている名前のテーブルに，Modelを関連付ける．ただし，```table```プロパティにテーブル名を代入する必要はない．Eloquentがクラス名の複数形をテーブル名と見なし，これをスネークケースにした文字列を自動的に代入する．また，テーブル名を独自で命名したい場合は，代入によるOverrideを行っても良い．

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    /**
     * モデルと関連しているテーブル
     *
     * @var string
     */
    protected $table = 'examples';
}
```

#### ・主キーとの関連付け

Eloquentは，```primaryKey```プロパティの値を主キーのカラム名と見なす．```keyType```で主キーのデータ型，また```$incrementing```プロパティで主キーのAutoIncrementを有効化するか否か，を設定できる．

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    /**
     * 主キーとするカラム
     * 
     * @var string 
     */
    protected $primaryKey = 'id';
    
    /**
     * 主キーのデータ型
     * 
     * @var string 
     */
    protected $keyType = 'int';
    
    /**
     * 主キーのAutoIncrementの有効化
     * 
     * @var bool 
     */
    public $incrementing = true;
}
```

#### ・タイムスタンプ型カラムとの関連付け

Eloquentは，```timestamps```プロパティの値が```true```の時に，Modelに関連付くテーブルの```created_at```と```updated_at```を自動的に更新する．また，タイムスタンプ型カラム名を独自で命名したい場合は，代入によるOverideを行っても良い．

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    const CREATED_AT = 'created_date_time';
    const UPDATED_AT = 'updated_data_time';
    
    /**
     * モデルのタイムスタンプを更新するかの指示
     *
     * @var bool
     */
    protected $timestamps = true; // falseで無効化
}
```


#### ・読み出し時のDateTimeクラス自動変換

データベースからタイムスタンプ型カラムを読み出すと同時に，CarbonのDateTimeクラスに変換したい場合，```data```プロパティにて，カラム名を設定する．

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * CarbonのDateTimeクラスに自動変換したいカラム名
     *
     * @var array
     */
    protected $dates = [
        'created_at',
        'updated_at',
        'deleted_at'
    ];
}

```


#### ・デフォルト値の設定

特定のカラムのデフォルト値を設定したい場合，```attributes```プロパティにて，カラム名と値を設定する．

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    /**
     * 属性に対するモデルのデフォルト値
     *
     * @var array
     */
    protected $attributes = [
        'is_deleted' => false,
    ];
}
```

<br>

## Eloquent｜Data Access

### artisanによる操作

```bash

```

<br>

### CREATE

#### ・insert文の実行

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    /**
     * Exampleエンティティを作成
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Example $example)
    {
        // 内部でinsert文が実行される
        $example->save();
    }
}
```

<br>

### READ

#### ・select文の実行

内部でselect文を実行する```all```メソッドまたは```get```メソッドの返却値の型は，Collectionである．

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    private $example;
    
    public function __construct(Example $example)
    {
        $this->example = $example;
    }
  
    /**
     * 全てのExampleエンティティを読み出し
     *
     * @param Id $id
     */
    public function findAllEntity()
    {
        return $this->example->all()->toArray();
    }
  
    /**
     * 条件に合うExampleエンティティを読み出し
     *
     * @param Id $id
     */
    public function findEntityByCriteria($id)
    {
        return $this->example->find($id)->toArray();
    }
}
```

#### ・Collectionクラス

List型で，データを保持できるオブジェクトのこと．データ型変換メソッドを持っている．

```php
<?php

$collection = collect([
    [
        'id'  => '1',
        'name' => '佐藤太郎',
    ],
    [
        'id'  => '2',
        'name' => '山田次郎',
    ],
]);

// Array型に変換する
$collection->toArray();
```

<br>

### UPDATE

#### ・update文の実行

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    private $example;
    
    public function __construct(Example $example)
    {
        $this->example = $example;
    }
    
    /**
     * Exampleエンティティを更新
     *
     * @param Id $id
     */
    public function update(Id $id)
    {
        $example = $this->example->find($id);
        
        // エンティティのデータを更新
        $example->name = $example->name;
        $example->type = $example->type;
        
        // 内部でupdate文が実行される
        $example->save();
    }
}
```

<br>

### DELETE

#### ・delete文の実行（物理削除）

```find```メソッドで読み出したモデルから，```delete```メソッドをコールし，このモデルを削除する．

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    private $example;
    
    public function __construct(Example $example)
    {
        $this->example = $example;
    }
  
    /**
     * Exampleエンティティを削除
     *
     * @param Id $id
     */
    public function delete(Id $id)
    {
        $example = $this->example->find($id);
        
        // 内部でdelete文が実行され，物理削除される．
        $example->delete();
    }
}
```

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    private $example;
    
    public function __construct(Example $example)
    {
        $this->example = $example;
    }  

    /**
     * Exampleエンティティを登録
     *
     * @param  Example $example
     */
    public function delete(Id $id)
    {
        $this->example->destroy($id);
    }
}
```

#### ・update文の実行（論理削除）

論理削除を行いたい場合，テーブルに対応するモデルにて，SoftDeletesトレイトを読み込む．

```php
<?php

namespace App\Domain\Entity;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Example extends Model
{
    /**
    * トレイトの読み込み
    */
    use SoftDeletes;

    /**
     * 読み出し時にCarbonのDateTimeクラスへ自動変換するカラム
     *
     * @var array
     */
    protected $dates = [
        'deleted_at'
    ];
}
```
さらに，マイグレーションファイルにて```softDeletes```メソッドを使用し，```deleted_at```カラムが追加されるようにする．```deleted_at```カラムのデフォルト値は```NULL```である．

```php
Schema::table('example', function (Blueprint $table) {
    $table->softDeletes();
});
```

上記の状態で，同様に```delete```メソッドをコールすると，物理削除ではなく，```deleled_at```カラムが更新されるようになる．```find```メソッドは，```deleled_at```が```NULL```でないデータを読み出さないため，論理削除を実現できる．

```php
<?php

namespace App\Http\Controllers;

use App\Domain\Entity\Example;
use App\Http\Controllers\Controller;

class ExampleRepository extends Repository
{
    /**
     * Exampleエンティティを登録
     *
     * @param Id $id
     */
    public function delete(Id $id)
    {
        $example = Example::find($id);
        
        // 内部でupdate文が実行され，論理削除される．
        $example->delete();
    }
}
```

<br>

## HTTP｜Middleware

### artisanによる操作

#### ・クラスの自動生成

```bash
# Middlewareを自動生成
$ php artisan make:middleware {クラス名}
```

<br>

### Middlewareの仕組み

#### ・Middlewareの種類

ルーティング後にコントローラメソッドの前にコールされるBeforeMiddleと，レスポンスの実行時にコールされるAfterMiddlewareがある

![Laravelのミドルウェア](https://raw.githubusercontent.com/Hiroki-IT/tech-notebook/master/images/LaravelのMiddlewareクラスの仕組み.png)

#### ・BeforeMiddleware

ルーティング時のコントローラメソッドのコール前に実行する処理を設定できる．```$request```には，```Illuminate\Http\Request```オブジェクトが代入されている．

**＊実装例＊**

```php
<?php

namespace App\Http\Middleware;

use Closure;

class ExampleBeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // 何らかの処理

        return $next($request);
    }
}
```
#### ・AfterMiddleware

コントローラメソッドのレスポンスの実行後（テンプレートのレンダリングを含む）に実行する処理を設定できる．```$response```には，```Illuminate\Http\Response```オブジェクトが代入されている．

**＊実装例＊**


```php
<?php

namespace App\Http\Middleware;

use Closure;

class ExampleAfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // 何らかの処理

        return $response;
    }
}
```

<br>

## HTTP｜Request

### artisanによる操作

#### ・クラスの自動生成

```bash
# フォームクラスを自動作成
$ php artisan make:request {クラス名}
```

<br>

### バリデーションルールの定義

#### ・```rules```メソッド，```validated```メソッド

FormRequestを継承したクラスにて，```rules```メソッドを使用して，ルールを定義する．バリデーションルールに反すると，一つ目のルール名（例えば```required```）に基づき，```validation.php```から対応するエラーメッセージを自動的に選択する．

**＊実装例＊**

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class ExampleRequest extends FormRequest
{
    /**
     * リクエストに適用するバリデーションルールを取得
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body'  => 'required',
        ];
    }
}
```

Controllerで，ExampleRequestを型指定すると，コントローラのメソッドをコールする前にバリデーションが自動で行われる．そのため，コントローラの中では，```validated```メソッドでバリデーションを終えたデータをいきなり取得できる．バリデーションが失敗した場合，メソッドのコール前に，ExampleRequestが元々のページにリダイレクトする処理を自動で実行する．

**＊実装例＊**

```php
/**
 * ブログポストの保存
 *
 * @param  ExampleRequest $request
 * @return Response
 */
// メソッドが実行される前にバリデーションが行われる．
public function store(ExampleRequest $request) 
{
    // バリデーション済みデータをいきなり取得
    $validated = $request->validated();
  
    ExampleRepository::update($validated);
}
```

#### ・```validate```メソッド

```Illuminate\Http\Request```インスタンスの```validate```メソッドを使用して，ルールを定義する．validationルールに反すると，一つ目のルール名（例えば```required```）に基づき，```validation.php```から対応するエラーメッセージを自動的に選択する．

**＊実装例＊**

```php
/**
 *
 * @param Request $request
 * @return Response
 */
public
function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body'  => 'required',
    ]);
}
```

validationルールは，配列で定義してよい．

**＊実装例＊**

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### ・Validatorファサード

```Illuminate\Support\Facades\Validator```ファサードの```make```メソッドを使用して，ルールを定義する．第一引数で，バリデーションを行うリクエストデータを渡す．validationルールに反すると，一つ目のルール名（例えば```required```）に基づき，```validation.php```から対応するエラーメッセージを自動的に選択する．

**＊実装例＊**

```php
<?php
  
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class ExampleController extends Controller
{
    /**
     * 新しいブログポストの保存
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('example/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // ブログポストの保存処理…
    }
}
```

<br>

### セッション

#### ・セッション変数の取得

```Illuminate\Http\Request```インスタンスの```session```メソッドを使用して，セッション変数を取得する．

**＊実装例＊**

```php
<?php
  
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 指定されたユーザーのプロフィールを表示
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');
    }
}
```

全てのセッション変数を取得することもできる．

```php
$data = $request->session()->all();
```

#### ・フラッシュデータの設定

現在のセッションにおいて，今回と次回のリクエストだけで有効な一時データを設定できる．

```php
$request->session()->flash('status', 'Task was successful!');
```

<br>

### Requestの認証

#### ・```authorize```メソッド

ユーザがリソースに対してCRUD操作を行う権限を持っているかを，コントローラのメソッドを実行する前に，判定する．

**＊実装例＊**

```php
/**
 * ユーザーがこのリクエストの権限を持っているかを判断する
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

<br>

## HTTP｜Controller

### artisanによる操作

#### ・クラスの自動生成

```bash
# コントローラクラスを自動作成
$ php artisan make:controller {クラス名}
```

<br>

### Requestのコール

####  ・データの取得

```input```メソッドを用いて，リクエストボディに含まれるデータを取得できる．

**＊実装例＊**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 新しいユーザーを保存
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');
    }
}
```

#### ・パスパラメータの取得

第二引数にパスパラメータ名を記述することで，パスパラメータの値を取得できる．

**＊実装例＊**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 指定したユーザーの更新
     *
     * @param  Request  $request
     * @param  string  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }
```

<br>

### Responseのコール

#### ・Json型データのレスポンス

```Symfony\Component\HttpFoundation\Response```を継承している．

**＊実装例＊**

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```

#### ・Viewテンプレートのレスポンス

**＊実装例＊**

```php
// データ，ステータスコード，ヘッダーなどを設定する場合
return response()
  ->view('hello', $data, 200)
  ->header('Content-Type', $type);
```

```php
// ステータスコードのみ設定する場合
return response()
  ->view('hello')
  ->setStatusCode(200);
```

<br>

## HTTP｜Auth

### artisanによる操作

#### ・Digest認証の環境構築

```bash
# ここにコマンド例
```

#### ・Oauth認証の環境構築

```/storage/oauth```キー，Personal Access Client，Password Grant Clientを生成する．

```bash
$ php artisan passport:install

Personal access client created successfully.
Client ID: 3
Client secret: xxxxxxxxxxxx
Password grant client created successfully.
Client ID: 4
Client secret: xxxxxxxxxxxx
```

ただし，生成コマンドを個別に実行してもよい．

```bash
# 暗号キーを生成
$ php artisan passport:keys

# クライアントを生成
## Persinal Access Tokenの場合
$ php artisan passport:client --personal
## Password Grant Tokenの場合
$ php artisan passport:client --password
```

<br>

### AuthファサードによるDigest認証

```attempt```メソッドを用いて，パスワードを自動的にハッシュ化し，データベースのハッシュ値と照合する．認証が終わると，認証セッションを開始する．```intended```メソッドで，ログイン後の初期ページにリダイレクトする．

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * 認証を処理する
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // 認証に成功した
            return redirect()->intended('dashboard');
        }
    }
}
```

<br>

### PassportによるAPIのOauth認証

#### ・Personal Access Token

開発環境で使用するアクセストークン．

1. 暗号キーとユーザを作成する．

```bash
$ php artisan passport:keys

$ php artisan passport:client --personal
```

2. 作成したユーザに，クライアントIDを付与する．

```php
/**
 * 全認証／認可の登録
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::personalAccessClientId('client-id');
}
```

3. ユーザからのリクエスト時，クライアントIDを元に『認証』を行い，アクセストークンをレスポンスする．

```php
$user = App\User::find(1);

// スコープ無しのトークンを作成する
$token = $user->createToken('Token Name')->accessToken;

// スコープ付きのトークンを作成する
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

#### ・Password Grant Token

本番環境で使用するアクセストークン．

1. API側では，Oauth認証（認証フェーズ＋認可フェーズ）を行うために，auth.phpで，```driver```を```passport```に設定する．また，認証フェーズで使用するテーブル名を```provider```に設定する．

**＊実装例＊**

```php
<?php

// 一部省略

'guards' => [
    'web' => [
        'driver'   => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver'   => 'passport',
        'provider' => 'users',
        'hash'     => false,
    ],
],
```

2. API側では，テーブルに対応するモデルのルーティングに対して，```middleware```メソッドによる認証ガードを行う．これにより，Oauth認証に成功したユーザのみがルーティングを行えるようになる．

**＊実装例＊**

```php
Route::get('user', 'UserController@index')->middleware('auth:api');
```
3. API側では，認証ガードを行ったモデルに対して，HasAPIToken，Notifiableのトレイトをコールするようにする．

**＊実装例＊**

```php
<?php
  
namespace App\Domain\Entity;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
    
    // 一部省略
}
```

4. API側では，Passportの```routes```メソッドをコールするようにする．これにより，認証フェーズでアクセストークンをリクエストするための全てのルーティング（``````/oauth/xxx``````）が有効になる．また，アクセストークンを発行できるよになる．

**＊実装例＊**

```php
<?php
  
use Laravel\Passport\Passport;

class AuthServiceProvider extends ServiceProvider
{
    // 一部省略

    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();
    }
}
```

5. API側では，暗号キーとユーザを作成する．

```bash
$ php artisan passport:keys

$ php artisan passport:client --password
```

6. 以降，ユーザ側のアプリケーションでの作業となる．『認証』のために，アクセストークンのリクエストを送信する．ユーザ側のアプリケーションは，```/oauth/authorize```へリクエストを送信する必要がある．ここでは，リクエストGuzzleライブラリを使用して，リクエストを送信するものとする．

**＊実装例＊**

```php
<?php

$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type'    => 'password',
        'client_id'     => 'client-id',
        'client_secret' => 'client-secret',
        'username'      => 'taylor@laravel.com',
        'password'      => 'my-password',
        'scope'         => '',
    ],
]);
```

7. ユーザ側のアプリケーションでは，ユーザ側のアプリケーションにアクセストークンを含むJSON型データを受信する．

**＊実装例＊**

```json
{
  "token_type":"Bearer",
  "expires_in":31536000,
  "access_token":"xxxxx"
}
```

8. ユーザ側のアプリケーションでは，ヘッダーにアクセストークンを含めて，認証ガードの設定されたAPI側のルーティングに対して，リクエストを送信する．レスポンスのリクエストボディからデータを取得する．

**＊実装例＊**

```php
<?php
  
$response = $client->request('GET', '/api/user', [
    'headers' => [
        'Accept'        => 'application/json',
        'Authorization' => 'Bearer xxxxx',
    ]
]);

return (string)$response->getBody();
```

<br>

## Views

### arisanによる操作

#### ・キャッシュの削除

```bash
# ビューのキャッシュを削除
$ php artisan view:clear

# 全てのキャッシュを削除
$ php artisan optimize:clear
```

<br>

### データの出力

#### ・データの出力

Responseインスタンスから渡されたデータは，```{{ 変数名 }}``で取得できる．`

**＊実装例＊**

```html
<html>
    <body>
        <h1>Hello!! {{ $data }}</h1>
    </body>
</html>
```

#### ・validationメッセージの出力

**＊実装例＊**

```html
<!-- /resources/views/example/create.blade.php -->

<h1>ポスト作成</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
            
        </ul>
    </div>
@endif

<!-- ポスト作成フォーム -->
```

<br>

### 要素の共通化

#### ・@extends，@parent，@show

三つセットで使うことが多いので，同時に説明する．テンプレート間が親子関係にある時，子テンプレートで親テンプレート全体を読み込む```@extends```を用いる．子テンプレートに```@section```を継承しつつ，要素を追加する場合，```@endsection```ではなく```@show```を使用する．

**＊実装例＊**

```html
<!-- 親テンプレート -->

<html>
    <head>
        <title>共通のタイトルとなる要素</title>
    </head>
    <body>
        
        @section('sidebar')
            親テンプレートのサイドバーとなる要素
        @show

        <div class="container">
            共通の要素
        </div>
    </body>
</html>
```

```@parent```で親テンプレートの```@section```を継承する．

```html
<!-- 子テンプレート -->

@extends('layouts.app')

@section('sidebar')
    @parent
    <p>子テンレプートのサイドバーに追加される要素</p>
@endsection
```

#### ・@include（サブビュー）

読み込んだファイル全体を出力する．読み込むファイルに対して，変数を渡すこともできる．```@extentds```との使い分けとして，親子関係のないテンプレートの間で使用するのがよい．両者は，PHPでいう```extends```（クラスチェーン）と```require```（単なる読み込み）の関係に近い．

```html
<div>
    @include('shared.errors')
    <form><!-- フォームの内容 -->
    </form>
</div>
```

<br>

### 子テンプレートにおける要素の出力

#### ・@yield，@section

子テンプレートのレンダリング時に，HTMLの要素を動的に出力する場合に使用する．親テンプレートにて，```@yield('example')```を定義する．これを継承した子テンプレートのレンダリング時に，```@section('example')```-```@endsection```で定義した要素が，```@yieid()```部分に出力される．

**＊実装例＊**

```html
<!-- 親テンプレート -->

<!DOCTYPE html>
<html lang="ja">
    <head>
        <meta charset="utf-8">
        <title>アプリケーション</title>
    </head>
    <body>
        <h2>タイトル</h2>        
        @yield('content')
    </body>
</html>
```

```html
<!-- 子テンプレート -->

@extends('layouts.parent')

@section('content')
    <p>子テンプレートのレンダリング時に，yieldに出力される要素</p>
@endsection
```

子テンプレートは，レンダリング時に以下のように出力される．

```html
<!-- 子テンプレート -->

<!DOCTYPE html>
<html lang="ja">
    <head>
        <meta charset="utf-8">
        <title>アプリケーション</title>
    </head>
    <body>
        <h2>タイトル</h2>
        <p>子テンプレートのレンダリング時に，yieldに出力される要素</p>
    </body>
</html>
```

<br>

#### ・@stack，@push

子テンプレートのレンダリング時に，CSSとJavaScriptのファイルを動的に出力する場合に使用する．親テンプレートにて，```@stack('example')```を定義する．これを継承した子テンプレートのレンダリング時に，```@push('example')```-```@endpush```で定義した要素が，```@stack()```部分に出力される．

**＊実装例＊**

```html
<!-- 親テンプレート -->

<head>
    <!-- Headの内容 -->

    @stack('scripts')
</head>
```

```html
<!-- 子テンプレート -->

@push('scripts')
    <script src="/example.js"></script>
@endpush
```

<br>

### Twigとの互換

#### ・Bladeで実装した場合

**＊実装例＊**

```html
<!-- 親テンプレート -->

<html>
    <head>
        <title>@yield('title')</title>
    </head>
    <body>

        @section('sidebar')
            親テンプレートのサイドバーとなる要素
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

```html
<!-- 子テンプレート -->

@extends('layouts.master')

@section('title', '子テンプレートのタイトルになる要素')

@section('sidebar')
    @parent
    <p>子テンレプートのサイドバーに追加される要素</p>
@endsection

@section('content')
    <p>子テンプレートのコンテンツになる要素</p>
@endsection
```

#### ・Twigで実装した場合

**＊実装例＊**

```html
<!-- 親テンプレート -->

<html>
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>

        {% block sidebar %} <!-- @section('sidebar') に相当 -->
            親テンプレートのサイドバーとなる要素
        {% endblock %}

        <div class="container">
            {% block content %}<!-- @yield('content') に相当 -->
            {% endblock %}
        </div>
    </body>
</html>
```
```html
<!-- 子テンプレート -->

{% extends "layouts.master" %} <!-- @extends('layouts.master') に相当 -->

{% block title %} <!-- @section('title', 'Page Title') に相当 -->
    子テンプレートのタイトルになる要素
{% endblock %}

{% block sidebar %} <!-- @section('sidebar') に相当 -->
    {{ parent() }} <!-- @parent に相当 -->
    <p>子テンレプートのサイドバーに追加される要素</p>
{% endblock %}

{% block content %} <!-- @section('content') に相当 -->
    <p>子テンプレートのコンテンツになる要素</p>
{% endblock %}
```

<br>

## Config

### 環境変数

#### ・envメソッド

ルートディレクトリにある```.env```ファイルに定義された値を，環境変数として出力することができる．

```php
<?php

env('<環境変数名>', '<デフォルト値>');
```

#### ・app.php

ルートディレクトリにある```.env```ファイルのアプリケーションに関する値が出力される．

```php
<?php

return [
    // アプリケーション名
    'name'            => env('APP_NAME', 'Laravel'),

    // 実行環境名
    'env'             => env('APP_ENV', 'production'),

    // エラー時のデバッグ画面の有効化
    'debug'           => (bool)env('APP_DEBUG', false),

    // アプリケーションのURL
    'url'             => env('APP_URL', 'http://localhost'),

    // assetヘルパーで付与するURL
    'asset_url'       => env('ASSET_URL', null),

    // タイムゾーン
    'timezone'        => 'UTC',

    // 言語設定
    'locale'          => 'en',
    'fallback_locale' => 'en',
    'faker_locale'    => 'en_US',

    // APIキー
    'key'             => env('APP_KEY'),

    // 暗号化アルゴリズム
    'cipher'          => 'AES-256-CBC',

    // サービスプロバイダー
    'providers'       => [

    ],

    // クラス名のエイリアス
    'aliases'         => [

    ],
];



```

#### ・database.php

```.env```ファイルのデータベースに関する値が出力される．

```php
<?php

use Illuminate\Support\Str;

return [

    // 使用するDBMSを設定
    'default'     => env('DB_CONNECTION', 'mysql'),

    'connections' => [

        // データベース接続情報（SQLite）
        'sqlite' => [

        ],

        // データベース接続情報（MySQL）
        'mysql'  => [

        ],

        // データベース接続情報（pgSQL）
        'pgsql'  => [

        ],

        // データベース接続情報（SQLSRV）
        'sqlsrv' => [

        ],
    ],

    // マイグレーションファイルのあるディレクトリ
    'migrations'  => 'migrations',

    // Redis接続情報
    'redis'       => [

    ],
];
```

