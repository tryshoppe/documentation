{{partial:getting_started_series}}

# Checking out

This tutorial will guide you through setting up your application to allow
your customers to "check out" and complete their orders.

By this point, you should have already configured your application to allow
customers to add items to your basket. The next step is customers are ready 
to complete their order, provide their details and payment information.

In this tutorial, we're going to construct a simple 3-step check out process.

1. Ask customer to provide details and address
2. Ask customer to provide their credit card details
3. Display order details & ask customer to confirm the order

We will create three routes for these steps:

```ruby
::config/routes.rb
match "checkout", to: "orders#checkout", as: "checkout", via: [:get, :patch]
match "checkout/pay", to: "orders#payment", as: "checkout_payment", via: [:get, :post]
match "checkout/confirm", to: "orders#confirmation", as: "checkout_confirmation", via: [:get, :post]
```

All our checkout actions will take place within the orders controller which 
we created in the [baskets](baskets) section of the tutorial.

Let's go ahead and create a link so users can get to our newly routed checkout pages.
Open up your application layout and add a "Checkout" link next to view basket.

```rhtml
::app/views/layouts/application.html.erb
<%= link_to "Checkout", checkout_path %>
```

Now we need to define a controller action for our initial step. Open your order controllers
and add the following method. We will be expanding this later but for now, we just want
to get a clean instance of the order which the user wishes to check out.

```ruby
::app/controllers/orders_controller.rb
def checkout
  @order = Shoppe::Order.find(current_order.id)
end
```

On our checkout page, we want to display the contents of the basket as well as providing
a form for visitors to enter their details.

```rhtml
::app/views/orders/checkout.html.erb
<h2>Checkout</h2>
<%= render 'items', :order => @order %>

<%= form_for @order, :url => checkout_path do |f| %>
  <dl>
    <dt><%= f.label :first_name, 'Name' %></dt>
    <dd><%= f.text_field :first_name %> <%= f.text_field :last_name %></dd>
    
    <dt><%= f.label :billing_address1, 'Address' %></dt>
    <dd><%= f.text_field :billing_address1 %></dd>
    <dd><%= f.text_field :billing_address2 %></dd>
    <dd><%= f.text_field :billing_address3 %></dd>
    <dd><%= f.text_field :billing_address4 %></dd>
    <dt><%= f.label :billing_postcode, 'Post code' %></dt>
    <dd><%= f.text_field :billing_postcode %></dd>
    <dd><%= f.collection_select :billing_country_id, Shoppe::Country.ordered, :id, :name, :include_blank => true %></dd>
    
    <dt><%= f.label :email_address %></dt>
    <dd><%= f.text_field :email_address %></dd>
    
    <dt><%= f.label :phone_number %></dt>
    <dd><%= f.text_field :phone_number %></dd>
    
    <dd><%= f.submit 'Continue' %></dd>
  </dl>
<% end %>
```

This will display the order items partial we created previously along with a form requesting
details from the user. Orders can have seperate delivery addresses however this is outside of
the scope of this tutorial.

In order to handle this form's submission, we need to make some changes to our `checkout` 
action we started earlier. Open up your orders controller and amend your `checkout` method
to look like this:

```ruby
::app/controllers/orders_controller.rb
def checkout
  @order = Shoppe::Order.find(current_order.id)
  if request.patch?
    if @order.proceed_to_confirm(params[:order].permit(:first_name, :last_name, :billing_address1, :billing_address2, :billing_address3, :billing_address4, :billing_country_id, :billing_postcode, :email_address, :phone_number))
      redirect_to checkout_payment_path
    end
  end
end
```

When the form is submitted, the approved parameters will be sent to the `Shoppe::Order#proceed_to_confirm`
method which will update the order and push the status of the order to `confirming`. Once
an order is in this stage, various validations will be required. If any validation errors
occur the form will be re-rendered and you may wish to display the ActiveRecord errors
as appropriate.

Up next is building a payment page. However, this process is payment processor specific and 
depends on exactly how you want to implement this. You may wish to refer to the documentation
provided in your payment processors Shoppe module to accomplish this. In this tutorial, we'll
just render a form asking for details and discard them. The checkout action we just implemented
will send users to our `payment` action. 

The following is the view which should be popped into the action's view page.

```rhtml
::app/views/orders/payment.html.erb
<h2>Make your payment</h2>
<%= form_tag do %>
  <dl>
    <dt><%= label_tag 'card_number' %></dt>
    <dd><%= text_field_tag 'card_number' %></dd>
    <dt><%= label_tag 'expiry' %></dt>
    <dd><%= text_field_tag 'expiry' %></dd>
    <dt><%= label_tag 'security_code' %></dt>
    <dd><%= text_field_tag 'security_code' %></dd>
    <dd><%= submit_tag "Continue" %></dd>
  </dl>
<% end %>
```

This form submits back to the same action. We'll now just implement that method in our controller.
This method will just redirect users to our confirmation page. You'll almost certainly want to 
do something else here but we can worry about that another day.

```ruby
::app/controllers/orders_controller.rb
def payment
  if request.post?
    redirect_to checkout_confirmation_path
  end
end
```

Finally, we're ready to display our confirmation page and complete the order. Open up the
`orders#confirmation` view  and pop the following into it.

```rhtml
::app/views/orders/confirmation.html.erb
<h2>Place your order</h2>
<%= render 'items', :order => current_order %>
<%= button_to "Place order" %>
```

Clicking on 'Place order' will re-call the same action as a POST request. At this point
we just need to mark the order as completed, clear the order from the customer's session
and congratulate the customer on their order.

```ruby
::app/controllers/orders_controller.rb
def confirmation
  if request.post?
    current_order.confirm!
    session[:order_id] = nil
    redirect_to root_path, :notice => "Order has been placed successfully!"
  end
end
```

You're done! Just test our your order process and then head over to your Shoppe interface
to view your newly placed order.

## Further techniques

* Implementing a payment gateway
* Catching errors during order confirmation
