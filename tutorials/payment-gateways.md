# Adding support for a payment gateway

## Stripe

Shoppe has a module for interfacing with Stripe. This small tutorial will
run through the steps in order to add it to your Rails application.

To begin, add the `shoppe-stripe` gem to your Gemfile

```ruby
::Gemfile
gem "shoppe-stripe", require: "shoppe/stripe"
```

and install the gem

```bash
bundle install
```

Edit your `application.js` file and add the following

```
::app/assets/application.js
//= require shoppe/stripe/form_handler
```

Now the Stripe API keys need to be entered into the settings page in the Shoppe admin.
Go to http://localhost:3000/shoppe/settings and enter the Secret & Publishable keys
from Stripe and set the currency code if required.

Stripe works by submitting cards details submitted by your users to their servers using
javascript before your form is submitted to your server. This means you don't need to worry
about PCI compliance. Once the card details have been received by Stripe, the server will
simply receive a token which can be used with the Stripe API to take payments later. You
can read more about Stripe.js on their [documentation page](https://stripe.com/docs/stripe.js).

To make this work, you need to create a payment form to request card details from your
customer. This module includes the nessessary javascript to handle the submission of the
data to Stripe and will present a token to the form's action URL automatically.

When creating the form, you should tag all fields which contain information which should
be sent to stripe with the data-stripe attribute. It should include one of the following 
values. The address information is only needed if you want to perform extra fraud checks.

+ number (required)
+ exp_month (required)
+ exp_year (required)
+ cvc
+ name
+ address_line1
+ address_line2
+ address_city
+ address_state
+ address_zip
+ address_country

In addition to fields for card details, you should also include a hidden field with the
`token` value in the `data-stripe` attribute.

Replace the contents of `payment.html.erb` with the following if you've been following
the other tutorials. You may notice that we are adding `name: nil` to the fields. This 
prevents the form from sending the non-encrypted details to your application.

```rhtml
::app/views/orders/payment.html.erb
<%= shoppe_stripe_javascript %>

<h2>Make your payment</h2>

<%= form_tag nil, class: "stripeForm" do %>
  <%= hidden_field_tag "stripe_token", nil, "data-stripe" => "token" %>
  <dl>
    <dt><%= label_tag 'card_number' %></dt>
    <dd><%= text_field_tag 'card_number', nil, name: nil, "data-stripe" => "number" %></dd>
    <dt><%= label_tag 'expiry_month' %></dt>
    <dd><%= text_field_tag 'expiry_month', nil, name: nil, "data-stripe" => "exp_month" %></dd>
    <dt><%= label_tag 'expiry_year' %></dt>
    <dd><%= text_field_tag 'expiry_year', nil, name: nil, "data-stripe" => "exp_year" %></dd>
    <dt><%= label_tag 'security_code' %></dt>
    <dd><%= text_field_tag 'security_code', nil, name: nil, "data-stripe" => "cvc" %></dd>
    <dd><%= submit_tag "Continue" %></dd>
  </dl>
<% end %>
```

Once your user's card details have been exchanged for a token, it will be entered into 
a hidden field which is sent to your application. The endpoint which receives this token
must now exchange it for a "customer token". This module includes the method needed to
handle the exchange and store it along with the order.

```ruby
::app/controllers/orders_controller.rb
def payment
  @order = Shoppe::Order.find(session[:current_order_id])
  if request.post?
    if @order.accept_stripe_token(params[:stripe_token])
      redirect_to checkout_confirmation_path
    else
      flash.now[:notice] = "Could not exchange Stripe token. Please try again."
    end
  end
end
```

The `accept_stripe_token` method will save a property to the order called
`stripe_customer_token` which will be used when the order is confirmed to take
payment automatically.

Your Stripe integration is now complete!

More information about the `shoppe-stripe` order can be found on [GitHub](https://github.com/tryshoppe/shoppe-stripe).