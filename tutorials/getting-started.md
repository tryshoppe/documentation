# Getting started with Shoppe

In this tutorial, we're going to build a new Shoppe store from scratch into a brand new 
Rails application. We will start at the every beginning and guide you through all the
steps needed to build a simple store with a basket and checkout process.

We're assuming that you have Rails, Ruby, RubyGems & Bunlder installed at their latest
versions. This guide was produced using Rails 4.0.0 & Ruby 2.0.

During this tutorial, we will execute commands such as `rails` or `rake`. You may find
you need to prefix these commands with `bundle exec` depending on your local system
setup.

## Getting started

To begin, we need to create a basic Rails application for our Store. We're going to
call our store **Widgets Inc**.

```bash
$ rails new widget_shop
```

Your Rails application will be created in a directory called `widget_shop`. Open this
up and add the following to your Gemfile.

```ruby
gem "shoppe"
````

Once added and saved, you need to reload your bundle:

```bash
$ bundle
```

You now need to run a few generators which will add various & content to your local
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
# => Rails 4.0.0 application starting in development on http://0.0.0.0:3000
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

## Displaying your products

The first thing we need to do is create a controller which will be used to display
our products to our customers.

```bash
$ rails generate controller products
```

Now we need to route some requests to the controller. Head over to your `config/routes.rb`
file and add the following routes.

```ruby
get 'product/:permalink' => 'products#show', :as => 'product'
root :to => 'products#index'
```

Let's now go and add some methods to our `ProductsController` to get some data.

```ruby
class ProductsController < ApplicationController

  def index
    @products = Shoppe::Product.root.ordered.includes(:product_category, :variants, :default_image).group_by(&:product_category)
  end

  def show
    @product = Shoppe::Product.find_by_permalink(params[:permalink])
  end
  
end
```

In the `index` method we are getting all our products and grouping them by their category.
In the `show` method we are just getting a single product based on its permalink.

We should now go and create some views to show this information. Firstly, let's create a
view for our product list. Go and add the following into `app/views/products/index.html.erb`.

```html
<h2>Products</h2>

<% @products.each do |category, products| %>
  <h3><%= category.name %></h3>
  <ul>
    <% products.each do |product| %>
    <li>
      <h4><%= link_to product.name, product_path(product.permalink) %></h4>
      <p><%= product.short_description %></p>
      <p><b>Price:</b> <%= number_to_currency product.price %></p>
    </li>
    <% end %>
  </ul>
<% end %>
```

If you now take a look at http://localhost:3000 (when your server is running), you'll see a
list of products along with a short description and a price for the product. Clicking 
on the product, will take you the product's `show` action.

![Image](http://s.adamcooke.io/cZDqP.png)

We'll now add a view for the product show action.

```html
<h2><%= @product.name %></h2>

<% if @product.default_image %>
  <%= image_tag @product.default_image.path, :width => '200px', :style => "float:right" %>
<% end %>

<p><%= @product.short_description %></p>
<p>
  <b><%= number_to_currency @product.price %></b>
  <%= link_to "Add to basket", product_path(@product.permalink), :method => :post %>
</p>

<hr>
<%= simple_format @product.description %>
<hr>

<p><%= link_to "Back to list", root_path %></p>
```

If you click on one of the products in the list, you should now see some information 
about the product along with it's image and a link for adding it to your basket.

![Image](http://s.adamcooke.io/4nfIL.png)
