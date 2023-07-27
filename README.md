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


### Using the django CLI inteface

It is possible to interact with django using a CLI inteface in an interactive python shell by running the command :

```bash 
python manage.py shell
```
here is an example of interaction with the API

```python 
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2023, 7, 27, 7, 24, 58, 117642, tzinfo=datetime.timezone.utc)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

In the REPL we see the string represention for the question as <Question: Question object (1)>. In order to make this more readable we can add a duner method to the model class. 

```python
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

It is possible to add convinence function to the model such as : 

```python 
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```


Now the interaction with the model with the API looks like this :
```python
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith="What")
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text="Not much", votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text="The sky", votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith="Just hacking")
>>> c.delete()
```

