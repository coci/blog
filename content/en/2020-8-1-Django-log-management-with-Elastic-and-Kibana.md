---
utid: 20200801101142
date: 2020-08-01 10:11:42
title: Django log management with Elastic and Kibana
_index: dajngo-log-efk
categories:
  -
tags:
  -
---

log management play the most crucial role in any system. logs can tell us where bugs happened and also can determine useful information about system.

In this guide, I’m going to teach you how to create log management with EFK stack and then connect it to your python application. If you don’t have enough information about EFK, it stands for Elasticsearch, Filebeat, and Kibana that I'll explain them in more detail.

### Elasticsearch :

Elasticsearch is an open source, full-text search and analysis engine.

### Filebeat :

Filebeat is an open source tool that collect data and ship them to Elasticsearch.

### Kibana :

Kibana is a visualization layer that works on top of Elasticsearch, providing users with the ability to analyze and visualize the data.

## Objective :

this is a glimpse overview about what's going on this guide that might help : the application will produce logs and store them in a specified path and then Filebeat ship that logs to our Elasticsearch, and with Kibana, which is a graphical interface, we can manage our logs.

Well, now you know about what we are going to do and why we choose them. Before we dive into the tutorial, I want to say that there are tons of tutorials on the internet about EFK. Still, all of them are base on regular installation, You might encounter some trouble to configure and connect them, so I decided to create a guide on docker. Let's get start.

## Create Django project and configuration logging system :

for the first step we are going to create simple Django application , follow steps :

```bash
$   mkdir django_logs
$   cd /django_logs
```

and then install Django and create project :

```bash
$  pip install django
$  django-admin startproject django_logs .
$  python3 manage.py startapp test_log
```

now modify /django_logs/settings.py :

```python
....

INSTALLED_APPS = [
	'django.contrib.admin',
	'django.contrib.auth',
	'django.contrib.contenttypes',
	'django.contrib.sessions',
	'django.contrib.messages',
	'django.contrib.staticfiles',
	'test_log.apps.TestLogConfig',
]
```

and it's time adding to logging package into our project , so add these in /django_logs/settings.py :

```python
LOGGING = {
	'version': 1,
	'disable_existing_loggers': False,
	'formatters': {
		'simple': {
			'format': '[%(asctime)s] %(levelname)s | %(funcName)s | %(name)s | %(message)s',
			'datefmt': '%Y-%m-%d %H:%M:%S',
		},
	},
	'handlers': {
		'logger': {
			'level': 'DEBUG',
			'class': 'logging.handlers.RotatingFileHandler',
			'filename': BASE_DIR + '/logs/test.log',
			'formatter': 'simple',
		}
	},
	'loggers': {
		'signal': {
			'handlers': ['logger'],
			'level': 'DEBUG',
		}
	}
}
```

just notice that the line :

```python
....
'filename': BASE_DIR + '/logs/test.log' 
....
```

Tells Django where to store logs; you can modify this, but remember it because we tell to Filebeat where our data will store.

now let's create view that might manipulate log , so add these line to test_log/views.py :

```python
from django.http import HttpResponse
from django.shortcuts import render
from .task import add

import logging

def index(request):
	logger = logging.getLogger("loggers")
	message = {
		'message' : "user visits index()"
	}
	logger.info(message)
	return HttpResponse("hello")
```

If you review our code, you will notice that every time user visits our view, Django will create a log with 'message': "user visits index()."

please visit our route once and if you go to "/logs/test.log" and see the content of test.log files you must encounter with something like this :

```bash
[2020-07-25 04:39:33] INFO | index | t.views | {'message': 'user visits index()'}
```

ok we've done with Django . let's dive right into creating EFK stack .

## Elastic , Filebeat and Kibana :

As well I mentioned earlier, we use docker to cut off troubling with installation and configurations.

In other words, to manage three different tools ( Elastic, Filebeat, Kibana ), we must use docker-compose.

now create docker-compose.yml in your project directory and add these lines :

```python
version: '2'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: elasticsearch
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      discovery.type: single-node
    networks:
      - efk

  kibana:
    image: docker.elastic.co/kibana/kibana:7.8.0
    ports:
      - 5601:5601
    networks:
      - efk
    links:
      - elasticsearch
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:7.8.0
    volumes:
      - /path-to-filebeat.yml/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /path-to-logs-directory/logs:/usr/share/filebeat/logs
    environment:
      ELASTICSEARCH_URL: <http://elasticsearch:9200>
    networks:
      - efk
    links:
      - kibana
      - elasticsearch
    depends_on:
      - elasticsearch
networks:
  efk:
    driver: bridge
```

keep in mind that just modify these lines : 

```python
- /path-to-filebeat.yml/filebeat.yml: ....
- /path-to-logs-directory/logs: ....
```

we have our docker-compose configuration, now as you see we need to create filebeat.yml configuration, well, create filebeat.yml file in your project directory and add these lines :

```python
filebeat.inputs:

- type: log
  enabled: true
  paths:
    - /usr/share/filebeat/logs/*.log
    
  multiline.pattern: '^\\['
  multiline.negate: true
  multiline.match: after

output.elasticsearch:
  hosts: ["<http://elasticsearch:9200>"]
```

well, it's time to fire up our work, open terminal and enter :

```python
$  docker-compose up -d
```

( it just takes minutes )

## Kibana dashboard :

If everything is done correctly , you must see this dashboard after you open [localhost:5601](http://localhost:5601) in browser:

![block](/blog/images/blockchain.jpg)

now navigate to Kibana→discover :

![gitea](/blog/images/2.jpg)

then go to index patterns → create index pattern

![gitea](/blog/images/3.jpg)

and then :

![gitea](/blog/images/4.jpg)

now everything is done and you can see logs , just navigate to menu → observability → logs .