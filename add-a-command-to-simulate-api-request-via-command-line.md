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
use Illuminate\Console\Command;
use Illuminate\Http\Request;

class LocalRouteTest extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'test:local-route
                            {uri : The request URI}
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
        $userId = $this->option("user_id");

        $req = Request::create($uri, 'GET');

        if ($userId){
            /** @var User $user */
            $user = User::query()->findOrFail($userId);
            auth()->setUser($user);
        }

        $res = app()->handle($req);
        $responseBody = $res->getContent();
        $this->info($responseBody);
    }
}

```

usage:

```bash
php artisan test:local-route /horizon/api/stats
```

```bash
php artisan test:local-route /api/v1/admin/me -U 1
```



