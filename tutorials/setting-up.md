{{partial:getting_started_series}}

# Setting up in a new application

To begin, we need to create a basic Rails application for our Store. We're going to
call our store **Widgets Inc**.

```bash
$ rails new widget_shop
```

Your Rails application will be created in a directory called `widget_shop`. Open this
up and add the following to your Gemfile.

```ruby
::Gemfile
gem 'shoppe', '~> 1.0'
````

Once added and saved, you need to reload your bundle:

```bash
$ bundle
```

You now need to run a few generators which will add various content to your local
Rails application. 

```bash
$ rails generate shoppe:setup
$ rails generate nifty:attachments:migration
$ rails generate nifty:key_value_store:migration
```

The first of these will add a route to your `config/routes.rb` file specifying that
your Shoppe admin interface will be mounted at `/shoppe`. The second two will create
database migrations used by two dependencies of Shoppe. [Nifty Attachments](https://github.com/niftyware/attachments)
is used to store images in your database and the [Nifty key value store](https://github.com/niftyware/key-value-store)
is used to store key/values for certain Shoppe models. 

Next up is adding the appropriate tables to your database and loading the initial
data needed for Shoppe.

```bash
$ rake db:migrate shoppe:setup
```

Shoppe ships with a set of data which can be very useful while developing. It includes
a number of products, delivery services & tax rates. We will go ahead and load this
data into our datbase.

```bash
$ rake shoppe:seed
```

## Starting your application

The Shoppe system is now added to your application. Let's start it up and confirm it's
working as we would expect.

```bash
$ rails server
# => Booting WEBrick
# => Rails 4.0.1 application starting in development on http://0.0.0.0:3000
# => Run `rails server -h` for more startup options
# => Ctrl-C to shutdown server
# [2013-10-29 14:35:09] INFO  WEBrick 1.3.1
# [2013-10-29 14:35:09] INFO  ruby 2.0.0 (2013-06-27) [x86_64-darwin12.4.0]
# [2013-10-29 14:35:09] INFO  WEBrick::HTTPServer#start: pid=5213 port=3000
```

Once the server is up and running, go ahead and navigate to http://localhost:3000/shoppe.
You will be asked for a username & password, enter `admin@example.com` and `password`.
You can change these from the Users page within here however for development we can 
leave them as-is.

![Image](http://s.adamcooke.io/EKsQw.png)
