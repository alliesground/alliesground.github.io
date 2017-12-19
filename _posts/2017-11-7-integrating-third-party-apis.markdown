---
layout: post
title:  "Integrating 3rd party APIs using Wrapper objects"
date:   2017-11-07 10:12:10 +1100
categories: refactoring
---
When integrating 3rd party APIs in our application we might be tempted to directly call the API methods
from within our application, and since those calls can be made from several places in our application, it can lead to API method calls being scattered all over our application.

In future if the API’s interfaces changes, it will have a side effect to our application, and the worst part is that in order to make changes we have to look for the method calls all over our application, which can create maintenance issues caused by applications that cannot embrace change.

Techniques like Single Resposibility, DRY (Dont repeat yourself), dependency isolation and many others can help in writing codes that embrace changes and reduces maintainence issues.

In our current case of integrating 3rd party APIs, our application depends on the external services over which we do not have any control and its impossible to avoid the side effects when the APIs changes. 

In such cases, even though we might not be able to totally remove the side effects, we can definitely reduce it by isolating the dependencies into an object over which we have control.

By isolating the dependencies into an object, we are basically creating a boundary between our application and the external services, so now whenever our application needs to communicate with the external services it has to do so through the boundary object which will be our wrapper class and all the responsibility of interacting with the external services will be handled by this class.

Now, in future if there is any changes made to the external APIs we can simply go to this single wrapper class and adjust the changes. There is no need to dig into our application code to make the changes.

#### Let’s see how we can accomplish this.

I’ll be using github’s client library to access its services.  

{% highlight ruby %}
class RepositoriesController < ApplicationController
  def show
    github_wrapper = GithubWrapper.new(access_token, ENV[‘TEST_USER_ACCESS_TOKEN’])
    @repository = github_wrapper.repo(current_user.repository.full_name)
  end

  def create
    repository = current_user.repositories.build(repository_full_name: params[:repository][:repository_full_name])

    if repository.save
      github_wrapper = GithubWrapper.new(access_token: ENV[‘TEST_USER_ACCESS_TOKEN’])
      github_wrapper.create_repo(name:params[:name])
    end
end
{% endhighlight %}

Here the ```RepositoriesController``` ```create``` action is responsible for creating a repository in our application database.

If creating a repository in our database is successful, it calls Github’s API through ```GithubWrapper``` class to create a corresponding repository in Github.

```TEST_USER_GITHUB_TOKEN``` is a token received for every user after successful authentication with github. Here I'm using the token provided by Github for testing purposes.

#### GithubWrapper class:
{% highlight ruby %}
class GithubWrapper
  def initialize(access_token:)
    @client ||= Octokit::Client.new(
      access_token: access_token
    )
  end

  def create_repo(name:)
    @client.create_repository(name)
  end

  def repo(repo_full_name:)
    @client.repository(repo_full_name)
  end
end
{% endhighlight %}

The ```GithubWapper``` class wraps the Octokit::Clinet library(Github’s client library) and interacts with github for our application.

## Conclusion
By wrapping the Github's API in our own GithubWrapper class we are isolating the external API dependencies.

The benefit of this isolation is that it makes our application easy to maintain by embracing change.

For example: In the future if Github makes some changes to its interfaces, we will have to go to only one place i.e. the ```GithubWrapper``` class in order to make changes, rather than having to find and change Github's interfaces all over our application.
