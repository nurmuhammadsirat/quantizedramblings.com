---
title: Same name for Module and Classes
asset_path: /assets/same-name-for-module-and-classes
updated: 2017-03-28 12:59
---

Yes, it is possible. However, you can't define both a class and a module by the same name. Observe:

```
irb(main):001:0> class Foo; end
=> nil

irb(main):002:0> module Foo
irb(main):003:1> class Bar; end
irb(main):004:1> end
TypeError: Foo is not a module
```

Module calledd `Foo` can't be created since it is already a class. But if you really want `Bar` to be namespaced under `Foo`, then:

```
irb(main):005:0> class Foo
irb(main):006:1> class Bar; end
irb(main):007:1> end
=> nil

irb(main):009:0> Foo
=> Foo

irb(main):010:0> Foo::Bar
=> Foo::Bar
```

So when should we define a class or a module as a namespace? Easy rule: if it needs to be instantiated, define a class. If not, a module should suffice.
