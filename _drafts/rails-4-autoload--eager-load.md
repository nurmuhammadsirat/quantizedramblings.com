---
title: Rails 4: autoload & eager load
asset_path: /assets/rails-4-autoload--eager-load
updated: 2017-03-28 12:58
---

In development and test environment config files, you'll notice that `config.eager_load` is set to false, but true in production. `config.autoload_paths` is not explicitly set, so I guess it will be to whatever Rails has set to default. I never really gave these a second thought, until I ran into some problems with it.

In my Rails application, I saw that there is a module being defined as such:

```
module Bar
  def some_method_1
  end

  def some_method_2
  end
end
```

This file is currently saved in app/services/foo/bar.rb. Looks ok, no? But when I tried to access it in the console, I got:

```
pry(main)> Bar
NameError: uninitialized constant Bar
```

But when I set `config.eager_load = true`...

```
pry(main)> Bar
=> Bar
```

Now when I keep `config.eager_load = false` but changed the module to:

```
module Foo
  module Bar
    def some_method_1
    end

    def some_method_2
    end
  end
end
```

Now,

```
pry(main)> Foo::Bar
=> Foo::Bar
```

What is happening here?

It turns out that Rails will load all files in app/*/*.rb _but only at the first level_. Rails will look at the namespace and class that is required and try to load the appropriate file. So if you were to attempt to access a class called `Post::Comment::Image`, the Rails loading algorithm will look for it in app/*/post/comment/image.rb.

If `config.eager_load` is set to true, then this does not matter as Rails will attempt to load every file in `config.autoload_paths` recursively into the nested directories. But in the spirit of _convention over configuration_, let's properly place our files where the namespace claims it to be.

(To be honest, all of this has been documented in [http://guides.rubyonrails.org/autoloading_and_reloading_constants.html#autoload-paths](http://guides.rubyonrails.org/autoloading_and_reloading_constants.html#autoload-paths). I'm just regurgitating it here for my own sake.)
