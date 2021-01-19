# Auth

## JWT Auth can parse token mechanism \(default\)

package `tymon/jwt-auth`  version `dev-develop|1.0.*`

Using this package to do jwt auth, it attempts to parse token by different methods as a chain.

* header:  `Bearer <token>`  
* query string: `?token=<token>`
* request input data: e.g: form data `token=<token>`
* route parameter:  you need to write the router like this  `/me/token/<token>` , `/token/<token>/info`, then you can get token by this method.
* cookie: `token` in cookies

following the order, if token parsed successfully by ANY method in the chain, it stops and regards it as the token, then send to auth process. if no token parsed by ALL methods, the it regards there is no token.

referred source code:

{% code title="\\Tymon\\JWTAuth\\Providers\\AbstractServiceProvider::registerTokenParser \(v1.0.2\)" %}
```php
    /**
     * Register the bindings for the Token Parser.
     *
     * @return void
     */
    protected function registerTokenParser()
    {
        $this->app->singleton('tymon.jwt.parser', function ($app) {
            $parser = new Parser(
                $app['request'],
                [
                    new AuthHeaders,
                    new QueryString,
                    new InputSource,
                    new RouteParams,
                    new Cookies($this->config('decrypt_cookies')),
                ]
            );

            $app->refresh('request', $parser, 'setRequest');

            return $parser;
        });
    }
```
{% endcode %}



