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

### Acessing the admin web interface


Generating admin sites for your staff or clients to add, change, and delete content is tedious work that doesn’t require much creativity. For that reason, Django entirely automates creation of admin interfaces for models.

Django was written in a newsroom environment, with a very clear separation between “content publishers” and the “public” site. Site managers use the system to add news stories, events, sports scores, etc., and that content is displayed on the public site. Django solves the problem of creating a unified interface for site administrators to edit content.

The admin isn’t intended to be used by site visitors. It’s for site managers.


```bash 
python manage.py createsuperuser
```

fill the informations asked in the interactive formulaire and run the developpement server :

```bash 
python manage.py runserver
```

and go to the page <siteURL>/admin

you may need to allow CRSF tokens for other hosts if the server is not running locally by adding in ```settings.py``` the variable ```CSRF_TRUSTED_ORIGINS```

```python
CSRF_TRUSTED_ORIGINS = ["https://redoules-organic-xylophone-j64vr4ppw7f7gj-8000.preview.app.github.dev"]
```


Make the Polls model editable by the admin by modifiying the ```admin.py``` file in the polls folder:

```python 
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```


### Creating views for the polls application 

A view is a public page of the application. For a blog you may find view for the homepage, the article view, a view listing the articles published in a certain year and so on. 

For the polls application we will create the 4 views :
 * Question “index” page – displays the latest few questions.
 * Question “detail” page – displays a question text, with no results but with a form to vote.
 * Question “results” page – displays results for a particular question.
 * Vote action – handles voting for a particular choice in a particular question.

The views are created in the ```views.py``` : 

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

and have to be wired to a URL in the ```urls.py``` 

```python 
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

The urlpatterns defined above explain how to collect the arguments in the URL that are sent to the view parameters. 
The view has to return a HttpResponse or return a 404 error.

Let's create a view that can interact with the database by using the django API

```python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

Now, if we want to split the interaction with the database and the formatting of the HTML, we can use a template.
This can be done by creating a ```templates``` folder. Let's create another folder with the same name as the app (```polls```) and create a index.html file within it containing :

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

We can then update the view in order to load and populate the template.
Notice that the links are hard coded "/polls/questionid". In order to make the view independant form the route we can rewrite the template like this 

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```


A shorter way to write the view is by using a django shortcut ```render``` : 
```python 
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
    
```


#### Raising a 404 page 

Now, let’s tackle the question detail view – the page that displays the question text for a given poll. Here’s the view:


```python
from django.http import Http404
from django.shortcuts import render

from .models import Question


# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
    
```


a better way to do that is to use the shortcut ```get_object_or_404```

```python 
from django.shortcuts import get_object_or_404, render

from .models import Question


# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
    
```


here is the html code for the template 

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```


It is a good practice to specify a namespace with the variable ```app_name``` in the ```urls.py``` file so that django explicitly know which view to load 

```python 
from django.urls import path

from . import views


app_name = "polls"

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),

]

```

And change the index.html file to explicitly specify the app name :

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

### Working with views

#### Form

Let's replace the content of the detail.html template by : 
```html 
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```

We set the form’s action to {% url 'polls:vote' question.id %}, and we set method="post". Using method="post" (as opposed to method="get") is very important, because the act of submitting this form will alter data server-side. Whenever you create a form that alters data server-side, use method="post". This tip isn’t specific to Django; it’s good web development practice in general.

Since we’re creating a POST form (which can have the effect of modifying data), we need to worry about Cross Site Request Forgeries. Thankfully, you don’t have to worry too hard, because Django comes with a helpful system for protecting against it. In short, all POST forms that are targeted at internal URLs should use the {% csrf_token %} template tag.

Now, let’s create a Django view that handles the submitted data and does something with it. In the ```urls.py``` file the following line links the vote view to the url questionid/vote/
```python
path("<int:question_id>/vote/", views.vote, name="vote"),
```

