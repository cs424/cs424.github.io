---
layout: post
title:  Practical 6
author: CS424
date:   2015-11-05 11:00:00
categories: practical
---
**0.** Finish Practical Sheet 5.

(Alternatively, 
move your old depot application out of the way
and get a fresh copy 
by first creating a fork of <https://github.com/gpfeiffer/depot.git>
on your `github` page, and secondly cloning the fork into your
working directory.
Then `cd depot` and run `rake db:migrate` and `rake db:seed` to set up the database.)

**1. Partials.** A _partial_ is a named chunk of a view template in
its own file.  With the `render` method, it can be included into
views, layouts, or even other partials.  And you can pass arguments to
a partial, ... it really has a lot in common with a subroutine of a computer program.

The filename of a partial starts with an underscore (`_`),
followed by the name of the partial and the extension
`.html.erb`.

Let's start by moving the formatting of line items in the shopping cart
into its own file.  This is a sufficiently complex task too deserve
a place of its own. In the `app/views/line_items` folder, create a
file `_line_item.html.erb`.  Note the leading `_` and the use of singular
in the file name.  This file now contains a partial with name `line_item`
(without leading `_`).  Copy the code that describes the table row
of a line item in a shopping cart from the cart view into this new file:
{% highlight html+ruby %}
<tr>
  <td><%= item.quantity %>&times;</td>
  <td><%= item.product.title %></td>
  <td class="item_price"><%= number_to_currency(item.total_price) %></td>
</tr>
{% endhighlight %}
Then replace each occurence of `item` by `line_item` (3 times) so that it matches the name of
the partial.

**2.** Now replace the loop over all line items which contains that
chunk in the cart view by the single line:
{% highlight html+ruby %}
<%= render @cart.line_items %>
{% endhighlight %}
Reload the page and make sure it still works as desired.  If it works, the
`render` method has correctly identified the thing to render as a list of line items, has found the instructions for rendering a single line item in the
`line_item` partial in the `line_items` view folder and has applied this
to each item in the list.
Note how a series of naming conventions makes it possible to remove
one level of complexity from the program.

**3.** Next, 
we are going to wrap up the shopping cart display into a partial,
so that we can easily include it in the side bar of the catalog page.

Following the sam naming conventions as before, 
create a new file for a partial `cart`
(which name? which folder?).
Then copy the contents of the cart view
`show.html.erb` into the new file
and replace each occurrence of `@cart` by `cart`
so that it matches the name of
the partial.  Also remove the 
notice at the top of the file.

You can now replace everything except for that notice
in the original cart view file by the single line:
{% highlight html+ruby %}
<%= render @cart %>
{% endhighlight %}
Check whether it still works as desired.  Actually, what do the tests report?

**4.**  You are now in a position to display a cart wherever you want,
with a single command.
In order to have a cart to display on the store page, add the lines
{% highlight ruby %}
include CurrentCart
before_action :set_cart
{% endhighlight+ruby %}
before the `index` method of the store controller (which file?).
This will take care of setting `@cart` to the current cart
when the `index` action is called, and make the variable available
in the view.

**5.**
The partial can now be invoked by the text
{% highlight html+ruby %}
<div id="cart">
  <%= render @cart %>
</div>
{% endhighlight %}
inside the `<div>` with ID `side` of the layout
(which file?).  Try it out!

**5a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.

    git add .
    git commit -m "moved cart into a partial."
    git push

**6.** We need to modify the styling rules for carts so that
they not only apply to pages created by the carts controller.
This is done by replacing `.carts {` in the `carts.css.scss` file
by the sequence
{% highlight scss %}
.carts, #side #cart {
{% endhighlight %}

**7.** In the `application.css.scss` style file, add
{% highlight scss %}
form, div {
	display: inline;
}

input {
	font-size: small;
}

#cart {
	font-size: smaller;
	color: white;

	table {
	    border-top: 1px dotted #595;
	    border-bottom: 1px dotted #595;
	    margin-bottom: 10px;
	}
}
{% endhighlight %}
to the `#side` bracket, before the `ul` bracket.

**8.**  Now adding a line item should redisplay the store
and not the cart. Change the appropriate line in the `create` method
of the line items controller to
{% highlight ruby %}
format.html { redirect_to store_url }
{% endhighlight %}

**8a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.

**9. Ajax.** 
If only the contents of the cart has changed then
only that portion of the page really needs to be updated,
rather than requesting, composing and sending a new version of the entire page.
 _Ajax_ refers to the ability of the browser to
