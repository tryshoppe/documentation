# Extending Shoppe

## Order Model

The Order model in Shoppe has 3 different callbacks that can be ran. There are
currently `confirmation`, `acceptance` and `rejection`.

You can see all 3 of these callbacks being used in the [shoppe-stripe gem](https://github.com/tryshoppe/shoppe-stripe)
at different parts of an order.

## Building a Staff Order Notification Module

In this example, we will build a gem which will send staff an email saying
a new order has been created. This will use the `confirmation` callback.

```bash
$ bundle gem shoppe-notification
```

Open the newly created directory `shoppe-notification` in your favourite 
text editor and open the `shoppe-notification.gemspec` file. Edit the required
information such as summary, description and homepage (if required) and 
add the Shoppe gem as a dependency.

```ruby
::shoppe-notification.gemspec
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'shoppe/notification/version'

Gem::Specification.new do |spec|
  spec.name          = "shoppe-notification"
  spec.version       = Shoppe::Notification::VERSION
  spec.authors       = ["Dean Perry"]
  spec.email         = ["dean@voupe.com"]

  spec.summary       = "A Shoppe module which emails staff on new orders"
  spec.description   = "A Shoppe module which emails staff on new orders"
  spec.homepage      = "http://tryshoppe.com"

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_dependency "shoppe"

  spec.add_development_dependency "bundler", "~> 1.8"
  spec.add_development_dependency "rake", "~> 10.0"
end
```

Once the gemspec file has been edited, we need to add a `setup` method
which will link to the `confirmation` callback. Add the following to 
the `lib/shoppe/notification.rb` file.

```ruby
::lib/shoppe/notification.rb
def self.setup
  Shoppe::Order.before_confirmation do
    Shoppe::NotificationMailer.order_received(self).deliver_now
  end
end
```

Create a new file at `lib/shoppe/notification/engine.rb` and enter the following

```ruby
::lib/shoppe/notification/engine.rb
module Shoppe
  module Notification
    class Engine < Rails::Engine

      initializer "shoppe.notification.initializer" do
        Shoppe::Notification.setup
      end

    end
  end
end
```

and add this to the top of `notification.rb`, the file we edited earlier

```ruby
require "shoppe/notification/engine"
```

Create the Mailer at `app/mailers/shoppe/notification_mailer.rb` with the following code

```ruby
::app/mailers/shoppe/notification_mailer.rb
module Shoppe
  class NotificationMailer < ActionMailer::Base
  
    def order_received(order)
      @order = order
      staff = Shoppe::User.all.map(&:email_address)
      mail from: Shoppe.settings.outbound_email_address, to: staff, subject: "New Order Received"
    end

  end
end
```

The `Shoppe::User.all.map(&:email_address)` code collects all the email addresses for 
users who can login to Shoppe. This doesn't include any email addresses for orders.

Once the mailer has been created, you can now create the email text file that will
be rendered and sent to the staff.

```
::app/views/shoppe/notification_mailer/order_received.text.erb
Hello Team!

Re: Order #<%= @order.number %>

We're pleased to let you know that we have just received a new order! You might want to accept and ship it.

Many thanks,

<%= Shoppe.settings.store_name %>
<%= Shoppe.settings.email_address %>
```

The main part of the Gem has been completed. All you need to do now is push the gem to
Rubygems (if you want) and add it to the Gemfile of your project and that's it! When a new
order has been received, staff will receive an email saying so.
