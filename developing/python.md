# Contents

- [Asyncio](#asyncio)
    - [async loop through list](#async-loop-through-list)
- [Classes](#classes)
    - [general](#general)
        - [initialization](#initialization)
        - [importing](#importing)
        - [__str__ and __repr__](#__str__-and-__repr__)
        - [call parent class from child class with the same name](#call-parent-class-from-child-class-with-the-same-name)
        - [metaclasses](#metaclasses)
    - [binary](#binary)
        - [convert decimal to binary](#convert-decimal-to-binary)
        - [shift](#shift)
    - [dictionaries](#dictionaries)
        - [setdefault](#setdefault)
    - [strings](#strings)
        - [format](#format)
- [functions](#functions)
    - [arguments](#arguments)
        - [default values](#default-values)
    - [get function signature](#get-function-signature)
- [decorators](#decorators)
    - [decorator with args](#decorator-with-args)
- [log](#log)
    - [Logger](#logger)
- [tests](#tests)
    - [assert](#assert)
        - [check number of database calls](#check-number-of-database-calls)
        - [assertRaises](#assertraises)
    - [mocking](#mocking)
        - [basics](#basics)
        - [usage](#usage)
        - [examples](#examples)
            - [patch class attribute or method](#patch-class-attribute-or-method)
            - [patch stand alone function and test arguments](#patch-stand-alone-function-and-test-arguments)
        - [return values](#return-values)
        - [return different value on each call](#return-different-value-on-each-call)
        - [mocking immutable built-ins](#mocking-immutable-built-ins)
        - [mocking models](#mocking-models)
        - [check method call without mocking](#check-method-call-without-mocking)
- [additional tools](#additional-tools)
    - [pytest](#pytest)
        - [fix databases](#fix-databases)
        - [fix Django](#fix-django)
        - [test dir](#test-dir)
        - [supress warnings](#supress-warnings)
        - [supress excessive logs](#supress-excessive-logs)
    - [poetry](#poetry)
        - [use system packages](#use-system-packages)
        - [add files from private git repo](#add-files-from-private-git-repo)
        - [packets version specification](#packets-version-specification)
    - [virtual envs](#virtual-envs)
        - [create venv](#create-venv)
        - [install version of Python](#install-version-of-python)
        - [create pyenv](#create-pyenv)
        - [activate pyenv](#activate-pyenv)
        - [auto-activate](#auto-activate)
    - [package manager](#package-manager)
        - [pip](#pip)
            - [install requirements](#install-requirements)
            - [upgrade all](#upgrade-all)
            - [space](#space)
            - [use index-url](#use-index-url)
            - [from git](#from-git)
- [receipts](#receipts)
    - [web server](#web-server)
    - [decode base64](#decode-base64)
    - [decode cyrillic](#decode-cyrillic)
    - [get python version](#get-python-version)
    - [encode URL](#encode-url)
    - [convert list to list of pairs](#convert-list-to-list-of-pairs)
    - [get first item from list with condition](#get-first-item-from-list-with-condition)
    - [set dictionary value if it isn't set](#set-dictionary-value-if-it-isnt-set)
- [troubleshooting](#troubleshooting)
    - [asdf conflicts with AUR helper](#asdf-conflicts-with-aur-helper)
    - [no pip in venv](#no-pip-in-venv)
    - [psycopg2](#psycopg2)
    - [no __version__ when installing](#no-__version__-when-installing)
- [Machine Learning](#machine-learning)
    - [cor function](#cor-function)
    - [try](#try)
- [Typecheking](#typecheking)

# Asyncio

## async loop through list
```python
results = await asyncio.gather(*map(<func>, <list>))
```


# Classes

## general

### initialization
- `Foo(*args, **kwargs)` means `Foo.__call__(*args, **kwargs)`
- object `Foo` is an instance of class `type`, so calling `Foo.__call__(*args, **kwargs)` is equal to `type.__call__(Foo, *args, **kwargs)`
- calling `type.__call__(Foo, *args, **kwargs)` calls `type.__new__(Foo, *args, **kwargs)` which returns `obj`
- `obj` is initialied during calling `obj.__init__(*args, **kwargs)`
- result of the whole process - initialized `obj`

`__new__` method allocates memory and returns object   

it could be customized to achive desired results   

**Singletone** pattern example
```python
    class Singleton(object):
        _instance = None

        def __new__(cls, *args, **kwargs):
            if cls._instance is None:
                cls._instance = super().__new__(cls, *args, **kwargs)
            return cls._instance
```
```python
    >>> s1 = Singleton()
    ... s2 = Singleton()
    ... s1 is s2
    True
```
returns the same instance if it's already has been created (note that `__init__` method would be called for each `Singletone()` call)   

**Borg** pattern example   
```python
    class Borg(object):
        _dict = None

        def __new__(cls, *args, **kwargs):
            obj = super().__new__(cls, *args, **kwargs)
            if cls._dict is None:
                cls._dict = obj.__dict__
            else:
                obj.__dict__ = cls._dict
            return obj
```
```python
    >>> b1 = Borg()
    ... b2 = Borg()
    ... b1 is b2
    False
    >>> b1.x = 8
    ... b2.x
    8
```
instances would be different, but share same data   

### importing
`package1.py`
```python
   class A(object):
     def __init__(self):
       ...
```
`package2.py`
```python
   from package1 import A

   class B(object):
     def __init__(self):
       ...
```
in this case in `package2` a variable named `A` is created and points to actual `A` class in memory.
`package1` also contains a variable named `A` which also points to class `A` in memory.
```
   package1.A --+
                |----> <actual class A>
   package2.A---+
```
now when `A` is referred from `B` class, interpreter will attempt to find an `A` variable in   
namespace of `package1` and use it to get to class in memory.

### __str__ and __repr__
They are useful to get information about object in log/debug.   

Instead of just the name of object's class, some useful info (like state of variables) can be represented.
* `__repr__` should be unambiguous, e.g. `log("MyClass(this=%r, that=%r)" % (self.this, self.that))` (**NOTE**: `%r` is mandatory to get `__repr__` of objects)
* `__str__` should be reliable, i.e. human-readable. If `__str__` is not implemented, `__str__ = __repr__` is assumed.

### call parent class from child class with the same name
* Python 3
```python
   def method():
       super().method()
```
* Pyhton 2
```python
    def method():
        super(ClassName, self).method()
```

### metaclasses
**Metaclasses** are class of classes.
```python
MyClass = MyMetaClass()
my_object = MyClass()
```
`type` are actually metaclasses, used to create other classes.

```python
type(name, bases, attrs)
```
* `name` name of the class
* `bases` tuple of the parent class (for inheritance, can be empty)
* `attrs` dictionary containing attributes names and values

[more](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python/6581949#6581949)

## binary

### convert decimal to binary
* `f'{X:08b}'` where `X` is decimal number

### shift
*Shift* is useful to check if `N`th bit is `1` or `0`:  
`<bit ownership> & (1 << N)`
E.g.  
```
8 & (1 << 3)  
8 -> 01000 
       \____ 3rd position

1 << 3 => 0001 << 3 => 1000

01000 & 01000 => 01000 => 8
```
or  
```
8 & (1 << 2)  
8 -> 01000 
       \____ 3rd position

1 << 2 => 0001 << 2 => 0100
                         \___ 2nd position

01000 & 00100 => 00000 => 0
```

## dictionaries

### setdefault
Returns specified key. If key doesn't exist, inserts the key with a given value.  
```python
x = my_dict.setdefault('some_key', 'default_value')
```
will return the real value, if `some_key` presents in dict, otherwise `default_value`


## strings

### format
* basic form (use only for python < 2.5)
  `"string %s" % var`   

  takes only single **var** or **tuple**, so to avoid errors, if **var** can be list/tuple/etc `"string %s" % (var,)` must be used.
* format
  `"string {}".format(var)`
* f-string (use only for python >3)
  `f'string {var}'`
  


# functions

## arguments
### default values
```python
b=1
def foo(a=b):
  print (a)

foo()
b=2
foo()
```
output
```
1
1
```
Default arguments getting values at the moment of function creation.
But if argument's default is _mutable_ (e.g. `list`), value can be changed.

## get function signature
```python
import inspect

inspect.signature(<func>)
```


# decorators

## decorator with args
```python
def decorator(decorator_arg):
    def actual_decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if decorator_arg:
                ...
            return func(*args, **kwargs)
        return wrapper
    return actual_decorator
```


# log

`<log_message> -> Logger -> Handler -> <destination>`

* `Filter` controls log output
* `Formatter` controls how output is formatted

## Logger
    Loggers should NEVER be instantiated directly,
    but always through the module-level function `logging.getLogger(name)`.
    Multiple calls to `getLogger()` with the same name will always return a reference to the same `Logger` object.


# tests

## assert
### check number of database calls
`assertNumQueries(<num>, <func>)`
```python
with assertNumQueries(<num>):
    ...
```
**NOTE**: doesn't support `msg`!

### assertRaises
`assertRaises(<exception>, <callable>, <argument>)`

it is important to pass `<callable>`, not `<function()>`

## mocking

### basics
To patch **instance** attribute or method  
```python
   from package2 import B

   class TestB:
     @mock.patch('package2.A')  # replace the memory address that package2.A points to (now it's instance of the Mock class)
     def test_b(self, mock_A):
       test = B()
```
```
   package1.A ------> <actual class A>
   package2.A---+
                |----> <Mock object>
   mock_A-------+
```

`mock_A.return_value` contains instance of `mock_A`   

`mock.patch(target, ...)`   

`target` must be in form `package.module.ClassName`   

It's important to do a proper import, because
```python
   import random
   from random import choice

   def a():
       return choice([1,2,3])

   def b():
       return random.choice([1,2,3])

    with mock.patch('random.choice', return_value=1000):
        print('a', a())
        print('b', b())
```
result will be
```python
   >>> a 3
   >>> b 1000
```

**NOTE** be aware if mocked function is inside a `.py` file inside folder,  
the file **should not** be called as one of methods inside it, e.g.  

```python
<project>/<folder>/__init__.py

from .some_module import some_module

<project>/<folder>/some_module.py
def some_module():
  ...
```
In this case, `some_module` method won't be patched!

### usage

* straightforward:
```python
'<some>.<module>.<func>' = mock.Mock(return_value='<something>')
'<some>.<module>.<func>' = mock.Mock(side_effect='<some_mock_func>')
```

* as context manager:
```python
with mock.patch('some.module.func', return_value='<something>':
    some.module.func()
```

* as decorator:
```python
@mock.patch('some.module.func', return_value='<something>')
def test(self, m_func):
    some.module.func()
```

### examples

#### patch class attribute or method
To patch **class** attribute or method
```python
  @mock.patch.object(Class, 'attribute/method')
  def test(self, mocked_attribute/method):
    mocked_method.side_effect=...
    mocked_method.return_value=...
```
also, mocked object/method can be specified
```python
  def faked_method(self):
    ...

  @mock.patch.object(Class, 'attribute/method', faked_method)
  def test(self):
    mocked_method.side_effect=...
    mocked_method.return_value=...
```

in this case `attribute` is **Class** attribute. To patch **instance** attribute   

`@patch.object(Class, method)` must be used **?**

#### patch stand alone function and test arguments
```python
@mock.patch('<project>.<path>.<module>.<method>')
def test(self, m_method):
    m_method.assert_called_with(<attributes>)
```

### return values
to mock return value `mock_instance.some_method.return_value` could be used   

to assign a function as a return value `mock_instance.some_method.side_effect` could be used

### return different value on each call
`mock_instance.some_method.side_effect = [<return_value_1>, <return_value_2>, ...]`

### mocking immutable built-ins
assuming:
`some_module.py`
```python
import datetime

def some_method():
  now = datetime.datetime.utcnow()
  ...
```
to patch built-in `utcnow()`:
`test.py`
```python
import some_module

def test():
    with mock.patch('some_module.datetime.datetime', mock.Mock(utcnow=mock.Mock(return_value=<mocked_date>))):
      some_module.some_method()
      ...
```

### mocking models
`pip install model_bakery`
```python
from model_bakery import baker
from ...models import Model

class SomeTestCase(TestCase):
    def setUp(self):
        self.mocked_model = baker.make(
            Model,
            define_field='...',
        )
```

### check method call without mocking
```python
@mock.patch(
'path.to.method',
side_effect='path.to.method',
)  # behaviour of mocked method is not changed, but now it's possible to check args it's called with
```


# additional tools

## pytest
### fix databases
If no db is used and Django is used, than `SimpleTestCase` should be used.  
And `DATABASES = {}` in `settings.py`

### fix Django
If `Apps not ready` error occured:
* install `pytest-django`
* `pytest --ds=<APP>.settings`

### test dir
if source code is in e.g. `./src` and tests are in `./tests`, either:
- `export PYTHONPATH="$PYTHONPATH:$PWD"`
- add to test `.py`
    ```
    import sys
    import os
    sys.path.append(os.path.abspath('../src/pkg'))
    ```
is needed

### supress warnings
`pytest -p no:warnings`

### supress excessive logs
`pytest  --show-capture=no`


## poetry

### use system packages
`<venv>/pyvenv.conf`:  
`include-system-site-packages = true`

### add files from private git repo
`poetry add git+ssh://git@github.com:<owner>/<repo>.git#<branch>&subdirectory=<path>'`
or in `pyproject.toml`: `<package> = {git = "git@github.com:<owner>/<repo>.git", rev = "<branch>", subdirectory = "<path>"}`

### packets version specification
General rule: if major/minor/patch version is explicitly defined,
then it is frozen except the last digit, e.g.:
- `~=1.2.3` allows `>=1.2.3 <1.3.0` i.e. _patch_ version can be updated, _minor_ is fixed
- `~=1.2` allows `>=1.2.0 < 2.0.0`

Wildcards:
- `*` allows `>=0.0.0`
- `1.*` allows `>=1.0.0 <2.0.0`
- `1.2.*` allows `>=1.2.0 <1.3.0`

Inequality:
`>= > < !=` can be used

Multiple:
`>=1.2, <1.5`

Caret (unsupported by **poetry!**):
_allows update except left-most non-zero digit_
- `^1.2.3` is converted to `>=1.2.3 <2.0.0`
- `^1.2` is converted to `>=1.2.0 <2.0.0`
- `^1` is converted to `>=1.0.0 <2.0.0`
- `^0.2.3` is converted to `>=0.2.3 <0.3.0`
- `^0.0.3` is converted to `>=0.0.3 <0.0.4`
- `^0.0` is converted to `>=0.0.0 <0.1.0`
- `^0` is converted to `>=0.0.0 <1.0.0`

Tilde (unsupported by **poetry**):
- `~1.2.3` is converted to `>=1.2.3 <1.3.0`
- `~1.2` is converted to `>=1.2.0 <1.3.0`
- `~1` is converted to `>=1.0.0 <2.0.0`


## virtual envs
### create venv
`python -m venv $venv`

useful options:  
- `--system-site-packages`
- `--prompt` changes venv prompt

### install version of Python
`pyenv install <version>`
### create pyenv
`pyenv virtualenv <python_version> <venv_name>`
### activate pyenv
`pyenv activate <venv_name>`
### auto-activate
add `<venv_name>` to `.python-version` in dir

## package manager
### pip
#### install requirements
`pip install -r requirements.txt`
`find ./ -name requirements.txt -exec pip install -r '{}' \;`
                
#### upgrade all
`pip list --outdated --format=freeze | grep -v '^\-e' | cut -d = -f 1  | xargs -n1 pip install -U`

#### space
`TMPDIR=/home/user/tmp/ python3 -m pip install a_package`

#### use index-url
`pip install <package> --index-url <url>`  
or
```
requirements.txt

<pac1>
<pac2>
--index-url <url>
```

#### from git
```
requirements.txt

<package>  @ git+https://github.com/owner/repo@branch
```


# receipts

## web server
`python -m http.server --cgi 8000`

## decode base64
```python
    import base64
    coded_string = '''Q5YACgA...'''
    base64.b64decode(coded_string)
```

## decode cyrillic
```python
<raw string>.encode('l1').decode()
```

## get python version
```python
    import sys
    print(sys.version)
    print(sys.executable)
```

## encode URL
```python
import urllib.parse

print(urllib.parse.quote("http://www.sample.com/", safe=""))
```

## convert list to list of pairs
```python
def pairwise(t):
    it = iter(t)
        return zip(it,it)

# for "pairs" of any length
def chunkwise(t, size=2):
    it = iter(t)
        return zip(*[it]*size)
```

## get first item from list with condition
```python
item = next(filter(<condition, e.g. lambda>, list), <default value>)
```

## set dictionary value if it isn't set
```python
some_dict.setdefault(<key>, <value>)
```


# troubleshooting

## asdf conflicts with AUR helper
`PATH=$(getconf PATH) <aur helper>`

## no pip in venv
`python -m venv --upgrade venv`

## psycopg2
* `pg_config` not found: install `postgresql-libs` (Arch) or `libqp-dev` (Ubuntu) or `libpqxx-devel` (Fedora)

## no __version__ when installing
check Python version


# Machine Learning

## cor function
```
plt.figure(figsize=(11,5))
cor = data.corr()['target'].sort_values(ascending = True)
plt.barh(cor.index[:-1], cor[:-1])
plt.yticks(fontsize = 15)
plt.xticks(fontsize = 12)
plt.xlabel('Correaltion', fontsize = 15)
plt.ylabel('Feature', fontsize = 15)
plt.title (label='correlation', fontsize = 20)
plt.show()
```

## try
* SAM model from META AI



# Typecheking
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from smth import Smth
```
