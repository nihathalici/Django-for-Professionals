CH-20 / Reviews App
========================================================

* Write the model.


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

* Apply it to the database
```shell
docker-compose exec web python manage.py makemigrations books
docker-compose exec web python manage.py migrate
```

* Update the admin.py
```python
# books/admin.py

from django.contrib import admin

from .models import Book, Review

class ReviewInline(admin.TabularInline):
    model = Review

class BookAdmin(admin.ModelAdmin):
    inlines = [
        ReviewInline,
    ]
    list_display = ("title", "author", "price",)


admin.site.register(Book, BookAdmin)
```

* Create a testuser and add reviews 

* Update the book_detail.html file.
```html
<!-- templates/books/book_detail.html -->
{% extends '_base.html' %}

{% block title %}{{ book.title }}{% endblock title %}

{% block content %}
  <div class="book-detail">
    <h2><a href="">{{ book.title }}</a></h2>
    <p>Author: {{ book.author }}</p>
    <p>Price: {{ book.price }}</p>
  <div>
    <h3>Reviews</h3>
    <ul>
      
      {% for review in book.reviews.all %}
      <li>{{ review.review }} ({{ review.author }})</li>  
      {% endfor %}    
    </ul>
  </div>
  </div>    
{% endblock content %}
```