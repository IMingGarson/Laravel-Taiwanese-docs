# 資料庫：Seeding

- [資料庫：Seeding](#資料庫seeding)
  - [介紹](#介紹)
  - [撰寫Seeders](#撰寫seeders)
    - [使用 Model Factories](#使用-model-factories)
    - [呼叫額外 Seeders](#呼叫額外-seeders)
    - [關閉 Model Events](#關閉-model-events)
  - [執行 Seeders](#執行-seeders)
      - [在正式站環境強制執行Seeders](#在正式站環境強制執行seeders)

<a name="introduction"></a>
## 介紹

Laravel提供seeder classes來為您的資料庫輸入資料，所有的seeder classes檔案都放置於 `database/seeders` 資料夾下。

預設情況下，已經有一個定義好的 `DatabaseSeeder`  class在該資料夾。

透過這個Class，您可以使用 `call` 函式來執行其他seed classes，藉此讓您能夠控制建立資料的順序。

> {小提式} 在seeder classes建立資料的過程中，[Mass assignment protection](/docs/{{version}}/eloquent#mass-assignment)預設是關閉的。


<a name="writing-seeders"></a>
## 撰寫Seeders

我們可以透過 `make:seeder` [Artisan command](/docs/{{version}}/artisan) 這個指令來撰寫一個seed class。Laravel製作的所有seeder都會放在 `database/seeders` 資料夾內。

```shell
php artisan make:seeder UserSeeder
```

一個seeder class預設只會有一個 `run` 函式。該函式會在輸入 `db:seed` [Artisan command](/docs/{{version}}/artisan) 後執行。

在 `run` 函式內，您可以用任何方式新增您的資料，您可以使用 [query builder](/docs/{{version}}/queries) 來手動輸入資料或使用 [Eloquent model factories](/docs/{{version}}/database-testing#defining-model-factories) 來建立資料。

舉例來說，我們可以修改預設的 `DatabaseSeeder` class－在 `run` 內寫一個新增資料的語法：

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeders.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@gmail.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> {小提示} 您可以給 `run` 函式任何您需要的dependencies作為參數型態，Laravel會透過 [service container](/docs/{{version}}/container) 自動使用這些參數型態。

<a name="using-model-factories"></a>
### 使用 Model Factories

當然，每次都得幫每個Model手動輸入資料是十分麻煩的。這時您可以使用非常方便的 [model factories](/docs/{{version}}/database-testing#defining-model-factories) 來一次產生大量資料。首先，先來看看 [model factory documentation](/docs/{{version}}/database-testing#defining-model-factories) 來學習如何定義您的factories。

舉例來說，建立50個使用者且每個使用者都有1篇貼文：

    use App\Models\User;

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }

<a name="calling-additional-seeders"></a>
### 呼叫額外 Seeders

在 `DatabaseSeeder` class，您可以使用 `call` 來使用額外的seed classes。

這種做法可以幫助您將建立資料的過程分割為數個檔案，不會讓其中一個seeder class變得過大。

`call` 接受一個Array作為參數，該Array內包含要執行的seeder classes。

    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="muting-model-events"></a>
### 關閉 Model Events

在建立資料的過程中，您也許會想關閉任何Model發送任何事件，您可透過使用 `WithoutModelEvents` trait來達到該效果。

當使用 `WithoutModelEvents` 時，它能保證Model不會發送任何事件，就算是使用 `call` 執行多個seed classes也一樣。


    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Console\Seeds\WithoutModelEvents;

    class DatabaseSeeder extends Seeder
    {
        use WithoutModelEvents;

        /**
         * Run the database seeders.
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

<a name="running-seeders"></a>
## 執行 Seeders

您可以透過執行 `db:seed` Artisan command 來建立資料到資料庫內。

預設情況下，`db:seed` 會執行 `Database\Seeders\DatabaseSeeder` 這個class，而該class可以用於呼叫其他seeder classes。

但您可以使用 `--class` 這個參數來個別指定您想執行的seeder class。

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

若您想刪掉現有的資料表並重新建立資料，您也可以使用 `migrate:fresh` 與 `--seed` ，這種做法對於重新建立資料庫是十分方便的。

```shell
php artisan migrate:fresh --seed
```

<a name="forcing-seeding-production"></a>
#### 在正式站環境強制執行Seeders

有些建立資料的作業可能會導致資料異動或遺失，為了避免在 `production` 環境發生這種情況，在執行seeders時會被詢問是否要在該環境執行；然而，您可以使用 `--force` 來跳過這個過程。

```shell
php artisan db:seed --force
```
