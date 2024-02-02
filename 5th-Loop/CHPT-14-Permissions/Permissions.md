CH-14 / Permissions
========================================================

* Modify the views.


```python
# books/views.py

from django.contrib.auth.mixins import LoginRequiredMixin  # new
from django.views.generic import ListView, DetailView

from .models import Book

class BookListView(LoginRequiredMixin, ListView):  # new
    model = Book
    context_object_name = "book_list"
    template_name = "books/book_list.html"
    login_url = "account_login"  # new

class BookDetailView(LoginRequiredMixin, DetailView):  # new
    model = Book
    context_object_name = "book"
    template_name = "books/book_detail.html"
    login_url = "account_login"  # new
```

* Check http://127.0.0.1:8000/ 

* Modify the model
```python
# books/models.py

import uuid
from django.contrib.auth import get_user_model
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
    cover = models.ImageField(upload_to="covers/", blank=True)

    class Meta:  # new
        permissions = [
            ("special_status", "Can read all books"),
        ]

    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse("book_detail", args=[str(self.id)])


class Review(models.Model):
    book = models.ForeignKey(
        Book,
        on_delete=models.CASCADE,
        related_name="reviews",
    )
    review = models.CharField(max_length=255)
    author = models.ForeignKey(
        get_user_model(),
        on_delete=models.CASCADE,
    )
    def __str__(self):
        return self.review
```

* Refresh the database
```
docker-compose exec web python manage.py makemigrations books
docker-compose exec web python manage.py migrate
```

* Go to http://127.0.0.1:8000/admin/ Navigate to the Users section.
* Create a new user: special@email.com. Scroll down to User permissions
* Select books | book | Can read all books. Save it.

* Modify the views
```python

# books/views.py

from django.contrib.auth.mixins import (
    LoginRequiredMixin,
    PermissionRequiredMixin
)  # new

from django.views.generic import ListView, DetailView

from .models import Book

class BookListView(LoginRequiredMixin, ListView):
    model = Book
    context_object_name = "book_list"
    template_name = "books/book_list.html"
    login_url = "account_login"

class BookDetailView(LoginRequiredMixin, DetailView):
    model = Book
    context_object_name = "book"
    template_name = "books/book_detail.html"
    login_url = "account_login"
    permission_required = "books.special_status"  # new

```

* Try out with different users http://127.0.0.1:8000/books/