change the internal (DOM) representation of a page, and to interact
with the server on that level.  This usually involves
a few (little) changes to the existing programs.
First we instruct the `"Add to Cart"` button to send an Ajax request
to the server (rather than requesting a new copy of the entire page.
This is done by adding the option `remote: true` to the button (which file?):
{% highlight html+ruby %}
<%= button_to 'Add to Cart', line_items_path(product_id: product),
  remote: true %>
{% endhighlight %}

**10.** 
On the receiving end, allow
the `create` method of the line items
controller to respond to Ajax requests
by adding one line
{% highlight ruby %}
format.js
{% endhighlight %}
to the `create` method of the line items
controller, after the line you changed in step 8.



**11.**
Finally, in the line items views folder, create a file `create.js.erb`
with a single line of _JavaScript_:
{% highlight javascript %}
$('#cart').html("<%= escape_javascript render(@cart) %>")
{% endhighlight %}
Executing this code will replace contents of the `<div>` with 
ID `cart` by the result of rendering the cart partial.

Try it out: the page should refresh much quicker and smoother!

**11a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.

**12. Highlights.** JavaScript can do way more with your pages.
In order to avail of some predefined effects, include the line

    gem 'jquery-ui-rails'

into your `Gemfile` and run

    bundle install

Then add
{% highlight javascript %}
//= require jquery.ui.effect-blind
{% endhighlight %}
to the javascript assets in the file `application.js`.

**13.** In order to highlight the latest item in a cart, we need a way
to identify it.  For that, replace `format.js` in the line item
controller's `create` method by:
{% highlight ruby %}
format.js { @current_item = @line_item }
{% endhighlight %}
Then, in the `line_item` partial, replace the opening `<tr>` by:
{% highlight html+ruby %}
<% if line_item -- @current_item %>
<tr id="current_item">
<% else %>
<tr>
<% end %>
{% endhighlight %}
I don't like this code (why?) but it marks the latest item with a unique ID,
that javascript can use to modify the corresponding part of the DOM.
Add
{% highlight javascript %}
$('#current_item').css({'background-color':'#88ff88'}).
  animate({'background-color':'#114411'}, 1000)
{% endhighlight %}
to the `create.js.erb` template in the line items view folder.

Add a new item a watch Yellow Fade in action.

**14. Empty Carts.** If the cart is empty, there is no need to display it.
One way to achieve this is to set 'display: none' in the CSS style
of the cart `<div>`.
In order to construct such a `<div>` element with a conditional attribute, 
it is convenient to supply a helper method that uses the 
`content_tag` method for this purpose.  Add
{% highlight ruby %}
def hidden_div_if(condition, attributes = {}, &block)
  if condition
    attributes["style"] = "display: none"
  end
  content_tag("div", attributes, &block)
end
{% endhighlight %}
to the `ApplicationHelper` module in the `app/helpers` folder.
Then replace the `<div>` with ID `"cart"` in the application layout by
{% highlight html+ruby %}
<%= hidden_div_if(@cart.line_items.empty?, id: 'cart') do %>
  <%= render @cart %>
<% end %>
{% endhighlight %}

The `jquery` library has various effects built in. In order to
gradually display a newly created cart, add the following line to the
top of the `create.js.erb` template in the line items view folder:
{% highlight javascript %}
if ($('#cart tr').length == 1) { $('#cart').show('blind', 1000); }
{% endhighlight %}

**15.** Remove the message about currently empty carts (from where?).
It is no longer needed, and it no longer works correctly.

**15a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.

**16. Testing AJAX.** If you run the tests now (try it: `rake test`),
they'll fail badly.  For two reasons: we did not design the latest improvements
with the tests in mind, and we did not reflect some modifications
in the tests.  To adjust this, replace the
`<%= hidden_div_if ... %>` in the layout by
{% highlight html+ruby %}
<% if @cart %>
  <%= hidden_div_if(@cart.line_items.empty?, id: 'cart') do %>
    <%= render @cart %>
  <% end %>
<% end %>
{% endhighlight %}
and then run the test again.

**17.**  The only test that still fails is about a page redirect.  In the
line items controller test, change the corresponding line to
{% highlight ruby %}
assert_redirected_to store_path
{% endhighlight %}

**18.** To test the AJAX functionality, add the following test to the line items controller test.
{% highlight ruby %}
test "should create line item via ajax" do
  assert_difference('LineItem.count') do
    xhr :post, :create, :product_id => products(:one).id
  end
  assert_response :success
end
{% endhighlight %}

**18a.** If all works fine, commit the changes to your local `git`
repository, and push them to the `github` cloud.
