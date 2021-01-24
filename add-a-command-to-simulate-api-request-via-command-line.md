# Add a Command to simulate API request via Command line

For our real Laravel Application, recently some clients complaint about how slow it is suddenly. We want to know if the speed reduce come from PHP process it self or other issues. we can create a command to do some experiments.

Create a command `LocalRouteTest`

```bash
php artisan make:command LocalRouteTest
```

Register it in Kernel and edit it:

```php
<?php

namespace App\Console\Commands;

use App\Model\User;
use Dingo\Api\Http\Request;
use Illuminate\Console\Command;
use Tymon\JWTAuth\Facades\JWTAuth;

class LocalRouteTest extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'test:local-route
                            {uri : The request URI}
                            {method : The request method. e.g: GET|POST|PUT|DELETE, etc}
                            {--P|parameters= : Parameter JSON string. }
                            {--U|user_id= : Binding user ID}
    ';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Test Local Route';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     * @throws \Exception
     */
    public function handle()
    {
        $uri = $this->argument('uri');
        $method = $this->argument('method');
        $userId = $this->option("user_id");

        $params = $this->option("parameters");
        $params = $params ? json_decode($params, true) : [];

        $req = Request::create($uri, $method, $params);

        if ($userId){
            /** @var User $user */
            $user = User::query()->findOrFail($userId);
//            auth()->setUser($user);

            $token = JWTAuth::fromUser($user);
//            $req->headers->set('Authorization', "Bearer {$token}");
            $req->query->set("token", $token);
        }

        $res = app()->handle($req);
        $responseBody = $res->getContent();
        $this->info($responseBody);
    }
}


```

usage:

```bash
php artisan test:local-route /horizon/api/stats GET
```

```bash
php artisan test:local-route /api/v1/admin/me GET -U 1
```

```bash
php artisan test:local-route /api/v1/admin/dashboard_folder/me POST -U 1 -P '{"name": "command_created_dashboard"}'
```

