# Introduction

Originally, this repository was a fork of [https://github.com/mewwts/addict](addict) by Mats Julian Olsen.
Overtime, it has substatially diverged in functionality and codebase that it seemed to make
sense to breakout as its own repository. 

The original addict:  provides an alternative and succient interface to manipulate a dictionary. This is especially
useful when dealing with heavily nested dictionaries. As example (taken from https://github.com/mewwts/addict)
a dictionary created using standard python dictionary interface looks as follows:
```python
body = {
    'query': {
        'filtered': {
            'query': {
                'match': {'description': 'addictive'}
            },
            'filter': {
                'term': {'created_by': 'Mats'}
            }
        }
    }
}`

``` 
can be summarized to following three lines:

```python
body = Dict()
body.query.filtered.query.match.description = 'addictive'
body.query.filtered.filter.term.created_by = 'Mats'
```

This repo builds on original addict and adds contraptions to track changes in the addict. 
This features comes in quite handy for reactive webapps where one has to respond 
to all the changes made on the browser. Addict-tracking-changes is the underpinning
data-structure in ofjustpy: a python based webframework build from [](justpy)
 



## Usage

### Get and clear history
```python
mydict = Dict()

mydict.a.b.c = 15

for _ in mydict.get_changed_history():
    print(_) 
```
The output
```
/a/b/c
```

Use clear_changed_history method to clear all changed history.

```
mydict.clear_changed_history()
for _ in mydict.get_changed_history():
    print(_)
```
outputs empty. 

### Freeze/unfreeze 
Once a dict is frozen, adding new keys will raise KeyError. 
Although modification to existing keys is allowed 

To freeze:
```
mydict.freeze()
```

To unfreeze:
```
mydict.unfreeze()
```

### pickling
History is lost during pickling dump and load operation. 
To enable tracking after pickle.load, use set_tracker operation.


### Handling dict and list as values of keys
A py-dict is treated as any other opaque value object. 
Hence,
```python
mydict = Dict()

mydict.a.b.c = {'kk': 1}
mydict.a.b.e = {'dd': 1}

for _ in mydict.get_changed_history():
    print(_) 
```
will print paths
```
/a/b/c
/a/b/e
```
and not 
```
/a/b/cc/kk
/a/b/e/dd
```

List values on the other hand are exposed. Addict will walk within the list recursively,
an report all the list location. So, for following Dict with list values:
```
        trackerprop.a.b = [1, [1]]
        trackerprop.a.c = [2, [2, [3]]]
```
get_changed_history will report following paths:
```
            "/a/b/0",
            "/a/b/1/0",
            "/a/c/0",
            "/a/c/1/0",
            "/a/c/1/1/0",
```



## Caveats
1. Only tracks  field additions.  Deletions and updates are not tracked. 
2. freeze doesn't guards against deletions
3. 
4. building dict from another dict following expression wont' work
```python
cjs_cfg = Dict(x, track_changes=True)
```
instead use
```python 
cjs_cfg = Dict(track_changes = True)
with open("cjs_cfg.pickle", "rb") as fh:
    x = pickle.load(fh)
for _ in oj.dictWalker(x):
    oj.dnew(cjs_cfg, _[0], _[1])

```


