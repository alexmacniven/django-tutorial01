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

## Playing with the API
- Use the built-in shell to update python path correctly; `$ python manage.py shell`
- We get a ORM to interact with the underlying database
    ```python
    >>> from polls.models import Question, Choice
    >>> from django.utils import timezone
    >>> q = Question(question_text="Whats good?", pub_date=timezone.now())
    >>> q.save()  # Writes changes
    >>> q.id
    1
    >>> q.question_text
    "Whats good?"
    >>> # Update field/attribute values
    >>> q.question_text = "Whats new?"
    >>> q.save()
    >>> # Display all question objects
    >>> Question.objects.all()
    <QuerySet [<Question: Question object (1)>]>
    ```
- Because models are objects, we can define functions/methods for them
    - It's good practice to add `__str__` methods to models for readability
    - For our models, this could be as simple as returning the text
        ```python
        # polls/models.py

        from django.db import models


        class Question(models.Models):

            ...

            def __str__(self) -> str:
                return self.question_text
        ```
    - We can also add any number of utility methods
        ```python
        # polls/models.py

        import datetime 

        from django.db import models
        from django.utils import timezone


        class Question(models.Model):
            
            ...

            def was_published_recently(self) -> bool:
                """Return if published within the last day."""
                # Note the use of timezone.now() not datetime.now()
                return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
        ```
- Lets have a closer look at some API functionality
    ```python
    >>> from polls.models import Choice, Question
    >>> Question.objects.all()
    <QuerySet [<Question: Whats good?>]> # __str__ works.
    >>> # Filter Question table by an id
    >>> Question.objects.filter(id=1)
    <QuerySet [<Question: Whats good?>]>
    >>> # Filter Question table by a pattern
    >>> Question.objects.filter(question_text__startswith="What")
    <QuerySet [<Question: Whats good?>]>
    >>> from django.utils import timezone
    >>> current_year = timezone.now().year
    >>> Question.objects.filter(pub_date__year=current_year)
    <QuerySet [<Question: Whats good?>]>
    >>> # Primary key look-up is most common
    >>> q = Question.objects.get(pk=1)
    >>> q
    <QuerySet [<Question: Whats good?>]>
    >>> # Return all choice objects related to question -- currently none
    >>> q.choice_set.all()
    <QuerySet []>
    >>> # Create some choices
    >>> q.choice_set.create(choice_text="Pizza", votes=0)
    <Choice: Pizza>
    >>> q.choice_set.create(choice_text="Coffee", votes=0)
    <Choice: Coffee>
    >>> c = q.choice_set.create(choice_text="Cats", votes=0)
    >>> c
    <Choice: Cats>
    >>> # Access question from choice
    >>> c.question
    <Question: Whats good?>
    >>> # Question now has related choices
    >>> q.choice_set.all()
    <QuerySet [<Choice: Pizza>, <Choice: Coffee>, <Choice: Cats>]>
    >>> q.choice_set.count()
    3
    >>> # API follows relationships without limitations.
    >>> # From Choice we can filter to find all choices for all questions whose
    >>> # publication date is the current year.
    >>> # Use dunder scores to separate relationships.
    >>> Choice.objects.filter(question__pub_date__year=current_year)
    <QuerySet [<Choice: Pizza>, <Choice: Coffee>, <Choice: Cats>]>
    >>> # Select and delete a choice
    >>> c = Choice.objects.filter(choice_text__startswith="Pizza")
    >>> c.delete()
    ```
    ```python
    >>> # Note how this filter returns the same Question twice.
    >>> # One for each Choice matching the filter query.
    >>> Question.objects.filter(choice__choice_text__startswith="Co")
    <QuerySet [<Question: Whats good?>, <Question: Whats good?>]>
    ```

## Introducing the Django Admin
- Create a superuser; `$ python manage.py createsuperuser`
- With a running dev server go to `http://127.0.0.1:8000/admin`
- The *Groups* and *Users* content is provided by `django.contrib.auth`
- To view our polls models we need to add them;
    ```python
    # polls/admin.py

    from django.contrib import admin

    from .models import Choice, Question


    admin.site.register(Question)
    admin.site.register(Choice)
    ```
- Now we have nice GUI for managing the polls app models