CH-3 / Switch to Docker
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
