---
description: Tips to write Laravel artisan CLI
---

# Artisan Console

## when you execute any artisan command, all command classes will be instantiated. 

{% hint style="info" %}
When you execute any artisan command like this:

`php artisan xxx:xxx`

at first Laravel will `instantiate` ALL the available command classes, even through you are not use them at all. After this process, your command will be executed.

That is why if any command class' construction failed \(thrown exception\), any other command cannot be executed properly. **So, please make your command class' constructor clean and do not include complex logic there**!  
{% endhint %}

## 

