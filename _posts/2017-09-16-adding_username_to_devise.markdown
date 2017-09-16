---
layout: post
title:  "Adding Username to Devise"
date:   2017-09-16 20:37:41 +0000
---

Devise is great for out-of-the-box user authentication, but it can feel like you get what Devise gives you and nothing else.

I had a bit of a struggle adding a username to Devise so I thought I'd store it here for easy reference in case I or anyone else needs it again.

**STEP ONE: ADD USERNAME FIELD TO USERS TABLE**

Yeah, I know it's super basic and who would ever forget this first step? Me? Yes. Yes I did. Shut up, it's not funny.

*Migration*
`rails generate migration add_username_to_users username:string:uniq`

*run the migration*
`rake db:migrate`

**STEP TWO: MODIFY APPLICATION CONTROLLER**

Modify `application_controller.rb` by adding a method `configure_permitted_parameters` and add :username, :email, :password, :password_confirmation, and :remember_me.

```
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    added_attrs = [:username, :email, :password, :password_confirmation, :remember_me]
    devise_parameter_sanitizer.permit :sign_up, keys: added_attrs
    devise_parameter_sanitizer.permit :account_update, keys: added_attrs
  end
end
```

**STEP THREE: CREATE A VIRTUAL ATTRIBUTE CALLED 'LOGIN' IN THE USER MODEL**

You can add `login` as an `attr_accessor` in your User model and then it will refer to either :username or :email.

```
class User < ApplicationRecord

attr_accessor :login
```

If :login is going to be used elsewhere in your code, write a reader and writer instead of using `attr_accessor`.

```
class User < ApplicationRecord

  def login=(login)
    @login = login
  end

  def login
    @login || self.username || self.email
  end
```

**STEP FOUR: TELL DEVISE TO USE LOGIN IN AUTHENTICATION KEYS**

Finally, add this to your `config/initializers/devise.rb` file:

```
config.authentication_keys = [ :login ]
```

That should do it. You still have to modify your views so username shows up, but now you can go cray throwing :username into your code and letting your users know they are special, because they are, dammit. 
