# Contents

- [Django Rest Framework](#django-rest-framework)
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
- [queryset](#queryset)
    - [values](#values)
- [timezone](#timezone)

# Django Rest Framework
## router

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

# queryset
## ArrayFiled
List of items.   
- `__contains` returns objects where the values passed are a subset of the data
- `__contained_by` return objects where the data is a subset of the values passed
- `__overlap` data shares any results with the values passed
- `__len` length of array

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
