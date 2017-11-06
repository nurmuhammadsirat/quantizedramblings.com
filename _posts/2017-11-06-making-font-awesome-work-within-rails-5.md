---
title: Making Font Awesome Work Within Rails 5
asset_path: /assets/making-font-awesome-work-within-rails-5
updated: 2017-11-06 14:02
---

The easiest way to get Font Awesome to work is to use [the CDN itself](http://fontawesome.io/get-started/), which requires you to give them your email address.

But there may be times when you want to host it yourself. You can either use a [gem](https://github.com/bokmann/font-awesome-rails), or if you prefer to have more control, you can [download the zip from Font Awesome directly](fontawesome.io/get-started/).

If you did the latter:

* Unzip the file and you'll see a bunch of folders.
* Select the `font-awesome.min.css` in the `css` folder and copy it to `vendor/assets/stylesheets`.
* Copy the folder `fonts` to `vendor/assets`.
* Open your `application.css` and enter the line `*= require font-awesome.min` somewhere between the other require statements. There is no dependency for Font Awesome, but if your other css files depend on this, ensure that the Font Awesome's require line is higher.
* Next, open `font-awesome.min` in your favourite text editor.
* Replace all instances of `url('../fonts/...)` to `url('/assets/...)`.

Restart your Rails server if needed. This should now work and you should no longer see any more RoutingError in your logs when the application is trying to find the font file.
