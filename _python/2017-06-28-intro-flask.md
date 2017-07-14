---
layout: 
title: "1 - Introduction to Flask"
excerpt: "Installation and other starting information."
tags:
    - python
    - flask
---

What's up everyone. For this tutorial series we will be focusing on backend python development using Flask. Flask is a "micro webdevelopment framework for Python". The micro just means that Flask likes to keep it simple. 

We are going to jump right into it, but I will warn that these tutorials are not for the python beginner (although I'm just that beginners who have experience with code will be able to figure it out). I suggest having some knowledge of python before starting this.

## Installation
When working with python projects, I think it is very important for everyone to be organized. This means keeping everything in a virtual environment. Virtual environments pretty much just puts a container around your app. This way you can download all sorts of dependencies that will only download and effect your application. Say you have two very large projects that use different versions of a particular dependency. This way we can keep things seperate and organized.

Open a shell:
```bash
# OSX
$ sudo pip install virtualenv
# Linux
$ sudo apt-get install python-virtualenv
```

Now that we have our virtual environment dependency downloaded, we can actually create a container for our first project:
```bash
$ mkdir project1
$ cd project1
$ virtualenv venv
New python executable in venv/bin/python
Installing setuptools, pip, wheel............done.
```
To move into using your virtual environment:
```bash
$ . venv/bin/activate
```

If you would like to deactivate your venv:
```bash
$ deactivate
```

For now let's keep ourselves in our virtual environment. You should be able to see (venv) on your command line denoting that you are currently in your virtual environment. 

To download Flask (finally):
```bash
$ pip install Flask
```

## Quickstart

The following converters exist:
string
int
float
path
any
uuid

HTTP Methods
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```
possible methods:
GET
HEAD
POST
PUT
DELETE
OPTIONS
