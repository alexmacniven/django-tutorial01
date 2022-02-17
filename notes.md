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