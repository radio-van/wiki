# Contents

- [router](#router)
- [localization](#localization)
    - [make/update localization files](#makeupdate-localization-files)
- [migrations](#migrations)
- [models](#models)
    - [fields](#fields)
        - [attributes](#attributes)
        - [methods](#methods)
        - [custom fields](#custom-fields)
        - [DateTimeFiled](#datetimefiled)
- [queries](#queries)
    - [ArrayFiled](#arrayfiled)
    - [related fields](#related-fields)
        - [select_related](#select_related)
        - [prefetch_related](#prefetch_related)
    - [subquery](#subquery)
    - [union](#union)
    - [values](#values)
- [timezone](#timezone)

# router

| URL                                      | HTTP method                            | action                           | URL name                |
| ---------------------------------------- | -------------------------------------- | -------------------------------- | ----------------------- |
| [.format]                                | GET                                    | root view                        | api-root                |
|                                          |                                        |                                  |                         |
| {prefix}/[.format]                       | GET                                    | list                             | {basename}-list         |
|                                          | POST                                   | create                           |                         |
|                                          |                                        |                                  |                         |
| {prefix}/{methodname}/[.format]          | GET (or as specified by 'methods' arg) | '@list_route' decorated method   | {basename}-{methodname} |
|                                          |                                        |                                  |                         |
| {prefix}/{lookup}/[.format]              | GET                                    | retrieve                         |                         |
|                                          | PUT                                    | update                           | {basename}-detail       |
|                                          | PATCH                                  | partial_update                   |                         |
|                                          | DELETE                                 | destroy                          |                         |
|                                          |                                        |                                  |                         |
| {prefix}/{lookup}/{methodname}/[.format] | GET (or as specified by 'methods' arg) | '@detail_route' decorated method | {basename}-{methodname} |

# localization

## make/update localization files
* create/update `.po` files
`django-admin makemessages -l <lang>` (e.g. `ru_RU` or `ru`)
or  
`./manage.py makemessages -l <lang>`

* edit them (e.g. in `Poedit`)

* compile `.mo` files
`django-admin compilemessages`
or  
`./manage.py compilemessages`

# migrations
Reflect current **Model**'s state to DB.
			
`./manage.py makemigrations`
`./manage.py migrate`

# models
## fields
Fields translates Python object to data of appropriate type, that can be stored into given DB.
Fileds classes are stored in **Model**'s `Meta` class.
Fields classes translate data or pass data to **serializer**.

### attributes
`<Model>._meta.get_field('<field>').<attribute>` (e.g. `max_length`)

### methods
- `.to_python(value)` converts data from DB to Python object.
- `.get_prep_value(value)` converts Python object to DB data (DB type-agnostic).
- `.get_db_prep_value(value, connection)` is the same, but for specific DB type.
- `.value_to_string(value)` converts Python object to appropriate format for **serializer**.
- `._get_val_from_obj(obj)` used to get value from Python object. **DEPRECATED**
- `.value_from_object(obj)` the same, used in new Django.

### custom fields
Usually, for custom field two classes are needed:
* custom class for model's user, this class' instance is used for representation and operations with data when **Model**'s field is changed.
* `Filed` subclass to transform the class above to value for storing in DB or DB's value into Python object.

### DateTimeFiled
* `auto_now_add = True` is **not** overriding by explicit argument!

# queries
## ArrayFiled
List of items.   
- `__contains` returns objects where the values passed are a subset of the data
- `__contained_by` return objects where the data is a subset of the values passed
- `__overlap` data shares any results with the values passed
- `__len` length of array

## related fields
Related fields are **OneToOne**, **ManyToMany**, etc.
If field of a related object is called, a separate DB call is executed:
```python
class Author(models.Model):
    name = models.CharField()


class Book(models.Model):
    name = models.CharField()
    author = models.ForeignKey(Author, on_delete=models.CASCADE)


class Store(models.Model):
    name = models.CharField()
    books = models.ManyToManyField(Book)


qs = Book.objects.all()
books = []
for book in qs:  # author is a related object
    books.append({'name': book.name, 'author_name': book.author.name})

# result -> N calls to DB (N = number of books)

qs = Store.objects.all()
stores = []
for store in qs:
    books = [book.name for book in store.books.all()]
    stores.append({'name': store.name, 'books': books}]

# result -> N calls to DB (N = number of stores)
```

### select_related
is used when single object is going to be selected, i.e. forwards `ForeignKey`, `OneToOne`.
creates SQL **join** with fields of related object.
```python
# ...
qs = Book.objects.select_related('author').all()
# ...

# result -> 1 call to DB
```

### prefetch_related
is used when set of objects is going to be selected, i.e. forwards `ManyToMany`.
does a separate lookup for each relationship, and performs **joining** in Python (not in SQL).
```python
# ...
qs = Store.objects.prefetch_related('books')
# ...

# result -> 2 calls to DB
```
```python
# ...
for store in qs:
    books = [book.name for book in store.books.filter(price__range=(200, 300))]
    stores.append({'name': store.name, 'books': books}]
# ...

# result -> N+1 calls to DB (N iterates number of stores + 1 for prefetching)
# iteration is made because .filter() changes prefetched query
```
```python
# ...
qs = Store.objects.prefetch_related(
    Prefetch('books', queryset=Book.objects.filter(price__range=(200, 300))))
# ...

# result -> 2 calls to DB
```

To prevent additional queries (i.e. making SQL JOIN), `__` can be used:  
```python
books = Books.objects.all().values("name", "author__name")
```

If `QuerySet` is used to look in, it can lead to complex query:
```python
author_ids = Author.objects.filter(name__startwith='A').values_list("id", flat=True)
for book in Books.objects.filter(author__id__in=author_ids):
    print(book.name)
```
will produce:
```sql
select <fields> from books where books.author_id in (select author.id from authors where <condition>)
```
with great number of ids such query is very heavy.  
Two lighter queries can be used instead, if result of the first query is already calculated:
```python
# ...
for book in Books.object.filter(author__id__in=list(author_ids)):
# ...
```
but `list()` will produce a list with repeating identical ids, better use `set()` instead.  
## subquery
e.g.   
annotate Posts with recent emails, get emails only for selected Posts   

```python
newest = Comment.objects.filter(post=OuterRef('pk')).order_by('-created_at')
Post.objects.annotate(newest_commenter_email=Subquery(newest.values('email')[:1]))
```

## union
get only uniq items from all querysets

`qs_res = qs1 | qs2 | qs3`   

or   

`qs1.union(qs2)`

## values
* `.values('<value1>', '<value2>', ...)` returns only particular values from queryset

e.g.
```python
>>> qs.values('id', 'age')
Queryset[{'id': <id_value>, 'age': <age_value>}, {...}, ...]
```

* `.values_list(...)` do the same thing, but returns a listof tuples

e.g.
```python
>>> qs.values_list('id', 'age')
Queryset[(<id_value>, <age_value>), (...), ...]
```

if only one value passed to `values_list()` result will be
`Queryset[(<value>, ), (...), ...]`

if `flat=True` is used, then return object will contain only values
`Queryset[<value>, <value>, ...]`

Return object is `Queryset`, it can be converted to list with `list(...)`

# timezone
* `timezone.now()` returns `datetime` object with current timestamp according to Time Zone
* `timezone.timedelta(<offset>)` returns `timedelta` object,
  `<offset>` can be: `days`, `weeks`, `hours` etc
  
- `timezone.now() - timezone.timedelta(<offset>)` returns `datetime` object,
  which date is `<offset>` days/weeks/hours from now
- `timezone.now() - <datetime>` returns `timedelta` object,
  `timedelta.days` shows offset in days, etc
