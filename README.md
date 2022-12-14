# [addict-tracking-changes](https://github.com/Monallabs-org/addict-tracking-changes)

## Introduction

Originally, this repository was a fork of [addict](https://github.com/mewwts/addict) by Mats Julian Olsen.
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

This repo builds on original addict and adds contraptions to track key additions in the dictionary.
This features comes in quite handy in building reactive webapps where one has to respond 
to all the changes made on the browser. Addict-tracking-changes is the underpinning
data-structure in [ofjustpy](https://github.com/Monallabs-org/ofjustpy): a python based webframework build from [justpy](https://github.com/justpy-org/justpy)
 
The functions relevant to tracking changed history are:
`get_changed_history` and `clear_changed_history`. 
The `get_changed_history` returns an iterator of XPath style paths like `/a/b/c/e` (see [Demo example](#demo-example)). 

## Usage and examples

### Installation
This project is not on PyPI. Its a simple package with no third party dependency. 
Simply clone from github and include the source directory in your PYTHONPATH. 

### Demo example

```python
from addict import Dict
body = Dict()
body.query.filtered.query.match.description = 'addictive'
body.query.filtered.filter.term.created_by = 'Mats'

for changed_path in body.get_changed_history():
    #<work with changed_path>

body.clear_changed_history()
```

### Behaviour when values are instances of container types 
addict works as expected when the values of keys are simple data types (such as string, int, float, etc.). However, for container types such as dict, list, tuples, etc. the behaviour 
is somewhat differs. 

* dicts 
are treated as opaque object just like simple data types

```python
from addict import Dict

mydict = Dict()
mydict.a.b.c = {'kk': 1}
mydict.a.b.e = {'dd': 1}

for _ in mydict.get_changed_history():
    print(_) 
```
will print
```
/a/b/c
/a/b/e
```
and not 
```
/a/b/cc/kk
/a/b/e/dd
```

* lists
are seen as container, i.e., `get_changed_history` will report path for each element of
the list 

```python
from addict import Dict

mydict = Dict()

mydict.a.b = [1, [1]]
mydict.a.c = [2, [2, [3]]]
```
get_changed_history will report following paths:
```
            "/a/b/0",
            "/a/b/1/0",
            "/a/c/0",
            "/a/c/1/0",
            "/a/c/1/1/0",
```

* tuple 
tuple  behave same as dict

* sets
sets behave same as dict



## Known bugs and Caveats
1. Only tracks  field additions.  Deletions and updates are not tracked. 
2. freeze doesn't guards against deletions
3. building dict from another dict as shown in following expression wont' work
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

### EndNotes
- Docs (in readthedocs format): https://monallabs-org.github.io/addict-tracking-changes/#introduction

- Developed By: webworks.monallabs.in

