CH-3 / Set-Up
========================================================

* Make a new directory

```shell
mkdir ch3-postgresql
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
pip install django~=4.0.0
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

* Create a requirements.txt file.
```shell
pip freeze > requirements.txt
```

Switch to Docker
========================================================

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

* Run Docker container in detached mode.
```shell
docker-compose up -d
```

* Check it in your web browser: http://127.0.0.1:8000/

* Create a superuser account.
```shell
docker-compose exec web python manage.py createsuperuser
```

* Navigate into the admin at http://127.0.0.1:8000/admin and log in.

PostgreSQL
========================================================

* Stop running the Docker container.

```shell
docker-compose down
```


* Edit the requirements.txt file and add psycopg2-binary.
```txt
asgiref==3.5.2
Django==4.0.4
sqlparse==0.4.2
psycopg2-binary==2.9.3
```

* Update the file docker-compose.yml.
```docker-compose.yml
version: '3.9'
services:
  web:
    build: .
    command: python /code/manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - 8000:8000
    depends_on:
      - db
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - "POSTGRES_HOST_AUTH_METHOD=trust"

volumes:
  postgres_data:
```

* Modify the settings.py
```python3
# django_project/settings.py

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "postgres",
        "USER": "postgres",
        "PASSWORD": "postgres",
        "HOST": "db",
        "PORT": 5432,
    }
}
```

* Build and spin up the two containers in detached mode.
```shell
docker-compose up -d --build
```

* Check it in your web browser: http://127.0.0.1:8000/

* Refresh the database
```shell
docker-compose exec web python manage.py migrate
```

* Create the superuser account again.
```shell
docker-compose exec web python manage.py createsuperuser
```

* Navigate into the admin at http://127.0.0.1:8000/admin and log in.

* Stop the running container.
```shell
docker-compose down
```
