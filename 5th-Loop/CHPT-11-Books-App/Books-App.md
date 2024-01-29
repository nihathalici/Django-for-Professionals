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
# books/urls.py

from django.urls import path

from .views import BookListView, BookDetailView

urlpatterns = [
    path("", BookListView.as_view(), name="book_list"),
    path("<int:pk>/", BookDetailView.as_view(), name="book_detail"),
]
```

* Write the view
```python
# books/views.py

from django.views.generic import ListView, DetailView

from .models import Book

class BookListView(ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/book_list.html"

class BookDetailView(DetailView):
    model = Book
    template_name = "books/book_detail.html"
```

* Create the template file book_detail.html within the templates/books folder. 
```html
<!-- templates/books/book_detail.html -->
{% extends '_base.html' %}

{% block title %}{{ object.title }}{% endblock title %}

{% block content %}
  <div class="book-detail">
    <h2><a href="">{{ object.title }}</a></h2>
    <p>Author: {{ object.author }}</p>
    <p>Price: {{ object.price }}</p>
  </div>    
{% endblock content %}
```

* Check http://127.0.0.1:8000/books/1/

* Modify the view, and add context_object_name
```python
# books/views.py

from django.views.generic import ListView, DetailView

from .models import Book

class BookListView(ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/book_list.html"

class BookDetailView(DetailView):
    model = Book
    context_object_name = "book"   # new
    template_name = "books/book_detail.html"
```

* Update the template file
```html
<!-- templates/books/book_detail.html -->
{% extends '_base.html' %}

{% block title %}{{ object.title }}{% endblock title %}

{% block content %}
  <div class="book-detail">
    <h2><a href="">{{ book.title }}</a></h2>
    <p>Author: {{ book.author }}</p>
    <p>Price: {{ book.price }}</p>
  </div>    
{% endblock content %}
```

* Modify the template file book_list.html, point to detail page
```html
<!-- templates/books/book_list.html -->
{% extends '_base.html' %}

{% block title %}Books{% endblock title %}

{% block content %}

{% for book in book_list %}
  <div>
    <h2><a href="{% url 'book_detail' book.pk %}">{{ book.title }}</a></h2>
  </div>    
{% endfor %}
{% endblock content %}
```

* Check http://127.0.0.1:8000/books/

* Add links to the navbar in templates/_base.html.
```html
<!-- templates/_base.html -->
{% load static %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{% block title %}Bookstore{% endblock title %}</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
    <!-- CSS -->
    <link rel="stylesheet" href="{% static 'css/base.css' %}">
  </head>
  <body>
  <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
    <div class="container-fluid">
      <a class="navbar-brand" href="#">Bookstore</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
      data-bs-target="#navbarCollapse" aria-controls="navbarCollapse"
      aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarCollapse">
        <ul class="navbar-nav me-auto mb-2 mb-md-0">
          <li class="nav-item">
            <a class="nav-link" href="{% url 'book_list' %}">Books</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="{% url 'about' %}">About</a>
          </li>          
          {% if user.is_authenticated %}
            <li class="nav-item">
              <a class="nav-link" href="{% url 'account_logout' %}">Log Out</a>
            </li>
          {% else %}
          <li class="nav-item">
            <a class="nav-link" href="{% url 'account_login' %}">Log In</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="{% url 'account_signup' %}">Sign Up</a>
          </li>
          {% endif %} 
        </ul>
      </div>
    </div>
  </nav>  
    <div class="container">
    {% block content %}    
    {% endblock content %}    
    </div>
    <!-- Bootstrap JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
    <!-- JavaScript -->
    <script src="{% static 'js/base.js' %}"></script>
  </body>
</html>
```

* Check http://127.0.0.1:8000/

* Add the get_absolute_url method to the model
```python
# books/models.py

from django.db import models
from django.urls import reverse   # new

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=6, decimal_places=2)

    def __str__(self):
        return self.title
    
    def get_absolute_url(self):   # new
        return reverse("book_detail", args=[str(self.id)])

```


* Update the book_list.html
```html
<!-- templates/books/book_list.html -->
{% extends '_base.html' %}

{% block title %}Books{% endblock title %}

{% block content %}

{% for book in book_list %}
  <div>
    <h2><a href="{{ book.get_absolute_url }}">{{ book.title }}</a></h2>
  </div>    
{% endfor %}
{% endblock content %}

```

* Now switch from pk to uuid. First modify the model
```python
# books/models.py

import uuid
from django.db import models
from django.urls import reverse

class Book(models.Model):
    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False
    )
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=6, decimal_places=2)

    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse("book_detail", args=[str(self.id)])

```

* Modify the urls
```python
# books/urls.py

from django.urls import path

from .views import BookListView, BookDetailView

urlpatterns = [
    path("", BookListView.as_view(), name="book_list"),
    path("<uuid:pk>/", BookDetailView.as_view(), name="book_detail"),  #new
]

```

* Clean up the database
```shell
docker-compose exec web rm -r books/migrations
docker-compose down
```

* Have a look at the database, and go on with resetting the database 
```shell
docker volume ls

docker volume rm ch4-bookstore_postgres_data
docker-compose up -d
docker-compose exec web python manage.py makemigrations books
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
```

* Log in as admin and make new entries

* Check http://127.0.0.1:8000/
