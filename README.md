# Introduction

Originally, this repository was fork of [https://github.com/mewwts/addict](addict) by Mats Julian Olsen.
Overtime, it has substatially diverged in functionality and maintainence code that it seemed to make
sense to breakout as its own repository. 

addict provides an alternative and succient interface to manipulate a dictionary. This is especially
useful when dealing with heavily nested dictionaries. As example (taken from https://github.com/mewwts/addict)
a dictionary created using standard python dictionary interface looks as follows:
``python
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
to all the changes made on the browser. 
 



## Usage:

```python
mydict = Dict()

mydict.a.b.c = 15

for _ in mydict.get_changed_history():
    print(_) 
```
The output
```
.a.b.c
```

Use clear_changed_history to clear all changed history.

```
mydict.clear_changed_history()
for _ in mydict.get_changed_history():
    print(_)
```
outputs empty. 

## Key Points

1. Only tracks changes in values. Field additions and deletions are not being tracked. 
2. Every field keeps a `__tracker` variable to keep track of all its members that have changed. 


# TODO
1. History not valid after
pickle.load
2. building dict from another dict
following expression wont' work
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
