{{partial:getting_started_series}}

# Building your product catalog

One of the first things every shop needs is a way to display its catalog to visitors.

To begin we will go ahead and create a controller which will be used to display our
products to our customers. All product information is stored and managed by Shoppe through
its admin interface.

```bash
$ rails generate controller products
```

Now we need to route some requests to the controller. Head over to your `config/routes.rb`
file and add the following routes.

```ruby
::config/routes.rb
get "product/:permalink", to: "products#show", as: "product"
post "product/:permalink", to: "products#buy", as: "buy"
root to: "products#index"
```

Let's now go and add some methods to our `ProductsController` to get some data.

```ruby
::app/controllers/products_controller.rb
class ProductsController < ApplicationController

  def index
    @products = Shoppe::Product.root.ordered.includes(:product_categories, :variants)
    @products = @products.group_by(&:product_category)
  end
  
  def show
    @product = Shoppe::Product.root.find_by_permalink(params[:permalink])
  end
  
end
```

In the `index` method we are getting all our products and grouping them by their category.
In the `show` method we are just getting a single product based on its permalink.

We should now go and create some views to show this information. Firstly, let's create a
view for our product list. Go and add the following into `app/views/products/index.html.erb`.

```rhtml
::app/views/products/index.html.erb
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

```rhtml
::app/views/products/show.html.erb
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

## Further techniques

* Display products based on their 'featured' boolean
* Display products within their categories
* Display stock information along with orders

