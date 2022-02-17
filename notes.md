# Notes
This notes file documents the following of the [polls app tutorial](https://docs.djangoproject.com/en/4.0/intro/tutorial01/)

## Installation and project creation
- Install django with pipx then `$ django-admin startproject mysite` to create the template
- Navigate to `/mysite` then create a *venv* and install django inside it
- Check everything is working with `$ python manage.py runserver`
    - Change the ip/port by supplying it `$ python manage.py runserver [IP]:[PORT]`
    - Read more about [runserver](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)

## Creating the Polls app
- App is a web application that does something - e.g., blog system, database of records, small poll app
- Project is a collection of apps and configs
- Apps can be anywhere on your Python Path
- Create an app with `$ python manage.py startapp polls`

- Create a view by going to `polls/views.py` and adding
    ```python
    from django.http import HttpResponse

    def index(request):
        return HttpResponse("Hello, world. You're at the polls index.")
    ```
- Create `polls/urls.py` to house an URLconf which will allow us to call the view
- The URLconf needs to map the index view
    ```python
    from django.urls import path

    from . import views

    urlpatterns = [
        path("", views.index, name="index"),
    ]
    ```
- Next we need to point the *root* URLconf at the `polls.urls` module, do this my adding a mappin in `mysite/urls.py`
    ```python
    from django.contrib import admin
    from django.urls import include, path

    urlpatterns = [
        path('polls/', include('polls.urls')),
        path('admin/', admin.site.urls),
    ]
    ```
    - So when parsing an url, ` http://localhost:8000/polls/`, once we hit `polls/` the handler will hand-off to `polls.urls`
    - And because the remaining parts of the url are empty `views.index` in `polls` will be loaded


## Database setup
- We edit our database config in this section of `mysite/settings.py`
    ```python
    # Database
    # https://docs.djangoproject.com/en/4.0/ref/settings/#databases

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
    ```
    - You should check out the documentation when working with production databases
- Also check the list of default installed apps for our project
    ```python
    # Application definition

    INSTALLED_APPS = [
        'django.contrib.admin',  # The admin site
        'django.contrib.auth',  # An auth system
        'django.contrib.contenttypes',  # Content types framework
        'django.contrib.sessions',  # Session framework
        'django.contrib.messages',  # Messaging framework
        'django.contrib.staticfiles',  # Static file management framework
    ]
    ```
- Some of the apps will need a database table, create these with `$ python manage.py migrate`

## Creating models
- Models are essentially a database layout with metadata
- Next we create two models; *Question* and *Answer* 
    - A question has two fields, the question (text) and a publication date.
    ```python
    # polls/models.py
    
    from django.db import models


    class Question(models.Model):
        # The variable name is used for the field name, unless overridden by the first arg
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField("date published")  # First arg specifies a field name
    ```
    - A choice has fields, choice (text), a vote tally, as well as a link to a question
    ```python
    # polls/models.py

    from django.db import models


    class Choise(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)
    ```
- Add polls app to `INSTALLED_APPS`
    - Link the `PollsConfig` class from `polls.apps`
    ```python
    INSTALLED_APPS = [
        'polls.apps.PollsConfig',
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    ]
    ```
- Make new migrations to be applied `$ python migrate.py makemigrations`
- Apply new migrations `$ python migrate.py migrate`
- Why are these processes separate?
    - Allows for new migration files (polls\migrations\0001_initial.py) to be added to vcs
