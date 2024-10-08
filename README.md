[![Maintainability](https://github.com/snake-soft/django-oscar-pg-search/actions/workflows/django.yml/badge.svg)](https://github.com/snake-soft/django-oscar-pg-search/actions/workflows/django.yml)
[![Maintainability](https://api.codeclimate.com/v1/badges/a289293e4e1af1114d74/maintainability)](https://codeclimate.com/github/snake-soft/django-oscar-pg-search/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/a289293e4e1af1114d74/test_coverage)](https://codeclimate.com/github/snake-soft/django-oscar-pg-search/test_coverage)
[![image](https://codecov.io/gh/snake-soft/django-oscar-pg-search/branch/master/graph/badge.svg?token=TALCIZ5E3Q)](https://codecov.io/gh/snake-soft/django-oscar-pg-search)
[![image](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)


Postgresql search handler for Django-Oscar
==========================================
This creates a search handler without the need of any search backend. It is designed for the e-commerce framework [Oscar](https://github.com/django-oscar/django-oscar).

It is implemented a little bit expensive but uses 4 annotated search vectors: \* upc \* title \* meta\_description \* meta\_title

This way the search can be manipulated through the meta fields. It is running productive in a heavily customized env for many months now. I think it should scale up to 5000 Products with 10 Attributes depending on how the products are loaded. We use it fully lazy with endless scrolling.


To-Do
==========================================

- Provide a generic way to use filter forms
- Writing Tests


Features
==========================================

+ Don't need to use some additional search backend like elastic
+ Creates filters (facets) for:
    - Data that is directly attached to the Product model
    - including foreign key choices
    - AttributeValues of the products
    - StockRecord entries


Installation
==========================================

Basic
----------------------------------------------
Install using pip:

```bash
pip install django-oscar-pg-search
```

```python
# settings.py

# replace 'oscar.apps.search.apps.SearchConfig' by 'oscar_pg_search.apps.PgSearchConfig'
INSTALLED_APPS = [
	# 'oscar.apps.search.apps.SearchConfig',
    'oscar_pg_search.apps.PgSearchConfig',
]

OSCAR_PRODUCT_SEARCH_HANDLER = 'oscar_pg_search.postgres_search_handler.PostgresSearchHandler'

# To avoid exception, haystack is not used:
HAYSTACK_CONNECTIONS = {"default": {}}
```

Trigram search is our search algorithm. A migration is included to enable it at your database, if it isn't already. Run migrations to install it

```
python manage.py migrate django-oscar-pg-search
```

This ends up executing the following SQL:
```
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

To remove the extension, run this:
```
python manage.py migrate django-oscar-pg-search zero
```

Optional Search box
----------------------------------------------
To install the included Search box that passes the previous search term.
(eg. in 'oscar/partials/search.html')

```python
{% include 'oscar_pg_search/partials/search.html' %}
```

Optional Filter Forms
----------------------------------------------
To use the included filter forms, include them in the template eg. in 'oscar/layout_2_col.html' after '{% block column_left_extra %}{% endblock %}'.

```python
{% include 'oscar_pg_search/catalogue/partials/filter_forms.html' %}
```

and include the mixin into the catalogue views.

```python
# apps/catalogue/views.py
from oscar.apps.catalogue import views
from oscar_pg_search.mixins import SearchViewMixin


class CatalogueView(SearchViewMixin, views.CatalogueView):
    pass


class ProductCategoryView(SearchViewMixin, views.ProductCategoryView):
    pass

```


Optional Use Chosen.js
----------------------------------------------
To empower multiple choice fields with chosen load it eg. in 'oscar/base.html'.

```python
# At the beginning:
{% load oscar_pg_search %}

# After jQuery is loaded:
{% oscar_pg_search_base %}
```


Settings
==========================================

If you want to add some fields that are directly attached to the Product
model:

```python
# settings.py
OSCAR_ATTACHED_PRODUCT_FIELDS = ['is_public', 'deposit', 'volume', 'weight',]
```
