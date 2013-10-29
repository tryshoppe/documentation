{{partial:getting_started_series}}

# Working with Shoppe baskets

Shoppe has the concept of orders - an order is a collection of products along with
other information about how they should be delivered and billed to a customer.

A basket is simply an order which has not yet been "checked out". Therefore, whenever
someone adds a product to their basket, there must also be an order in the backend
database.

## Accessing the basket

We recommend adding the concept of a `current_order` to your application. This means
that you can, at any point, call `current_order` to receive an order which you can
put items into.

The following two private methods should be added to your `app/controllers/application_controller.rb`
file.

```ruby
class ApplicationController < ActionController::Base
  
  private
  
  def current_order
    @current_order ||= begin
      if has_order?
        @current_order
      else
        order = Shoppe::Order.create(:ip_address => request.ip)
        session[:order_id] = order.id
        order
      end
    end
  end
  
  def has_order?
    !!(
      session[:order_id] &&
      @current_order = Shoppe::Order.includes(:order_items => :ordered_item).find_by_id(session[:order_id])
    )
  end
  
  helper_method :current_order, :has_order?
  
end
```

The `current_order` method will always return an instance of `Shoppe::Order`. The first time this 
is called for a visitor, an order will be created and its ID stored in a session. Any subsequent
calls within the same session will simply return the created order.

The `has_order?` method will return true or false depending on whether an order has been created
for the current session.

## Adding products to the basket

Once you have added these methods, we can go ahead and implement the `buy` method on the products controller.
We already created a route for this earlier.

```ruby
def buy
  @product = Shoppe::Product.find_by_permalink!(params[:permalink])
  current_order.order_items.add_item(@product, 1)
  redirect_to product_path(@product.permalink), :notice => "Product has been added successfuly!"
end
```

Let's go through this method. Firstly, we find the product we want to add. Secondly, we use our
`current_order` instance to add a new item to it by passing the product we wish to add plus the
quantity. Finally, we redirect users back to the product page with a message.

Of course, you can do different things with this - you may wish to do this with AJAX or may
want to allow users to add more than one item at a time. Those things are outside of the scope
of this tutorial.

## What's in the basket?

We haven't yet done anything to display the contents of the basket so although we
may have added items, they just disappear.

Let's open up our application layout at `app/views/layouts/application.html.erb` and pop the
following just after the opening `<body>` tag.
  
```erb
<% if has_order? %>
<p style="border:1px solid black;padding:10px;">
  You have <%= pluralize current_order.total_items, 'item'%> in your basket which cost
  <%= number_to_currency current_order.total_before_tax %>.
</p>
<% end %>
```

This will insert a box at the top of each page with your current basket price and 
number of items.

![Image](http://s.adamcooke.io/FazPV.png)

I'm sure people will forget what they put in their basket, so we should also add a view which
allows visitors to see what's in their basket.

Before we can build a page, we'll need to add a route. Add the following route to your
`config/routes.rb` file.

```ruby
get 'basket' => 'orders#show'
```

We'll also need a controller because it doesn't really fit into our products controller.

```bash
$ rails generate controller orders
```

Let's make a quick partial to render a table or products. We'll probably be needing this 
in other places as part of the checkout process so let's partial-it frmo the start.

Open up `app/views/orders/_items.html.erb` and pop the following in:

```erb
<table width='100%' border='1'>
  <thead>
    <tr>
      <td>Quantity</td>
      <td>Product</td>
      <td>Sub-Total</td>
      <td>Tax</td>
      <td>Total</td>
    </tr>
  </thead>
  <tbody>
    <% order.order_items.each do |item| %>
    <tr>
      <td><%= item.quantity %></td>
      <td><%= item.ordered_item.full_name %></td>
      <td><%= number_to_currency item.sub_total %></td>
      <td><%= number_to_currency item.tax_amount %></td>
      <td><%= number_to_currency item.total %></td>
    </tr>
    <% end %>
    
    <% if order.delivery_service %>
    <tr>
      <td></td>
      <td><%= order.delivery_service.name %></td>
      <td><%= number_to_currency order.delivery_price %></td>
      <td><%= number_to_currency order.delivery_tax_amount %></td>
      <td><%= number_to_currency order.delivery_price + order.delivery_tax_amount %></td>
    </tr>
    <% end %>
    
  </tbody>
  <tfoot>
    <tr>
      <td colspan='4'>Sub-Total</td>
      <td><%= number_to_currency order.total_before_tax %></td>
    </tr>
    <tr>
      <td colspan='4'>Tax</td>
      <td><%= number_to_currency order.tax %></td>
    </tr>
    <tr>
      <td colspan='4'>Total</td>
      <td><%= number_to_currency order.total %></td>
    </tr>
  </tfoot>
</table>
```

This table will include all items in the order plus any delivery service which is appropriate
for the order. In some instances, you may wish to allow users to choose a delivery service,
while this is possible it is outside the scope of this tutorial. Our example store implements this
so you can take a look there for an example.

Now, in order to render this we need to pop a quick file into `app/views/orders/show.html.erb`.

```erb
<h2>Your basket</h2>
<%= render 'items', :order => current_order %>
```

You can now browse to `/basket` to view this. Let's add a link to our basket information section
in the application layout though.

```erb
<%= link_to "View basket", basket_path %>
```

![Image](http://s.adamcooke.io/8l5ta.png)

## Emptying the basket

Users may wish to remove all the items in their basket. In this case, you can just delete the
current_order object. To do this, we'll just a create a route in `config/routes.rb` as follows:

```ruby
delete 'basket' => 'orders#destroy'
```

Then we'll add a `destroy` method to our orders controller.

```ruby
def destroy
  current_order.destroy
  session[:order_id] = nil
  redirect_to root_path, :notice => "Basket emptied successfully."
end
```

Finally, we need to link to this. We'll just put a link on the bottom of a basket page.

```erb
<p><%= link_to 'Empty basket', basket_path, :method => :delete %></p>
```


## Further techniques

* Allow customers to change the quantity of items in their basket
* Allow customers to remove individual items from the basket
* Allow customers to change their delivery service
* Catch errors when a product is not in stock and added to a user's basket
