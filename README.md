# First Routes & Controllers
This project demonstrates Rails routing.

## Learning Goals
* Be able to create routes in routes.rb
* Be able to read and understand Rails server error messages
* Know the three places that params come from
* Be able to nest query parameters
* Be able to write controller actions that read from and write to the database
* Know how and when to render errors

## First Routes
1. Setup database with ``` bundle exec rails db:create ```
2. Just for this assignment, add the line 
```config.action_controller.default_protect_from_forgery = false``` 
right after ```config.load_defaults 5.2``` in the config/application.rb file.
3. In config/routes.rb, generate first routes with:
```resources :users```
4. See first 8 routes with ```bundle exec rails routes```
5. Do the following:
* Comment out the resources :users line.
* Write out the eight routes using the route 'matching' syntax. For example: get 'users/:id', to: 'users#show', as: 'user'.
6. Run bundle exec rails routes again and ensure that the routes you've written match exactly the routes generated by the resources helper. NB: you'll probably be missing some names for some of the routes (they're listed in the left-most column); You can name your routes by adding an as option. get 'users/new', to: 'users#new', as: 'new_user'.

7. To give a little background, starting your Rails server creates an instance of a Router. This Router holds instances of Routes that are defined by your routes.rb. When a request comes in, the Router tries to match a Route based on the HTTP method and the url path (it does this with a regular expression, if you know what that is already). The first matched Route then instantiates an instance of the specified controller, and calls the specified action on it.

8. We have our initial routes now and have the endpoints necessary to manage a User resource. Notice though that our routes point to a UsersController, which we don't actually have yet. Nor do we have a User model. 

## First Controller

Each API endpoint creates/reads/updates/destroys (CRUD) a resource.

The router defines API endpoints (URLs), and records which controller and action to invoke for each one. Each API endpoint has a conventional meaning: create/read/update/destroy a resource. The controllers and their actions are the ones actually doing the CRUD ing.

I will create my first controller by navigating to ```app/controllers/``` and creating a new file called ```users_controller.rb``` 

``` 
class UsersController < ApplicationController
end
```
Note that controllers are always plural; a controller manages requests that pertain to a collection of resources. 
A resource is anything in your application that you will be CRUDing.

Controllers inherit from ApplicationController which is a controller itself, 
but one that never actually handles any requests directly. 
ApplicationController is where you'd put helper methods that you want 
to share across all controllers. 
ActionController::Base provides all the bells & whistles 
that Rails controllers have; it's like ActiveRecord::Base 
in that respect. All your controllers will inherit the features 
it provides since it is in the inheritance chain:

```UsersController < ApplicationController < ActionController::Base ```

## First Launch
We have our API endpoints setup and they map to a controller which we've created. 
How do we actually start taking requests?

```
$ rails server
=> Booting Puma
=> Rails 5.1.2 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.9.1 (ruby 2.3.1-p112), codename: Private Caller
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

Rails 5 ships with a web server called Puma.

As you can see, it loads your application in development mode 
(later we'll discuss the other two modes: production and testing), 
and is listening for requests at http://0.0.0.0:3000. 
The last part, :3000 specifies the port it is listening on. 
Rails defaults to port 3000 in development. 
The domain http://0.0.0.0 can be accessed from your browser as simply 
http://localhost.

In your browser, navigate to http://localhost:3000. 
Voila! A running Rails app with what will become a very familiar index page.

## First Request
Postman should already be installed, but if not, go ahead and install it.

Now let's try to get a list of all our users - our users index. 
This means we need to make a request that matches the HTTP verb and URI pattern that routes to 'UsersController#index'. 
If we bundle exec rails routes, we can see this is a GET request to '/users'. 
Go ahead and make that request with Postman. (Make sure to keep your rails server running in a terminal tab!)

Okay, that didn't work. Why?

The server log will be where you'll go to see what's going on in your application. 
All your puts and p statements in your application will also go to the server log. 
Always be looking at the server log; this is an essential debugging technique. 
We'll see what some of the most important information is in just a second. 
Here's what we see in the log:
```
Started GET "/users.html" for 127.0.0.1 at 2013-08-12 10:48:39 -0700

AbstractController::ActionNotFound - The action 'index' could not be found for UsersController:
```
Looks like a request came in; what's the error? It seems like it's complaining that 
we don't actually have an index action setup in our UsersController. 
Note that your application looked for an index action because the router specified 
that a GET request to /users maps to users#index, which is the Rails shorthand for UsersController#index.

Let's fix it. Add an empty index action to your UsersController:

```
class UsersController < ApplicationController
  def index
  end
end
```

Make the request in Postman again. It fails again, so look at the log:

```
Started GET "/users" for 127.0.0.1 at 2017-07-14 11:36:40 -0700
Processing by UsersController#index as */*
No template found for UsersController#index, rendering head :no_content
Completed 204 No Content in 178ms
```

NB: If you make a GET request through a browser at this point, you will get a 406 Not Acceptable status code.

This time, it's complaining that there's a missing template. Wait a minute; we never called render. 
Why is it trying to look for a template at all? 
Because in the absence of an explicit render statement, your controller will by default try to render a 
template with the same name as the controller action - 
in this case, it was looking for a template called index.html.erb in app/views/users.

We're not going to deal with views and templates just yet. To get rid of this error, let's just add a simple render:

```
class UsersController < ApplicationController
  def index
    render plain: "I'm in the index action!"
  end
end
```

Try again. It should work! Now take a look at the server log.

```
Started GET "/users" for 127.0.0.1 at 2017-07-14 11:43:06 -0700
Processing by UsersController#index as HTML
  Rendering text template
  Rendered text template (0.0ms)
Completed 200 OK in 1ms (Views: 0.6ms)
```

For every request, the server will tell you which controller and action is processing it. In this case, it was the UsersController's index action.

Woohoo! Your Postman request should have returned the string "I'm in the index action!" 
Victory is yours. Congratulations on successfully setting up, making, and processing your first Rails request.

## Playing with Parameters