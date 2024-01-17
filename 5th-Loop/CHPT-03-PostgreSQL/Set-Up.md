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