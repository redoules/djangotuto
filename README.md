# Tutorial django 

## Initialisation 

first install django
 
```bash
pip install django
```

then verify the version 

```bash
python -m django --version
```

If the version is greater than 4.2 create a project using the command 

```bash
django-admin startproject djangotuto
```

This will create a folder nammed djangotuto with the cookiecutter code used to start a django project.

## Project file structure

These files are:

 * The outer djangotuto/ root directory is a container for your project. Its name doesn’t matter to Django; you can rename it to anything you like.
**manage.py**: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about manage.py in django-admin and manage.py.

 * The inner djangotuto/ directory is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. djangotuto.urls).

    * **djangotuto/__init__.py**: An empty file that tells Python that this directory should be considered a Python package. If you’re a Python beginner, read more about packages in the official Python docs.

    * **djangotuto/settings.py**: Settings/configuration for this Django project. Django settings will tell you all about how settings work.

    * **djangotuto/urls.py**: The URL declarations for this Django project; a “table of contents” of your Django-powered site. You can read more about URLs in URL dispatcher.

    * **djangotuto/asgi.py**: An entry-point for ASGI-compatible web servers to serve your project. See How to deploy with ASGI for more details.

    * **djangotuto/wsgi.py**: An entry-point for WSGI-compatible web servers to serve your project. See How to deploy with WSGI for more details.


## Running the webserver 

during the developpement phase, use 
```bash 
python manage.py runserver
```
in order to launch the site.

In production, use a real webserver such as nginx or apache.

## Creating the poll app

We have perviouly created a project nammed ```djangotuto```, we can now add an application to this project with the command 

```bash
python manage.py startapp <myapp>
```

let's now create the polls app ```python manage.py startapp polls```.

This adds a new folder at the root of the ```djangotuto``` folder 

### creating a view

Django uses a MVC architecture, let's create the most simple view in the polls app view.py file 

```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

And map the view to a URL in the ```urls.py``` file of the polls app (not the main url.py file)

```python
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

After than, we need to root a URLconf from the main ```urls.py``` file to the app's URLconf by modifying the djangotuto's ```urls.py``` file

```python 
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```