---
layout: post
title:      "Getting a JWT From Rails to React"
date:       2021-01-15 13:31:45 -0500
permalink:  getting_a_jwt_from_rails_to_react
---


Against all given advice (due to the complexity that arises through our project scope), I decided to use some form of authentication in my app.  I worked through this issue with a friend in the cohort and here is what we learned:  "It can be a little tricky".

This article is intended as a brief instruction on generating the tokens and making sure they can be passed through to your front end applications.  I highly recommend viewing or searching for additional articles to get a deeper understanding of everything that happens along this process.

Luckily, after the amount of time we spent following other guides and trying to mix and match our requirements, we managed to get a handle on things.  The basic idea of this article is to walk you through the process of generating a JWT token in your Rails backend and being able to present it to your frontend when you are fetching your routes. I'll try to summarize the steps for building out your token generator below:

### 1.  Build out your application with ```rails new <app-name>```

Let Rails set up the basic framework and be sure you do NOT use the ```--api``` flag in order for this to work.  Once everything finishes getting set up, go ahead and open your files and open up your project's .GEMFILE so we can add in these necessary gems:

* ```bcrypt, '~> 3.1.7'```
* ```jwt```
* ```dotenv-rails```

### 2. Build out your 'User' model with your desired params

For my example, I built out a User model which contained a name, email, and password_digest.  I'd recommend using a scaffold in this scenario to build out all of your CRUD routes as you can simply delete any of the ones you won't be using later.  I also didn't specify my types when entering the command below as they will default to strings when not specifically paired with a type i.e. ```age:integer```

```rails g scaffold User name email password_digest```

You should now have enough functionality to start up your rails server though the results will be empty.  We'll be modifying the actions a little further in, so we can leave that be for now.

### 3. With our Users in place, we'll need a controller for authenticating them

We'll be spending most of our time in our controllers to get this set up.  Go ahead and add in a file for your 'auth_controller.rb' within the controllers folder and let's take a look at the code:

```
class AuthController < ApplicationController
    skip_before_action :require_login, only: [:login, :auto_login]
    # Calling this route will respond with a user and a JWT token if successfully authenticated
    def login
        user = User.find_by(email: params[:user][:email])
        if(!user)
            render json: { error: "No account associated with this email exists" }, status: :unauthorized
        else   
            if user.authenticate(params[:user][:password])
                token = JWT.encode({
                    user_id: user.id,
                    name: user.name,
                }, ENV['SECRET_KEY'])
                render json: {user: user, token: token}
            else
                render json: { message: 'Wrong password'}
            end
        end
    end

    # This route can be called to check the local system for an existing token and therefore return a user
    def auto_login
        if session_user
            render json: session_user
        else
            render json: {message: 'No currently logged in user'}
        end
    end
end
```

Here we have two methods defined 'login' and 'auto-login'.  When building your front-end, you'll use these routes with your fetch requests when trying to authenticate your sign-in. The first method searches for a user based upon the requested params.  If a user isn't found, an error message is generated and sent back in response.

If a user is found and the password successfully authenticates their account, a JWT token is generated.  Note that it will store our user's name and id in my example but can contain any information you feel is relevant to your application.   It comes from the 'jwt' gem we installed and is encoding whatever object we choose to pass in. Just be sure to avoid storing potentially sensitive information.  

The last object being passed to the JWT.encode() method is a Secret Key.   This key is just a string of characters and can be whatever you would like.  In my example, I've stored a secret key within a `.env` file and am calling that key as my final encoding argument.  We'll build a decode method which needs this key as well in order to successfully verify the authenticity of our token.

Then the user and the newly generated token is rendered in JSON as your response back to the frontend which can be used for authentication.


### 4. Provide Your Application Controller with the necessary Methods

This is where it starts to get a little bit tricky.  We need to add methods into our Application Controller which will be responsible for a few things:

* Receiving an existing token and decrypting it
* Passing the token over to our User application when signing in or signing up
* Checking the client's system to see if a token already exists on their machine

**Another thing to keep in mind is that you'll need to store these tokens in your browser's local storage in order to use the auto-login method defined below**

The methods used here are chained together to verify whether someone is logged in or needs to be logged in.  Here's a quick look at my entire controller:

 ```
class ApplicationController < ActionController::API
    before_action :require_login

    def logged_in?
        !!session_user
    end

    def require_login
        render json: {message: 'Please log in'}, status: :unauthorized unless logged_in?
    end

    def auth_header
        request.headers['Authorization']
    end

    def decoded_token
        if auth_header
            token = auth_header.split(' ')[1]
            begin
                JWT.decode(token, ENV['SECRET_KEY'], true, algorithm: 'HS256')
            rescue JWT::DecodeError
                []
            end
        end
    end

    def session_user
        decoded_hash = decoded_token
        if !decoded_hash.empty?
            user_id = decoded_hash[0]['user_id']
            @user = User.find_by(id: user_id)
        else
            nil
        end
    end
end
```

First we can start with our 'logged_in?' method.  It is checking the result of our 'session_user' method down at the bottom of our code.  It simply returns true or false.

The 'require_login' method will simply render a login message to the front end if our 'logged_in?' method returns 'false'

Our 'auth_header' method is necessary for requesting specific authorization headers from our client computer when they send over a request to check for a user.  This header includes the JWT token that we will decrypt to begin verifying our user.

Next, if 'auth_header' recieves a valid response, the 'decoded_token' method splits the header and pulls out the token it has received.  It then begins the decryption process using our Secret Key from before to verify that our token is valid.

Lastly we look in our 'session_user' method. If the JWT token was decrypted successfully, the user can be found within our database and returned as the result. 
This is what gives us the ability to use the 'auto-login' method from our Authentication Controller without the need to authenticate our credentials the next time they navigate to a specified web-page. 


### 5. Modify your Users Controller to produce a token when creating a new User account (optional)

The above steps work for an existing user because the credentials have been authenticated, but for a new user, we'll need to build a shorthand version of our JWT token generator.  This is optional, however, if you are okay with having a new user register for an account and then redirect them to the login screen rather than signing them in directly after registration.

Our scaffold command should have the CREATE method ready to go for us, but we'll modify it to look like the following:

```
 def create
    @user = User.new(user_params)

    if @user.save
      payload = {user_id: @user.id, name: @user.name}
      token = JWT.encode(payload, ENV['SECRET_KEY'])
      render json: {user: @user, token: token}, status: :created
    else
      render json: @user.errors, status: :unprocessable_entity
    end
  end
```
	

 This just makes sure a token is sent back as part of the response when a new user is created.
	
### 6. Configure your Routes
	
The last thing you'll need to do is just make sure you've added the proper routing requests to your 'routes.rb' file.  Specifically the custom ones we just created:

```
post '/login', to: 'auth#login'
get '/auto_login', to: 'auth#auto_login'
```

And that should be pretty much everything you need in order to generate and present your JWT tokens to your front-end application from your Rails back-end.  There are plenty of guides and best-practices for how to store and handle them out there, but once you've set up your fetch requests on the frontend, you should be able to log the responses and view the data coming through!  
