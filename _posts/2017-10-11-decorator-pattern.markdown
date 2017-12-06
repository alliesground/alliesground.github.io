---
layout: post
title:  "Extending Objects with Decorators"
date:   2017-10-11 00:45:55 +1100
categories: design-patterns
---
Decorator patterns satisfies the ‘O’ of SOLID principles, which stands for "Open for extension but closed for modification".

Decorator patterns allows us to extend the functionality of an object by encapsulation the display logic assocaited with a Domain model object(the object being extended).

For example let’s take a ``User`` ActiveRecord model with corresponding users table with ‘first_name’ and ‘last_name’ as its attributes,
and a method ``full_name``.

{% highlight ruby %}
class User < ActiveRecord::Base
  def full_name
    “#{first_name} #{last_name}”
  end
end
{% endhighlight %}

The full_name method above combines the first_name and last_name to return a full name of the user. This is entirely a display logic.

The User model is a domain object and it should only be concerned with business logic associated with user domain and not the display logic.

Keeping in mind the Single Responsibility Principle, we should extract this display logic into a separate object(Decorator object).

Decorator objects are often used to extend a single domain object, encapsulating the display logic for it.

In our example ``User`` is a domain object, so we will be writing a decorator object specifically for that object to encapsulate the display logic contained in its ``full_name`` method.

## Implementing Decorator Object

Let’s create a Decorator object.


{% highlight ruby %}
class UserDecorator
  attr_reader :user

  def initialize(user)
    @user = user    
  end

  def full_name
    “#{user.first_name} #{user.last_name}”
  end
end
{% endhighlight %}

Here we are creating a Decorator object that wraps the User object. We initialise this object by passing a user’s instance.
We’ve also moved the full_name method from User object to this Decorator object.

The ``UsersController`` looks something like:

{% highlight ruby %}
class UserController < ApplicationController
  def show
    @user = User.find(params[:id])
  end
end
{% endhighlight %}

So now in our ``#show`` view *users/show.html.haml* we can call:

{% highlight ruby %}
%b= UserDecorator.new(@user).full_name
{% endhighlight %}

This works fine for simple views, where just accessing the Decorator methods does the job, but views can become complex.

In complex views there might be cases where we need to access the methods in the model along with methods in the decorator.

In order to accomplish this we have to pass the ``UserDecorator`` object to the view from the controller ``#show`` action instead of ``@user`` object, and have that Decorator object delegate the method calls to the ``User`` object.

#### Let's implement that:

{% highlight ruby %}
class UsersController < ApplicationController
  def show
    @user = UserDecorator.new(User.find(params[:id]))
  end
end
{% endhighlight %}

Here we are passing ``UserDecorator`` object to the view, instead of the ``User`` object.

Our decorator object needs to respond to the calls for the methods that it encapsulates as well as
for the methods of the User model.

To accomplish this we use method delegation.

In our Decorator object we extend the Ruby forwardable module.

{% highlight ruby %}
class UserDecorator
  extend Forwardable
  def delegators :user, first_name
  
  # code commented out for brevity
end
{% endhighlight %}

The ``def delegators :user, first_name`` delegates the call to first_name on Decorator object to User object.

After delegating the methods we can now, in our view call the methods either in the Decorator object or in the User model object.

*users/show.html.haml*
{% highlight ruby %}
  %p= @user.first_name #calling first_name attribute of User model
  %p= @user.full_name #calling full_name method of the Decorator object
{% endhighlight %}

## Conclusion

Decorator patterns are useful when we want to grab data from the database and display it in a
different way in the view. It encapsulates the view logic associated with a domain object and separates
the view logic from domain logic thereby maintaining single responsibility principle.

You don’t have to roll your own Decorator objects like I did in this article, instead you can use <a href="https://github.com/drapergem/draper" target="_blank">draper</a> gem, which makes it easy to create Decorator objects.
