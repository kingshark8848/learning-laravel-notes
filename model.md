---
description: Tips about model
---

# Model

## model's boot

In a request life cycle, if any model are instantiating, it will first call `bootIfNotBooted()` function to boot the model. As the function name indicates, it would execute boot process only if it was not booted in this request life cycle.

How to execute boot process? look at `bootTraits()` function here:

it will find all traits current model used,  and **execute every trait's static function whose name is "boot\[Trait-Name\]"**. e.g:

 `protected static function bootLogsActivity()` in `LogsActivity` trait 

{% hint style="info" %}
Only when instantiating a model, and this model has not booted before in current request life cycle, it would boot! 
{% endhint %}

