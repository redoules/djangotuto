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


### Creating the database 

Django can manage a database for the website it serves. This database can be managed by the automatically generated admin page.

In order to setup the database connexion, go into ```settings.py```; by default, django uses sqlite (which is fine for this tutorial). The database ENGINE is specified in the ```settings.py``` file where it can be changed to manage a sqlite, mysql, postgresql, etc. database.

The name of the database is also specified in the ```settings.py``` file, for a SQLite databse, it is the fullname of the file.


In order to initialise the database, run the command :
```bash 
python manage.py migrate
```

Setup the following variables 

```
LANGUAGE_CODE = "fr-FR"

TIME_ZONE = "Europe/Paris"
```

Another important section is the ```INSTALLED_APPS``` that are modules available to use in the project.


### Creating the model 

In this application we want to create two "models" one for the questions and another for the choices.

The models are defined in the ```models.py```, the different types of columns are defined with fonctions such as ```Charfield``` or ```DateTimeField```

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Once the model is created (or updated), the database has to be updated.
First, the installed apps have to be update (in ```settings.py```). The PollsConfig class is in the polls/apps.py file, so its dotted path is 'polls.apps.PollsConfig'

Then, in order to update the database, run 

```bash
python manage.py makemigrations polls
```
the output will be 
```
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

The migration is created in the file djangotuto/polls/migrations/0001_initial.py

and then run the command 

```bash
python manage.py sqlmigrate polls 0001
```

with the right migration number, the output is : 

```BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" datetime NOT NULL);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL, "question_id" bigint NOT NULL REFERENCES "polls_question" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
COMMIT;
```

The sqlmigrate command doesn’t actually run the migration on your database - instead, it prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.

Now you can run the migrate command :

```bash
python manage.py migrate
```
The migrate command takes all the migrations that haven’t been applied (Django tracks which ones are applied using a special table in your database called django_migrations) and runs them against your database - essentially, synchronizing the changes you made to your models with the schema in the database.

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```
