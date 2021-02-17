---
description: Tips about model
---

# Model

## model's boot

In a request life cycle, if any model are instantiating, it will first call `bootIfNotBooted()` function to boot the model class. As the function name indicates, it would execute boot process if this model **class** was not booted in this request life cycle.

{% hint style="info" %}
model boots is on the class level, not instance level. That means the same class of models at most boots once in one request cycle.
{% endhint %}

How to execute boot process? look at `bootTraits()` function here:

it will find all traits current model used \(_all traits used by a class, its parent classes and trait of their traits_\),  and **execute every trait's static function whose name is "boot\[Trait-Name\]"**. e.g:

 `protected static function bootLogsActivity()` in `LogsActivity` trait 

![flowchart](.gitbook/assets/image.png)

{% hint style="info" %}
Only when instantiating a model, and this model class has not booted before in current request life cycle, it would boot! 
{% endhint %}

{% hint style="info" %}
booted Model classes \(class full name\) are recorded in Model's static property`$booted` \(Model::booted\)
{% endhint %}

## Recursive Relationships \( parent - children tree \)

In reality, there are many cases that models are organized in a tree structure. for example, a team might has one parent team, and multiple child teams. The table in RDBMS to store those teams is like this:

![](.gitbook/assets/image%20%281%29.png)

How to get child teams in a nested and recursive way? Just define the relation in the Team model in this way:

{% code title="Team.php" %}
```php
public function childTeams()
{
    return $this->hasMany(Team::class, 'parent_id');
}

public function childTeamRec()
{
    return $this->childTeams()->with('childTeamRec')->orderBy('display_num');
}
```
{% endcode %}

Here is an example Laravel Command to print a team's ALL child teams in a tree structure.

{% code title="PrintTeamsStructure.php" %}
```php
<?php

namespace App\Console\Commands;

use App\Model\Team;
use Illuminate\Console\Command;

class PrintTeamsStructure extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'maint:print-teams-structure
                            {team_id : The root team ID}
                            {--I|indent= : indent(optional, integer that greater or equal to 1).default: 1}
                            {--D|detail_flag : flag to show team details(optional). }
    ';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Print teams structure';

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
     */
    public function handle()
    {
        $teamId =  $this->argument('team_id');
        $indent = (int)$this->option("indent");
        if ($indent<=0){
            $indent = 1;
        }
        $detailFlag = $this->option("detail_flag");

        $team1 = Team::with("childTeams")->find($teamId);
        $this->_printRec($team1, [], $indent, $detailFlag);
    }

    protected function _printRec(Team $team, $lastList=[], $indent = 1, bool $detailFlag=false)
    {
        $prefix = "";
        if (count($lastList) != 0){
            $countLastList = count($lastList);
            foreach ($lastList as $key => $isLast){
                if ($key == $countLastList-1) {
                    break;
                }

                $prefix .= ($isLast ? " " : "│").str_repeat(" ", $indent-1);
            }

            $prefix .= ($lastList[$countLastList-1] ? "└" : "├").str_repeat("─", $indent-1);
        }

        $teamInfo = $prefix.$team->id;
        if ($detailFlag){
            $teamInfo .= ":".$team->name;
        }

        $this->info($teamInfo);

        $childCount = $team->childTeams->count();
        foreach ($team->childTeams as $index => $childTeam){
            $last = $index == $childCount -1;
            $newLastList = array_merge($lastList, [$last]);

            $this->_printRec($childTeam, $newLastList, $indent, $detailFlag);
        }
    }
}


```
{% endcode %}

```bash
php artisan maint:print-teams-structure 1

1
├10
│└5
│ ├2
│ ├6
│ │└8
│ └7
└11
 └3
  ├4
  └9

php artisan maint:print-teams-structure 1 -I 2 -D

1:System Administrator
├─10:Test Team6
│ └─5:Test Team
│   ├─2:Operations Team
│   ├─6:Test Team2
│   │ └─8:Test Team4
│   └─7:Test Team3
└─11:Test Team7
  └─3:Marketing Operation
    ├─4:Finance Team
    └─9:Test Team5
```

