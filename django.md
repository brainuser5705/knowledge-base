# Setting up and basic request/response

Virtual environment:
`python -m venv venv`

Installation:

`python -m pip install django`

Creating a project:

`django-admin startproject <name>`

The structure of the project:

 ```
 <name>/ 			(container)
 	manage.py		(cmdline utility to interact with the project)
 	<name>/			(actual Python package to import)
 		__init__.py (tells Python that it is a Python package)
 		settings.py (settings for Django project)
 		urls.py		(contains root URLConfs)
 		asgi.py		(entry-point for ASGI-compatible servers)
 		wsgi.py		(entry-point for WSGI-compatible servers)
 ```

Run the development server:

`python manage.py runserver`

Creating an app

- apps can live anywhere on the Python path (tutorial puts it in the same directory as `manage.py`)
- apps can be distributed to multiple projects

`python manage.py startapp <name>`

The structure of the app:

```
<name>/
	__init__.py
	admin.py
	apps.py
	migrations/
		__init__.py
    models.py
    tests.py
    views.py
```

Writing view:

In `views.py`:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```

To call the view, we need to map it to a URL, thus we make a URLconf

Make `urls.py` in the app directory:

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index')
]
```

Next, point the root URLconf (path is defined as `ROOT_URLCONF` in `settings.py`) of the project directory to the urls.py module in the app directory:

In the urls.py of the project directory:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
	path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

When to use `include()`: when referencing other URLconfs
It chops off whatever matches the URL up to that point and sends the remaining string to the included URLconf for future processing

`path(route, view)`:
`route`: string that contains a URL pattern; Django starts at the first pattern then makes it way down the list until there is a match (doesn't search GET/POST parameters)
`view`: if there is URL match, it calls the view function with an HttpRequest object and any captured values from the route

# Database setup

`settings.py` in your project directory: contains module-level variables representing Django settings

Django by default uses SQLite since it is included in Python:
In `settings.py`, we have:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3', (database bindings)
        'NAME': BASE_DIR / 'db.sqlite3',		(name of the database file)
    }
}
```

Set `TIME_ZONE`

`INSTALLED APPS` holds the name of all the Django apps that are activated in this project.
There are default applications (which can be removed if not needed):

- `django.contrib.admin`: the admin site
- `django.contrib.auth`: the authentication system
- `django.contrib.contenttypes`: framework for content types
- `django.contrib.sessions`: session framework
- `django.contrib.messages`: messaging framework
- `django.contrib.staticfiles`: managing static files

Each application make use of at least one database table.

To make the database table:

`python manage.py migrate`

- loops at the `INSTALLED_APPS` setting and creates the database tables
- derived from models files which is used to update current database schemas

Creating models

```python
from django.db import models

class ModelName(models.Model):
    machine_readable_name = models.FieldClass('optional_name', param=argument)
    ...
