---
layout: post
title:  Practical 9
author: CS424
date:   2015-11-19 11:00:00
categories: practical
---

**1.**
Finish Practical Sheet 8.
Alternatively, 
move your old depot application out of the way
and get a fresh copy 
by first creating a fork of <https://github.com/gpfeiffer/depot.git>
on your `github` page, and secondly cloning the fork into your
working directory.
Then `cd depot` and run `rake db:migrate` and `rake db:seed` to set up the database.

**2. Authentication.**  In this last practical 
of the course, you will set up user authentication and the deploy the `depot` appliaction on `heroku`.  This will hopefully make it possible for `depot` to send emails, a feature that would also be useful for restoring forgotten passwords ...

**3.** It is actually not too complicated to set up authentication
from scratch.  That's what the book does, and important lessons can be
learned on the way.  Here, we will follow a different path and use a
`gem` called `devise` for this task.  Provided that the gem is already
installed on your machine (if not, use your superuser privileges for a `sudo gem install devise -v 2.2.8` command), you can register it by adding the line

    gem 'devise', '~> 2.2.8'

to your `Gemfile` (we need to stick to that old version for compatibility with the version of rails we are using), and execute

    bundle install 

on the command line.

**4.** Next, execute the command:

    rails generate devise:install

Which files does it generate?  (Answer in the comment box below.)  If
all goes well, you also get 5 pieces of advise as a result of this
command.  Consider parts 1, 2, and 3 of these, and ignore parts 4 and
5 for now.

**5.** Now, we need to generate a model (and database table)
for usernames and passwords.  That's what the command

    rails generate devise user

does.  As you see, it creates a `user` model and a migration to
create a corresponding table in the database.  Inspect that file.
What components does the `user` model have?  What are the columns of the `users` table? (Answer in the comments box below.)
Run the migration to actually create the database.

**6.** Restart the server, and test whether your web site behaves any different.
Why should it?  In order to enforce authentication, add the line

    before_filter :authenticate_user!

to the application controller (which file?).  Now try your web site again.
(And try your neighbour's!)

**7.** Your shop is now in a state where everyone can simply sign up
and then modify the database.  The actual management of who is allowed to do what can be achieved through the introduction of _role based authorization_.
Here we simply 
cut off the `sign up` functionality.
New users then have to be signed up manually by an already registered user.
For this,  replace the `devise_for :users` line in `config/routes.rb` by

~~~~
  devise_for :users, :skip => [:registrations]
  as :user do
    get 'users/edit' => 'devise/registrations#edit', :as => 'edit_user_registration'
    put 'users' => 'devise/registrations#update', :as => 'user_registration'
  end
~~~~

**8.** Now, only registered users can see pages on the site.  In order to
open the site for unregistered customers, some restrictions need to be lifted.
For starters, add

~~~~
  skip_before_filter :authenticate_user!
~~~~

to the top of the store controller.  This allows people to
see the store catalogue without having to register first.
Then, they would need to be able to create line items and carts
to do their shopping.  For this, add

~~~~
  skip_before_filter :authenticate_user!, :only => [:create, :show]
~~~~

to both the carts controller and the line items controller.
There might be other actions that need to be treated in this way.

Documentation for devise can be found at its `github` pages
at <https://github.com/plataformatec/devise>.

**9.** Next, you probably want a button or so to log out of
the web site.  Study the source code of this blog at
<https://github.com/gpfeiffer/cs424blog> to find out how it's done here.

**9a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.

    git add .
    git commit -m "added authentication with devise."
    git push

**10. Deploying.**
We'll try to deploy our store application using the free services
provided by Heroku.  Go to <https://www.heroku.com/> and sign up with
your email address.  A confirmation email should then arrive in your mailbox.
After confirming, you are asked to pick a password for your new account.
Next, you'll be presented with your _dashboard_.

**11.**  The environment on heroku differs in some important ways from ours.
First, they prefer to always use the latest version of rails.
And then they use postgresql as their database, instead of `sqlite3`.
All of this can be taken into account.
Add the line 

    ruby '1.8.7'

to your `Gemfile` (just before the ` gem 'rails'` line), and replace
the line `gem 'sqlite3'` by

~~~~
group :development, :test do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
end
~~~~

Also add 

    gem 'rails_12factor'

to the end of the file.

**12.** It is necessary to update the `Gemfile.lock`, too.  In order to prevent them from installing gems that are meant for production only, use `bundle` with options:

    bundle install --without production

Also add the line

    config.assets.initialize_on_precompile = false

near the end of `config/application.rb`.

**12a.** If all works fine, commit your changes to the local
`git` repository: this step is a prerequisite for the next.

**13.** The communication with your application on the heroku cloud is facilitated through the `heroku-toolbelt` package.
If already installed, it provides the command `heroku`, see `heroku help` for a list of things it can do.
Then deploy your app with the commands:

    heroku create
    git push heroku master

And in order to set up a database on heroku and populate it we the books
we know:

    heroku run rake db:migrate
    heroku run rake db:seed

Then your shop should be up and running on heroku ...

(If `heroku-toolbelt` is not installed,  someone with superuser powers should follow the instructions on
<https://toolbelt.heroku.com/debian>,
then turn the `http` in the file `/etc/apt/sources.list.d/heroku.list` into
`https`, then run `sudo apt-get update` and `sudo apt-get install heroku-toolbelt`.)

**14. Email.** In order to set up sending of emails, the environment variables
form last week's practical need to be set on heroku:

~~~~
heroku config:set GMAIL_USERNAME=........
heroku config:set GMAIL_PASSWORD=........
~~~~

Now emails from your online shop can be sent for (at least) two different purposes:  to confirm orders, and to allow registered users to choose a new password in case they forgot the old one ...