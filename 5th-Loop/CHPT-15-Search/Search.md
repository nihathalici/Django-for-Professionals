CH-15 / Search
========================================================

* Modify the urls.py within the books folder.


```python
# books/urls.py

from django.urls import path

from .views import BookListView, BookDetailView, SearchResultsListView  # new

urlpatterns = [
    path("", BookListView.as_view(), name="book_list"),
    path("<uuid:pk>/", BookDetailView.as_view(), name="book_detail"),
    path("search/", SearchResultsListView.as_view(), name="search_results"),  # new
]

```

* Write the view
```python
# books/views.py

from django.contrib.auth.mixins import (
    LoginRequiredMixin,
    PermissionRequiredMixin
)

from django.views.generic import ListView, DetailView

from .models import Book

class BookListView(LoginRequiredMixin, ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/book_list.html"
    login_url = "account_login"

class BookDetailView(
    LoginRequiredMixin,
    PermissionRequiredMixin,
    DetailView):
    model = Book
    context_object_name = "book"
    template_name = "books/book_detail.html"
    login_url = "account_login"
    permission_required = "books.special_status"

class SearchResultsListView(ListView):  # new
    model = Book
    context_object_name = "book_list"
    template_name = "books/search_results.html"
```

* Create the search_results.html file within the templates/books folder
```html
<!-- templates/books/search_results.html -->

{% extends '_base.html' %}

{% block title %}Search{% endblock title %}

{% block content %}
  <h1>Search Results</h1>
  
  {% for book in book_list %}
    <div>
        <h3><a href="{{ book.get_absolute_url }}">{{ book.title }}</a></h3>
        <p>Author: {{ book.author }}</p>
        <p>Price: $ {{ book.price }}</p>
    </div>  
  {% endfor %}    
{% endblock content %}
        
```

* Update the views, and add the Q objects
```python
...

from django.db.models import Q  # new

...

class SearchResultsListView(ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/search_results.html"

    def get_queryset(self):  # new
        return Book.objects.filter(
            Q(title__icontains="beginners") | Q(title__icontains="api")
        )
```

* Add a search form to the _base.html
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
        <form class="d-flex" action="{% url 'search_results' %}" method="get">
        <input class="form-control me-2" type="search" name="q" placeholder="Search" aria-label="Search">
        <button class="btn btn-outline-success" type="submit">Search</button>
        </form>
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

* Update the views
```python

...

class SearchResultsListView(ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/search_results.html"

    def get_queryset(self):
        query = self.request.GET.get("q")
        return Book.objects.filter(
            Q(title__icontains=query) | Q(author__icontains=query)
        )

```