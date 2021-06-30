---
layout: post
title: Django and Rails for APIs
author: Kurt
postFooter: <a href="rubyonrails.org">djangoproject.com</a> and <a href="rubyonrails.org">rubyonrails.org</a>
---

# Goal of the API

The purpose is to build out a list of related managers and engineers. A manager can have many subordinate engineers, but an engineer can work under only one manager. The JSON output will look like this:

_managers_

```json
[
  {
    "id": 1,
    "name": "Alice",
    "engineers": [
      {
        "id": 1,
        "name": "Bob"
      },
      {
        "id": 2,
        "name": "Sarah"
      },
      {
        "id": 3,
        "name": "Pratik"
      }
    ]
  }
]
```

_engineers_

```json
[
  {
    "id": 1,
    "name": "Bob",
    "manager": {
      "id": 1,
      "name": "Alice"
    }
  },
  {
    "id": 2,
    "name": "Sarah",
    "manager": {
      "id": 1,
      "name": "Alice"
    }
  },
  {
    "id": 3,
    "name": "Pratik",
    "manager": {
      "id": 1,
      "name": "Alice"
    }
  }
]
```

# Django API Tutorial

## Main setup

```sh
cd code
mkdir screencasts
cd screencasts
mkdir dj_company_api
cd dj_company_api
pipenv install django django-extensions djangorestframework
pipenv shell
django-admin startproject company_api .
django-admin startapp managers
django-admin startapp engineers
```

In _settings.py_

```python
INSTALLED_APPS = [
    ...
    'django_extensions',
    'rest_framework',
    'managers',
    'engineers',
]
```

In main _urls.py_

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework import routers
from managers.views import ManagerViewSet
from engineers.views import EngineerViewSet


router = routers.DefaultRouter()
router.register('managers', ManagerViewSet)
router.register('engineers', EngineerViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include(router.urls))
]
```

## Managers app

### models.py

```python
from django.db import models


class Manager(models.Model):
    name = models.CharField(max_length=25)
```

### serializers.py

```python
from rest_framework import serializers
from .models import Manager


class ManagerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Manager
        fields = ['id', 'name', 'engineer_set']
        # depth = 1
```

### views.py

```python
from rest_framework import viewsets
from .models import Manager
from .serializers import ManagerSerializer


class ManagerViewSet(viewsets.ModelViewSet):
    queryset = Manager.objects.all()
    serializer_class = ManagerSerializer
```

## Engineers app

### models.py

```python
from django.db import models
from managers.models import Manager


class Engineer(models.Model):
    name = models.CharField(max_length=25)
    manager = models.ForeignKey(Manager, on_delete=models.CASCADE)
```

### serializers.py

```python
from rest_framework import serializers
from .models import Engineer


class EngineerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Engineer
        fields = '__all__'
        # depth = 1
```

### views.py

```python
from rest_framework import viewsets
from .models import Engineer
from .serializers import EngineerSerializer


class EngineerViewSet(viewsets.ModelViewSet):
    queryset = Engineer.objects.all()
    serializer_class = EngineerSerializer
```

## Run db commands

```sh
python manage.py makemigrations # both, or just managers or engineers
python manage.py migrate
```

## Create sample data in Django shell

```sh
python manage.py shell_plus
```

```python
Manager.objects.create(name='Alice')
Engineer.objects.create(name='Bob', manager=Manager.objects.get(pk=1))
Engineer.objects.create(name='Sarah', manager=Manager.objects.get(pk=1))
Engineer.objects.create(name='Pratik', manager=Manager.objects.get(pk=1))
exit()
```

## Run server

```sh
python manage.py runserver
```

## Postman: show data

1. Open Postman and run on [http://localhost:8000](http://localhost:8000).
2. Add `depth = 1` to both serializers.
3. Show Postman requests again.
4. Conclude tutorial.

# Rails API Tutorial

## Main setup

```sh
cd code/screencasts
rails new --api rails_company_api
rails g scaffold manager name
rails g scaffold engineer name manager:references
```

## Update models/manager.rb for ORM

```ruby
has_many :engineers
```

## Run db command

```sh
rails db:migrate
```

## Create sample data in Rails console

```sh
rails c
```

```ruby
Manager.create(name: 'Alice')
Engineer.create(name: 'Bob', manager_id: 1)
Engineer.create(name: 'Sarah', manager_id: 1)
Engineer.create(name: 'Pratik', manager_id: 1)
exit
```

## Run server

```sh
rails s
```

## Postman: show data

1. Open Postman and run on [http://localhost:3000](http://localhost:3000).
2. In the JSON controllers (on list and detail actions), add:

```ruby
include: :manager
include: :engineers
```

3. Show Postman requests again.
4. Conclude tutorial.
