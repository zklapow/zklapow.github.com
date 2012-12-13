---
layout: post
title: "Pythonic API's Using Metaclasses"
description: ""
category: python
tags: [python, object-oriented, programming]
---
{% include JB/setup %}

The other day I saw an awesome post about using pythons magic methods in order to [improve API's and make them more pythonic](http://ozkatz.github.com/better-python-apis.html). After applying some of his suggestions to his current project I started thinking about ways to make python apis *even more* pythonic.

<!-- more start -->

The Problem
===========

My problem was that I had database models but the api for using them was not as clean as I would have liked.
To illustrate this situation, lets take the simple example of a user class, with a list standing in for our database and storing all of the users that are created: 
**Edit:** There seemed to be some confusion here. The following example is meant to illustrate the use of a metaclass but is not a real world example. The list used in the class is meant to approximate a database connection, in real life the `User` class would probably have some `get_all()` method over which you would iterate.

{% highlight python %}
class User(object):
    users = []
    def __init__(self, name, email):
        self.name = name
        self.email = email

        User.users.append(self)

    def __repr__(self):
        return "User('%s', '%s')" % (self.name, self.email)
{% endhighlight %}

Great so now we can instantiate some users and grab them all via the `User.users` list, like so:
{% highlight python %}

>>> sally = User('sally', 'sally@gmail.com')
>>> bob = User('bob', 'bob@gmail.com')
>>> for user in User.users:
...     print(user)
User('sally', 'sally@gmail.com')
User('bob', 'bob@gmail.com')]

{% endhighlight %}

This is all well and good, however, what what I really wanted the ability to do this:

{% highlight python %}
>>> for user in User:
...     print(user)
{% endhighlight %}

At first the solution to this problem seems simple simple define a classmethod `__iter__` like so:
{% highlight python %}
    @classmethod
    def __iter__(cls):
        for u in cls.users:
            yield u
{% endhighlight %}

Of Types and Classes
====================

Why doesn't simply defining `__iter__` as a classmethod work? The simple explanation is that it is never being called. What we have to remeber here is that everything in python is an object *including classes*. So while sally and bob are instances of `User`, `User` itself is in fact an intance of `type`.

{% highlight python %}
>>> type(sally)
User
>>> type(bob)
User
>>> type(User)
builtins.type
{% endhighlight %}

In fact, because `User` is simply an instance of type we don't even need to declare the class at all, instead we could have used the `type` constructor itself to generate the class dynamically:

{% highlight python %}
>>> User = type('User', (), {...})
{% endhighlight %}

> If you are interested in dynamically generating classes you should check out the [standard library docs on `type`](http://docs.python.org/3.1/library/functions.html#type).

Of course `builtins.type` has no method `__iter__` and thus our loop throws an exception. So how do we make our class iterable?

Enter Metaclasses
=================

This problem can be solved by changing the type of the class `User` to something other than `builtins.type`. This can be achieved using what is known as a metaclass.(It should be noted that the following code will only work for python 3 as the way metaclasses are handled has been changed)

### What is a Metaclass?

As I said before everything in python is an object and every object has a class, including builtin types like `str` and `int`:
{% highlight python %}
>>> age = 10
>>> age.__class__
builtins.int
>>> string = 'test'
>>> string.__class__
builtins.str
{% endhighlight %}

And as we showed before the class of a class is `type`:
{% highlight python %}
>>> age.__class__.__class__
builtins.type
>>> string.__class__.__class__
builtins.type
{% endhighlight %}

So `type` can be thought of as the universal metaclass, it is the class which creates all other classes. Thus, a metaclass can be thought of as simply a class which creates other classes. The intances of a metaclass are classes, unlike normal classes whose instances are simply normal objects.

### So What?

The problem we were having earlier was that when we attempted to iterate over our class we were in fact attempting to iterate over an inatnce of `type`. We now know that `type` is simply the metaclass of `User`. Luckily, python lets us define our own metaclasses. We can now rewrite the `User` class like so:
{% highlight python %}
class IterMeta(type):
    def __init__(cls, name, bases, dict):
        print(cls)
        return type.__init__(cls, name, bases, dict)

    def __iter__(cls):
        for i in cls.items:
            yield i

class User(object, metaclass=IterMeta):
    items = []
    def __init__(self, name, email):
        self.name = name
        self.email = email

        User.items.append(self)

    def __repr__(self):
        return "User('%s', '%s')" % (self.name, self.email)
{% endhighlight %}

I have indcluded the print in the metaclasses `__init__` to demonstrate that the first argument that gets passed to all metaclass methods(`cls` here) is in fact jus the instance of the metaclass, which is our class `User`.

Using this new version of the `User` class our earlier example now works:
{% highlight python %}
>>> sally = User('sally', 'sally@gmail.com')
>>> bob = User('bob', 'bob@gmail.com')
>>> for u in User:
...     print(u)
...
User('sally', 'sally@gmail.com')
User('bob', 'bob@gmail.com')
{% endhighlight %}

### A Note About Python 2

All of the same things can be achieved in python 2, but the syntax for doign so is slightly different. Simply set the `__metaclass__` class attribute to set the metaclass in python 2. This even has the slight added advantage of allowing the metaclass to be defined within the class it is metaclassing.

Conclusion
==========

Hopefully all this talk of classes of classes and types of types hasn't twisted your brain up to much. I have demonstrated only one example of the usefullness of metaclasses here with the `__iter__` method. Metaclasses can be extremely powerful tools in customizing the behavior of class objects in python and can make APIs significantly more intuitive if used properly! By changing the type of the class itself we enable our classes to become much mur useful than simple object creators. If you want to see a great example of these concepts in action, check our [Django's ORM](https://docs.djangoproject.com/en/dev/topics/db/models/), it makes heavy use of metaclasses in order to provide its intuitive API, and inspired me to write this post!

<!-- more end -->