we now need to implemente the vote function in the ```views.py``` file

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```


This function deals with POST requests so we need to use the reverse function to return the previous URL and return a HttpResponseRedirect as a result. 
This view calls request.POST in order to gather the data that has to be changed server side, if that data is not present, the view returns the detail page with an error message.
In the else branch, the view updates the database with the user vote and returns the HttpResponseRedirect.


Finally, we add the template for the results page templates/polls/results.html: 

```html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

```

and the code for the view in ```views.py```

```python

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})

```

#### Generic views

Django provides a shortcut for creating views that display a model object’s detail page and a list of a particular type of object. These views are called generic views.

We first edit the ```urls.py``` file to use the generic views 

```python
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

We then edit the ```views.py``` file to use the generic views 

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"


def vote(request, question_id):
    ...  # same as above, no changes needed.
```

### Writing tests

Django integrates a test framework that can be used to test functionnalities of the application. We will write tests for the polls application, we will test the ```was_published_recently()```. That function returns true if the question was published in the last 24 hours but will also return true if the question is published in the future.

The tests are written in the ```tests.py``` file of the polls app. 

```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

to run the test, use the command 

```bash
python manage.py test polls
```

which results in 

```
Found 1 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
was_published_recently() returns False for questions whose pub_date
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/workspaces/djangotuto/djangotuto/polls/tests.py", line 17, in test_was_published_recently_with_future_question
    self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```


In order to fix the bug we have to edit the ```was_published_recently``` method in the ```Question``` model:
    
```python 
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

and run the test again 

```
Found 1 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
```

We can write more tests 

```python
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)


def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
``` 


We can as well test the view. Django provides a test Client to simulate a user interacting with the code at the view level. We can use it in ```tests.py```  or even in the shell.

```
python manage.py shell
```

```python
from django.test.utils import setup_test_environment
setup_test_environment() #installs a template renderer which will allow us to examine some additional attributes on responses such as response.context that otherwise wouldn’t be available

from django.test import Client
client = Client()


# get a response from '/'
response = client.get("/")
#Not Found: /
# we should expect a 404 from that address; if you instead see an
# "Invalid HTTP_HOST header" error and a 400 response, you probably
# omitted the setup_test_environment() call described earlier.
response.status_code
#404
# on the other hand we should expect to find something at '/polls/'
# we'll use 'reverse()' rather than a hardcoded URL
from django.urls import reverse
response = client.get(reverse("polls:index"))
response.status_code
#200
response.content
#b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#x27;s up?</a></li>\n    \n    </ul>\n\n'
response.context["latest_question_list"]
#<QuerySet [<Question: What's up?>]>
```

Let's imporve the view (by filtering polls that are in the future) and write tests that make sure the behaviour is well implemented.

In the ```views.py``` file, we need to edit the function ```get_queryset``` in order to check that the question is not published in the future. 

```python
from django.utils import timezone
def get_queryset(self):
    """
    Return the last five published questions (not including those set to be
    published in the future).
    """
    return Question.objects.filter(pub_date__lte=timezone.now()).order_by("-pub_date")[:5]
```

Now you can satisfy yourself that this behaves as expected by :
 1. firing up runserver, 
 2. loading the site in your browser, 
 3. creating Questions with dates in the past and future, 
 4. and checking that only those that have been published are listed.
You don’t want to have to do that every single time you make any change that might affect this - so let’s also create a test, based on our shell session above.

in the file ```tests.py``` we can add the following test : 

```python

from django.urls import reverse

...

def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse("polls:index"))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        question = create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        question = create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question2, question1],
        )
```

Let's test the DetailView as well, we want to avoid a user guessing the URL of a question that is published in the futur and starting voting.
We modify the queryset of the detailview

```python
class DetailView(generic.DetailView):
    ...

    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```

and add the following test in ```tests.py```

```python
class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text="Future question.", days=5)
        url = reverse("polls:detail", args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text="Past Question.", days=-5)
        url = reverse("polls:detail", args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
        
```