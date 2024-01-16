CH-2 / Switch to Docker
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