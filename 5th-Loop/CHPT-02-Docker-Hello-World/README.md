CH-2 / Set-Up
========================================================


* Make a new directory

```shell
mkdir ch2-hello
```

* Create a virtual environment within this new directory. 

```shell
python -m venv .venv
```

* Activate the virtual environment called .venv
```shell
source .venv/bin/activate
```

* Install Django
```shell
pip install django
```

* Create a new Django project called django_project. 
```shell
django-admin startproject django_project .
```

* Create the database
```shell
python manage.py migrate
```

* Run the Django development server
```shell
python manage.py runserver
```

* Quit the server with CONTROL-C. Create the app pages
```shell
python manage.py startapp pages
```

* Edit the settings file and add the pages app 
```python3
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Local
    "pages.apps.PagesConfig",  # new
]
```

* Set the URL route for the pages app
```python3
# django_project/urls.py

from django.contrib import admin
from django.urls import path, include  # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("pages.urls")),  # new
]
```
* Write the view without a template page.
```python3
# pages/views.py
from django.http import HttpResponse

def home_page_view(request):
    return HttpResponse("Hello, World!")
```

* Create a urls.py file within the pages app.
```python3
# pages/urls.py

from django.urls import path
from .views import home_page_view 

urlpatterns = [
    path("", home_page_view, name="home"),
]
```

* Create a requirements.txt file.
```shell
pip freeze > requirements.txt
```


Switch to Docker
========================================================

* It consists of 5 steps:

* 1. create a Dockerfile
* 2. add a .dockerignorefile
* 3. build the image  - docker build .
* 4. create a docker-compose.yml file 
* 5. spin up the container(s)  - docker-compose up

* Deactivate the virtual environment.

```shell
deactivate
```

* Create the Dockerfile next to the manage.py file.
```Dockerfile
# Pull base image
FROM python:3.10.4-slim-bullseye
# Set environment variables
ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
# Set work directory
WORKDIR /code
# Install dependencies
COPY ./requirements.txt .
RUN pip install -r requirements.txt
# Copy project
COPY . .
```

* Create the .dockerignore next to the existing manage.py file.
```Dockerfile
.venv
.git
.gitignore
```

* Build the Dockerfile. Don't forget the period at the end.
```shell
docker build .
```

* Create a docker-compose.yml file next to the Dockerfile.
```Dockerfile
version: '3.9'
services:
  web:
    build: .
    command: python /code/manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - 8000:8000
```

* Last step: run Docker container.
```shell
docker-compose up
```
