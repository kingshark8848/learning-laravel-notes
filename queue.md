# Queue

## retry\_after in `config/queue.php`

{% hint style="info" %}
When a relevant job normally run past`retry_after`time, this job will be sent back to queue again. The running one will still run until fail itself or exceed timeout .
{% endhint %}

{% hint style="info" %}
`retry_after` parameter does not determine when to retry a unsuccessful job. e.g: a job thrown an exception, it will be re-tried \(if not exceed max tries\), and `retry_after` doesn't determine when to retry this job. 
{% endhint %}

{% hint style="info" %}
Its value should be at least a few seconds **larger** than max queue job **timeout** value. Or, the job will be executed twice or more. e.g: 

retry\_after = 60, timeout = 90, when job runs past 60 second, it will be sent to queue again, and previous job will not end until past 90 second. so its very possible the job executed twice.
{% endhint %}

## timeout

{% hint style="info" %}
When timeout happened, the job immediately failed \(no matter how many retries\).
{% endhint %}

## queue limit: about support of closure

Queue dispatching/construction cannot serialize **closure**. That means any object you passed into Job \(via dispatch or constructor\) cannot have **closure properties**, or _recursively_ have closure properties in his all object properties.

## Laravel Horizon

...

