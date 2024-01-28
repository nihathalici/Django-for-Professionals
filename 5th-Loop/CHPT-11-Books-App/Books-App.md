CH-11 / Books App
========================================================

* Create the app named books

```shell
docker-compose exec web python manage.py startapp books
```

* Update the settings
```python
# django_project/settings.py 
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    "django.contrib.sites",
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Third-party
    "crispy_forms",
    "crispy_bootstrap5",
    "allauth",
    "allauth.account",
    # Local
    "accounts.apps.AccountsConfig",
    "pages.apps.PagesConfig",
    "books.apps.BooksConfig",  # new
]
```

* Write the model
```python
# books/models.py

from django.db import models

class Book(models.Model):
    titel = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=6, decimal_places=2)

    def __str__(self):
        return self.titel
```

* Create a new migration record.
```shell
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
```

* Edit admin.py
```python
# books/admin.py
from django.contrib import admin

from .models import Book

admin.site.register(Book)
```

* Go to http://127.0.0.1:8000/admin/ . Make a new entry.

* Update admin
```python
# books/admin.py

from django.contrib import admin

from .models import Book

class BookAdmin(admin.ModelAdmin):
    list_display = ("title", "author", "price",)


admin.site.register(Book, BookAdmin)

```

* Update the urls.py within the django_project directory
```python
# django_project/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    # Django admin
    path('admin/', admin.site.urls),
    # User management
    path("accounts/", include("allauth.urls")),
    # Local apps
    path("", include("pages.urls")),
    path("books/", include("books.urls")),
]
```

* Create a urls.py file for the books app
```python
# books/urls.py

from django.urls import path

from .views import BookListView

urlpatterns = [
    path("", BookListView.as_view(), name="book_list"),
]
```

* Write the view
```python
# books/views.py

from django.views.generic import ListView

from .models import Book

class BookListView(ListView):
    model = Book
    template_name = "books/book_list.html" 
```

* Within the templates directory create an app specific folder named books.
```shell
mkdir templates/books/
```

* Create the book_list.html file within the templates/books folder.
```html
<!-- templates/books/book_list.html -->
{% extends '_base.html' %}

{% block title %}Books{% endblock title %}

{% block content %}

{% for book in object_list %}
  <div>
    <h2><a href="">{{ book.title }}</a></h2>
  </div>    
{% endfor %}
{% endblock content %}    
```

* Restart the Docker containers.
```shell
docker-compose down
docker-compose up -d
```

* Check http://127.0.0.1:8000/books/ 

* Update views.py using context_object_name
```python
# books/views.py

# books/views.py

from django.views.generic import ListView

from .models import Book

class BookListView(ListView):
    model = Book
    context_object_name = "book_list"  # new
    template_name = "books/book_list.html"
```

* Modify the template file
```html
<!-- templates/books/book_list.html -->
{% extends '_base.html' %}

{% block title %}Books{% endblock title %}

{% block content %}

{% for book in book_list %}
  <div>
    <h2><a href="">{{ book.title }}</a></h2>
  </div>    
{% endfor %}
{% endblock content %}   
```

* Check http://127.0.0.1:8000/books/

Individual Book Page
========================================================

* Edit the urls.py within the books folder
```python

```
