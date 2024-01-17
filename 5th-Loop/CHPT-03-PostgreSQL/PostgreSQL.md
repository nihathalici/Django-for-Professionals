CH-3 / PostgreSQL
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