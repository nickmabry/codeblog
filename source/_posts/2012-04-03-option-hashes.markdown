---
layout: post
title: "Rails redirect_to Hashes"
date: 2012-04-03 19:40
comments: true
categories: code ruby rails
---
I ran across a surprise when using a Rails method a few days ago:
``` ruby rails/actionpack/lib/action_controller/metal/flash.rb
def redirect_to(options = {}, response_status_and_flash = {}) #:doc:
  if alert = response_status_and_flash.delete(:alert)
    flash[:alert] = alert
  end

  if notice = response_status_and_flash.delete(:notice)
    flash[:notice] = notice
  end

  if other_flashes = response_status_and_flash.delete(:flash)
    flash.update(other_flashes)
  end

  super(options, response_status_and_flash)
end
```
The `redirect_to` method accepts two parameters. The first is the `options` object, which will be itself an options Hash, a URL String, or the symbol `:back`. The second is the `response_status_and_flash` object, which may either be omitted or may be a Hash containing a `status` key and optional `:alert`, `:notice`, and `:flash` keys.

An `options` hash may contain a `:status` key which will be used in lieu of the `response_status_and_flash` hash `:status` if present, but it will not supercede the flash keys. Here are some valid calls:

``` ruby Valid redirect_to Calls
# Redirect to url with status
redirect_to post_url(@post), status: 301

# Redirect to url with status and flash
redirect_to post_url(@post), status: 301, notice: "We've moved forever!"
redirect_to post_url(@post), notice: "We're temporarily moved", status: 302

# Redirect to controller action with status
redirect_to action: 'atom', status: 301

# Redirect to controller action with status and flash
redirect_to({action: 'atom', status: 301}, notice: "We've moved forever!")
redirect_to({action: 'atom'}, notice: "We're temporarily moved", status: 302)
```

Did the last two surprise you? When `options` is a Hash and you would like to set the response flash, the call must be constructed with
parentheses and curly braces to correctly pack in all of the arguments. It took a little experimentation and some helpful folks on IRC for me to tease out the correct syntax. And if you accidentally place a flash option inside of the `options` hash, then the call will succeed with an additional query parameter in the redirect url.

I'm sure I'll mess these up again at some point, so I'll just leave this here for future reference.
