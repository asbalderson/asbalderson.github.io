---
layout: post
title: Another Flask Project Structure
---

## Background

I recently had the pleasure of learning Flask while I created an API for taking 
multiple choice tests online.  I'll write another blog post about that, but 
today I want to talk about the structure of Flask projects.  One which promotes 
more flexibility, readability, and expandability.

## The Problem

If you're anything like me, when you learned to use Flask, you Googled around 
to see what it was about.  Most places gave a basic project, which laied out 
something like this:

{% highlight bash %}
$ tree my_flask_project
my_flask_project
├── app.py
├── config.py
├── models.py
├── requirements.txt
├── static/
├── templates/
└── views.py
{% endhighlight %}

This is great for a simple example, but once you start to write a real 
application this starts to break down.  Views slowly accumulates more 
and more routes, models collects more and more classes and suddenly these files 
are huge and hard to find anything the developer is looking for.

The other big thing which bothers me about these examples is that they always
start the app.py, or the init with these lines of code:

{% highlight python %}
from flask import Flask
app = Flask(__name__)
{% endhighlight %}

Then the examples go around importing app and adding routes to it.  It works 
fine but what if you want to use some code outside of the app, or more applicably, 
want to run unit testing against routes without importing the whole application?

Also, who likes being put in a box.  Where is the freedom of being able to put 
things where they go logically to you, the developer?  There always has to be
a better way. A more free and flexible way to set up the project that is still 
clean and logical.

## A Solution

Believe it or not, flask actually has a some tools for making the project more
module.  The biggest of these is the Blueprint. A developer can assign routes 
to a Blueprint and then add the routes to the Flask application when they feel 
it is best to do so.  They can also be assigned to a testing application for 
unit tests! 

These let you break the view.py file down nicely into something like this:
{% highlight bash %}
$ tree my_flask_project
my_flask_project
├── app.py
├── config.py
├── models.py
├── requirements.txt
├── static/
├── templates/
├── views/
│   ├── index.py
│   └── forum.py
└── errors/
    ├── NotFound.py
    └── InternalServer.py
{% endhighlight %}
Now each page, route, grouping, error, or whatever logical grouping makes sense 
to you can be its own file.  Easy to find, easy to update, and easy to add.

Each file which has a route needs a Blueprint, these can be added at the top of
the file and used like a flask application:
{% highlight python %}
from flask import Blueprint
route_bp = Blueprint('route', __name__, template_folder='../templates)
{% endhighlight %}
Add a route to the blueprint like this:
{% highlight python %}
@route_bp.route('/route' methods=['GET'])
def get_route():
    #do the thing
{% endhighlight %}

It also needs to be added to the application.  Now the app doesnt need to be 
a global. so we can edit the app.py file from whatever the default was to 
something close to this:
{% highlight python %}
from flask import Flask
from .views.index import index_bp

def run():
    app = Flask(__name__)
    app.register_blueprint(index_bp)
    app.run()
    
if __name__ == '__main__':
    run()
{% endhighlight %}
It's still a bit over simplified, but it cleans things up a bit and adds some 
grouping.

In my most recent project, it also allowed me to group the flask errors and 
their routes into individual files.  Then import the errors into the routes and
raise them when it made sense to do so.  

After adding about 3 routes.  I got tired of importing the route, then adding 
it to the app one after the other.  After all, we program to make repetitive 
tasks simpler. I created a class which collected all the blueprints.  I added 
all the blueprints to the class, then tell allow the class to register all the 
blueprints.

Next we have models.  These are usually the class's and data structures used 
for collecting and organizing data.  This could also be the connection to the 
database if you are using SQLAlchemy or something similar.  There is, and never 
should be a time when these all need to be in one file, even in the "basic 
example" from above.  Break these out too, giving our final structure something 
like this:

{% highlight bash %}
$ tree my_flask_project
my_flask_project
├── app.py
├── config.py
├── models/
│   ├── users.py
│   └── posts.py
├── requirements.txt
├── static/
├── templates/
├── views/
│   ├── index.py
│   └── forum.py
└── errors/
    ├── NotFound.py
    └── InternalServer.py
{% endhighlight %}

## Conclusion

There are no rules for names of things in a project, or how things are 
organized.  Blueprints allow users to break the project apart into logical 
pieces which make debugging and testing easy.  [Check out](https://github.com/asbalderson/amttest) 
my most recent Flask 
project where I moved away from example structures to use my own structure 
that made sense to me, while staying organized and clean.  It also unit tests 
with more than 95% coverage.