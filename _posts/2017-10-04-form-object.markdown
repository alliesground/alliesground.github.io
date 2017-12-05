---
layout: post
title:  "Removing clutter from your controller with Form Object"
date:   2017-10-04 00:45:55 +1100
categories: design patterns
---
Form object is a simple ruby object that can help reduce the clutter in your controllers when creating a form that deals with many models.

Alternative to form object is nested attributes, which I believe, because of it's implicitness is not an ideal solution.

A registration form for example, can have fields for user_name, email and address, and let's say you want to save the user_name in the `users` table and address in `address` table.

With this requirements our controller will look something like:

{% highlight ruby %}
class UsersController < ApplicationController
  def create
    @user = User.new(params[:user])
    @user.address.build(params[:address])

    if @user.save
      UserMailer.send_registration_email
      UserMailer.send_email_notification_to_admin

      flash[‘success’] = “Successfully created a project”
      redirect_to root_path
    else
      flash[:error] = “could not create a project”
      render :new
     end
  end
end
{% endhighlight %}

This ``UsersController`` ``create`` action is responsible for doing multiple things. It is initialising and saving the user, initialising the address of that user, sending a registration confirmation email and sending another email to the admin to notify him/her about the new registration.

Clearly the ``create`` action is getting cluttered with responsibilities that is not necessarily it’s.

Usually the convention in rails is that, a controller is responsible for interacting with only one corresponding model.

For example the above ``UsersController`` ``create`` action should only be responsible for creating a new User object, and not getting itself cluttered by building new addresses or sending emails.

Form objects can help us remove this mess by encapsulating all the different behaviours, leaving the controller clean and conventional.

Lets create a form object.

{% highlight ruby %}
class RegistrationForm
  include ActiveModel::Model

  attr_accessor :name
  attr_accessor :email
  attr_accessor :address
end
{% endhighlight %}

So this is our form object which is simply a plain old ruby object with some additional interfaces provided by ``ActiveModel::Model``.

Our ``RegistrationFrom`` object has ``include ActiveModel::Model``. This is a module provided by rails. Inclusion of this module gives our ``RegistrationForm`` object interfaces for validation which we can use to validate the form attributes at the application level. This also give us the ability to initialise our ``RegistrationForm`` object with a hash of attributes similar to what Active Record does: ``@registration = RegistrationFrom.new(params[:registration])``

The accessor ``attr_accessor :name`` allows us to use the attribute names in the form view like
{% highlight ruby %}
form_for @registration_form do |f|
  f.input :name
end
{% endhighlight %}

The inclusion of ``ActiveModel::Model`` module also gives us a method called ``valid?`` which executes the validation process for the attributes being validated.  

Since this is a form object I attach the word form to the class names, just my convention.

After creating a form object lets create a corresponding controller, which will replace the above ``UsersController``.

{% highlight ruby %}
class ReigstrationFormsController < ApplicationController
  def create
    @registration_form = RegistrationForm.new(registration_params)
    if @registration_form.save
      # redirect with flash
    else
      render :new
    end
  end

  private
  def registration_params
    #permitting the params
    params.require(params[:registration_form]).permit(:name, :email, :address)
  end
end
{% endhighlight %}

This new controller looks simple without any clutter and it is following the convention of just saving the corresponding object.

Let's look at what's happening here.

First up we are initialising the ``RegistrationForm`` object by passing params hash which is possible because of ``ActiveModel::Model`` included in our ``RegistrationForm`` object.

Next ``save`` method is being called on the ``@registration_form`` instance.

Lets implement the ``save`` method in our RegistrationForm object

{% highlight ruby %}
class RegistrationForm
  include ActiveModel::Model

  attr_accessor :name
  attr_accessor :email
  attr_accessor :address

  validates_presence_of :email

  def save
    return false unless valid?
    user = User.create(name: name, email: email)
    user.addresses.create(address: address)
    send_registration_email
    notify_admin
  end

  private

  def send_registration_email
    UserMailer.send_registration_email
  end

  def notify admin
    UserMailer.notify_admin
  end
end
{% endhighlight %}

Here the ``save`` method first executes the validation process with ``valid?`` method provided by ``ActiveModel::Model``. This validate the attributes, similar to what Active Record validation does. If the attributes are valid, other models are created and actions are performed.


# Conclusion
Overall the ``RegistrationForm`` object acts as a orchestrator and directs multiple actions just like an orchestrator, which is its single responsibility i.e to orchestrate. What makes form objects better than nested attributes is its explicitness, which makes the code easy to follow.
