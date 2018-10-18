---
layout: post
title:  "Replacing Ruby class variables with class object instance variables"
date:   2018-10-18 11:00:00 +1100
categories: ruby
---
Class Variables in Ruby are class-hierarchy scoped rather than class-scoped, meaning that the class variables with same name, present in different classes that are in hierarchy, will be shared.

For example:

{% highlight ruby %}
class Parent
  @@val = 1
  
  def self.val
    @@val
  end
end

class Child < Parent
  @@val = 2
end

Child.val #=> 2
Parent.val #=> 2
{% endhighlight %}

Here Parent class has defined a class variable `@@val = 1`. Now in the inheritance hierarchy the class variable with the same name is assigned a new value by the child class. Since class variable are class-hierarchy scoped, `Parent.val` will give us result 2 because the class variable is shared between the parent and the child class, and the child class overwrites the value of `@@val` to 2.

Though class variable are useful in maintaining state in a class, because of it’s class-hierarchy scoped feature, class variables are accessible to lot of objects in an inheritance hierarchy, which can cause problems if we want to maintain state of certain subclasses separate from its parent. 

For example:

{% highlight ruby %}
class Bicycle
  @@total_count = 0

  def initialize
    @@total_count += 1		
  end

  def self.total_count
    @@total_count
  end
end

class MountainBike < Bicycle
end

class CityBike < Bicycle
end

MountainBike.new
MountainBike.new
MountainBike.new

MountainBike.total_count #=> 3

CityBike.total_count #=> 3
{% endhighlight %}

Here you can see that MountainBike and CityBike classes inherit from Bicycle class. The class variable defined in Bicycle class in now shared between all these classes in inheritance hierarchy.

Unlike instance variables, class variables are not created freshly for every class in the hierarchy, rather all the classes in the hierarchy share the same class variable with same name.

The problem with the above code is that the state of total_count of different types of bycicles are not maintained separately, as a result, when we created 3 new MountainBikes, the total_count of CityBike also increamented by 3 which is incorrect since we haven't initialized any CityBikes.

We can solve this issue by the use of instance variables associated with the class object and not with the object instance.

Every object in ruby can have instance variables and class is also and object of class 'Class'. So we can attach an instance variable to the class object as well.

The reason for attacing an instance variable to the class object is that, instance variables are fresh for every class, even in the inheritance hierarchy. So since the instance variables are unique for each class and since they are attached to the class object, this helps us to maintain the class level state separately for each class in the inheritance hierarchy.

For example, we can change the above code to:

{% highlight ruby %}
class Bicycle
  def self.total_count
    @total_count ||= 0
  end

  def self.total_count=(n)
    @total_count = n
  end

  def initialize
    self.class.total_count += 1
  end
end

class MountainBike < Bicycle
end

class CityBike < Bicycle
end

MountainBike.new
MountainBike.new
MountainBike.new

MountainBike.total_count #=> 3
CityBike.total_count #=> 0
{% endhighlight %}

To fully understand how the above code maintains the state of total_count separately for each class, you have to know about metaclasses.

[You can learn more about self, scope and metaclasses in this article.](https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)

Basically metaclasses are classes that each ruby object has, which store the methods and properties for that object.

In the above code the class method total_count is being added to the metaclass of self which in the Bicycle class is the  class object Bicycle. When the child classes like MountainBike and CityBike inherits from Bicycle, these two classes inherits the total_count methods and in the scope of these classes self becomes either the MoutainBike or CityBike, in which case the total_count method will be added to the appropritate metaclasses, which are separate from other metacalsses. This is how the total_count state of each class will be maintained separately.

In the above code the total_count of MountainBike is 3 because we have create 3 new MountainBikes.

And since now the CityBike maintains its own total_count, the CityBikes total_count is 0, because we haven't created any new CityBikes yet.


<!-- Post comments -->
<div class="comments">
  {% include disqus.html %}
</div>
