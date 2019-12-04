# Pagination

## How to put previous request query string to links \(next\|previous, etc\)

when you build a paginator like this:

```text
$paginator = Model::query()
            ->orderByDesc("created_at")
            ->paginate(15)
        ;
```

you can use paginator's appends function, to put any query string into every link, like:

```text
$paginator->appends(['sort' => 'votes'])
```

You can get previous request query string using `request()->query()`, so combined by these two tricks, you can solve the problem by coding like this:

```text
$paginator = Model::query()
            ->orderByDesc("created_at")
            ->paginate(15)
            ->appends(request()->query())
        ;
```



