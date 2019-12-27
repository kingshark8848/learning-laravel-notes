# Transaction

## DB::transaction and its nested usage

Every time  `DB::transaction` is used, The request global Connection instance will increase its `$transactions` property by 1. When logic inside this `DB::transaction` segment finished, `$transaction` will decrease by 1. Only when `$transactions` becomes 1,  it will really commit in DB. 

e.g:

```php
    DB::transaction(function (){
        DB::transaction(function (){


            DB::transaction(function (){
                $user = \App\Model\User::query()->find(2);
                $user->name = "Admin2";
                $user->save();
            });

            Log::debug("hello");

        });

        Log::debug("hello");
    });
```



1. After line 1, the Connection's `$transactions` become 1
2. After line 2, the Connection's `$transactions` become 2
3. After line 5, the Connection's `$transactions` become 3
4. After line 8, execute Connection's `commit()`. `$transactions` is 3, so no actual db commit. `$transaction` become 2.
5. After line 11, execute Connection's `commit()`. `$transactions` is 2, so no actual db commit. `$transaction` become 1.
6.  After line 15,  execute Connection's `commit()`. `$transactions` is 1, so **apply actual db commit**. `$transaction` become 0.

{% hint style="info" %}
DB::transaction's exception handling / rollback are another topic and has many details. Generally, either throw the exception and jump out of the current transaction segment \(if db transactions nested, the higher level transaction will catch the exception and deal again, form a chain\), or retry it.
{% endhint %}