```

- Each model is a subclass of `django.db.models.Model` and contains class variables, each which represents a database field in the model.
- Each class variable/field is a `Field` instance
  - the name of the instance is the `machine_readable_name` which the database will use as the column name
  - optional first parameter is the human-readable name
  - required and optional arguments

Working with Foreign fields

- When using a`ForeignKey` field, Django creates a set to hold the "other side" which can be accessed as `foreign_key.model_set.all()`
- Do `foreign_key.model_set.create(variables...)` which will insert the created object into the set and return the created object
- You can use double underscores to separate relationship:
  `Model.objects.filter(foreignkey_variable_attribute=value)`

Activating models

Tell the project that the app has been installed:

In `settings.py`:

```python
INSTALLED_APPS = [
    'app_name.apps.AppNameConfig',	#(function in apps.py of app directory)
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

`python manage.py makemigrations`:
tells Django that there are changes to the model and for them to be stored as a migration (which are stored as files)

`python manage.py sqlmigrate <app-name> <migration-number>`
show you the SQL commands that will be migrated

`python manage.py check`
check for problems

`python manage.py migrate`
make the migration

Django applies all migration that haven't been applied (tracked by a table in database called `django_migrations`)

Django has a database-access API
We can access it via the command line: `python manage.py shell`

`__str__()` method in the models for representation

Django Admin

- `python manage.py createsuperuser`

- Go to `/admin/` page

- To make our models modifiable:

  - Go to `admin.py` in your app directory

  - ```python
    from django.contrib import admin
    from .models import ModelName
    
    admin.site.register(ModelName)
    ```

More in-depth views

You can also have keyword arguments in the view:

```python
# views.py
def view(request, keyword_argument1, ...):
    ...
```

```python
# urls.py
urlpatterns = [
    ...
    path('<type:keyword_argument1>/', views.view, name="view")
    path('<type:keyword_argument1>/page/', views...)
    ...
]
```

The call to the view will result as this `view(request=<HttpRequest object>, keyword_argument1=value)`

Views can do anything, but it must return a `HttpResponse`, `Http404`.

# Templates

Create a `templates` directory in your app directory.

`TEMPLATES` setting: by default, `APP_DIRS` is True so `DjangoTemplates` backend will look for templates subdirectory in each of the `INSTALLED_APPS`.

Inside the `templates` director, create a directory with the same name as your app (name spacing to avoid conflict with other apps) and put your HTML template inside.

Example of template:

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

Instead of hardcoding the url, you could use the `name` argument defined in `path()`. You can do `<a href="{% url 'name' argument %}">...` and you can change the url in `path()` without needing to change the template.

If different apps have the same name for the URL? You can namespace the `urls.py` file:

```python
app_name = 'appname'
urlpatterns = [...]
```

`<a href="{% url 'appname:name' argument %}">...`

How to use the template in the view:
You would pass in a context dictionary that maps the template variables to Python objects.

1. Loader
	```python
   from django.http import HttpResponse
   from django.template import loader
   
   from .models import Question
   
   def index(request):
       ...
       loader.get_template('template_file') #relative from the `templates` directory
       
       context = {
           'template_variable' : value
       }
       
       return HttpResponse(template.render(context, request))
   ```
   
2. render()

   ```python
   from django.shortcuts import render
   from .models import Question
   
   def index(request):
       ...
       context = {
           ...
       }
       return render(request, 'template_file', context)
       
   ```



404 error

```python
from django.shortcuts import render
from django.http import Http404

def view(request, argument):
    try:
        variable = Model.objects.get(value=argument)
    except Model.DoesNotExist:
        raise Http404("model with argument does not exist")
    return render(...)
```

OR a shorthand way to maintain loose coupling

```python
from django.shortcuts import get_object_or_404, render

def view(request, argument):
    variable = get_object_or_404(Model, variable=argument, ...)
```

`get_object_or_404()` takes an arbitrary number of keyword arguments and passes them to the `get()` function of the model's manager.

`get_list_or_404()` passes into the `filter()` function instead and raises the error if the list is empty

# Forms

How do HTML form work?
`action` attribute: defines the URL where the form data should be sent to
`method` attribute: defines which HTTP method to send the data with
Each input field of the form have an associated name and value.
This will send the data via a URL defined by `action` and the view from there can do the processing.

```html
<form action="{% url 'polls:vote' question.id %}" method="post">
    
{% csrf_token %}
    
<fieldset>
    
    <legend><h1>{{ question.question_text }}</h1></legend>
    
    {% if error_message %}
    	<p><strong>{{ error_message }}</strong></p>
    {% endif %}
    
    {% for choice in question.choice_set.all %}
    
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    
    {% endfor %}
    
</fieldset>
    
<input type="submit" value="Vote">
</form>
```

- `{% csrf_token %}` for internal urls
- `forloop.counter` the number of times the `for` tag has gone through the loop

The view:

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

- `request.POST` and `request.GET` are dictionary-like objects where we can access the data/parameters with by key names. (Ex. `request.POST['name']`)
- `KeyError` for error messages
- `HttpResponseRedirect` takes an URL and redirects to it rather than returning a response
  (Always return a `HttpResponseRedirect` after dealing with POST data)
- `reverse()` helps us avoid hardcoding URLs, it is given the name of the view and the parameters

# Generic Views

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
- We are changing the pattern string variable to `pk` because generic view expects the primary key value captured to be called `pk`.

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    # display a list of objects
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    # display a detail page for a type of object
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

- Now it is a class
- Each view must know the `model` 
- by default the `template_name` is called `<app name>/<model name>_detail.html` but we can override it
- `context_object_name`: overriding the attribute to specify that we want the context variable to a different name

https://docs.djangoproject.com/en/3.2/intro/tutorial05/

# Static files

`django.contrib.staticfiles` collects static files from each of your apps (and other locations) into a single location for organization

`STATICFILES_FINDERS` contains a list of finders that know how to get your static files
For example, `AppDirectoriesFinder` look for a static subdirectory in each of the `INSTALLED_APPS`.

Create a `static` directory in your app directory and namespace it just like templates.

Put your static files.

At the top of your template files, put

```html
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'static file' %}"
```

