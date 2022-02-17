# Contents

- [Classes](#Classes)
    - [general](#Classes#general)
        - [initialization](#Classes#general#initialization)
        - [importing](#Classes#general#importing)
        - [__str__ and __repr__](#Classes#general#__str__ and __repr__)
        - [call parent class from child class with the same name](#Classes#general#call parent class from child class with the same name)
        - [metaclasses](#Classes#general#metaclasses)
    - [binary](#Classes#binary)
        - [convert decimal to binary](#Classes#binary#convert decimal to binary)
        - [shift](#Classes#binary#shift)
    - [strings](#Classes#strings)
        - [format](#Classes#strings#format)
- [functions](#functions)
    - [arguments](#functions#arguments)
        - [default values](#functions#arguments#default values)
    - [get function signature](#functions#get function signature)
- [decorators](#decorators)
    - [decorator with args](#decorators#decorator with args)
- [tests](#tests)
    - [assert](#tests#assert)
        - [check number of database calls](#tests#assert#check number of database calls)
        - [assertRaises](#tests#assert#assertRaises)
    - [mocking](#tests#mocking)
        - [basics](#tests#mocking#basics)
            - [patch class attribute or method](#tests#mocking#basics#patch class attribute or method)
            - [patch stand alone function and test arguments](#tests#mocking#basics#patch stand alone function and test arguments)
        - [return values](#tests#mocking#return values)
        - [return different value on each call](#tests#mocking#return different value on each call)
        - [mocking immutable built-ins](#tests#mocking#mocking immutable built-ins)
        - [mocking models](#tests#mocking#mocking models)
        - [check method call without mocking](#tests#mocking#check method call without mocking)
- [additional tools](#additional tools)
    - [virtual envs](#additional tools#virtual envs)
        - [create venv](#additional tools#virtual envs#create venv)
        - [install version of Python](#additional tools#virtual envs#install version of Python)
        - [create pyenv](#additional tools#virtual envs#create pyenv)
        - [activate pyenv](#additional tools#virtual envs#activate pyenv)
        - [auto-activate](#additional tools#virtual envs#auto-activate)
    - [package manager](#additional tools#package manager)
        - [pip](#additional tools#package manager#pip)
            - [install requirements](#additional tools#package manager#pip#install requirements)
            - [upgrade all](#additional tools#package manager#pip#upgrade all)
- [receipts](#receipts)
    - [web server](#receipts#web server)
    - [decode base64](#receipts#decode base64)
    - [decode cyrillic](#receipts#decode cyrillic)
    - [get python version](#receipts#get python version)
    - [encode URL](#receipts#encode URL)
    - [convert list to list of pairs](#receipts#convert list to list of pairs)
    - [get first item from list with condition](#receipts#get first item from list with condition)
    - [set dictionary value if it isn't set](#receipts#set dictionary value if it isn't set)

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
