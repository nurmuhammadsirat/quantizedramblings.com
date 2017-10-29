---
title: Writing (Good) Tests With RSpec
asset_path: /assets/writing-good-tests-with-rspec
updated: 2017-10-10 23:24
---

Writing tests are a given. We are so overwhelmed and inuadated with people telling us to write tests for this and tests for that. Heck, the current development craze is to even let the [development and design be driven by tests](https://en.wikipedia.org/wiki/Test-driven_development). We depend on it so much that it has gotten to the point where we are fearful of removing tests, even though it may be repeated or unnecessary. Well, surely, too many tests are better than too little.

But how different are writing tests with writing code?

Sometimes I pick up a task where I have to depend on the tests available to decide how to proceed. A typical test in RSpec will look like this:

```Ruby
RSpec.describe Form do
  context '.create_with' do
    let(:params) {
      # A huge param hash here...
    }

    context 'an inner context' do
      let(:params) {
        # Another huge hash...
      }

      # ...
      # 100 lines of stuff
      # ...

      context '2nd leve inner context' do
        let(:name) { 'John Newname' }
        # ...
        # Another 100 lines of tests...
        # ...

        it 'is expected not to change the name during when updating' do
          expect(form.person.name).to eq('Jill Oldname')
        end
      end
    end
  end
end
```

Let's zoom in on the test. The way I break it down is:

```Ruby
it 'is expected not to change the name during when updating' do   # Description of the test
  expect(form.person.name).to eq('Jill Oldname')             # Actual test
end
```

The actual test consists of just one line. Whats `form`? Why does it have a `person`? Why is that person's name supposed to be `Jill Oldname`? The test code may be DRY and concise, but it conveys very little inside it. We have to jump around all over the place to figure out what the test is trying to do and trying to convey. From the description, it seems that something has been done to the form, but the name is not supposed to be changed. But hey, that's just the description. It's nothing more than a comment.

And do you trust code comments?

!["Code Comments"]({{ page.asset_path }}/code-comments.jpg)

So when writing specs, I make it a point to prioritize **readability** over **DRY-ness**.

```Ruby
it 'is expected not to change the name during when updating' do
  # Data setup
  form = Form.create( id: 10, name: 'Jill Oldname' )
  form_input_params = { id: 10, name: 'John Newname', ... }

  # Action Performed
  post :update, form_input_params

  # Assertion
  expect(form.person.name).to eq('Jill Oldname')
end
```

It becomes slightly clearer now. It seems that there is already some type of `form` object already in the database, and an update to the same form should not have changed the name.

While this is much more verbose, and probably much less DRY, it conveys a lot more in a single glance. Debugging becomes much easier, we have no qualms to remove it if it becomes unnecessary, and we gain a lot more confidence in manipulating our test strategies.

Some have argued to me that this consumes more memory, since we don't take advantage of memoization. It could also potentially increase the overall unit test time. Well, nothing is perfect. We have to expect trade-offs when we prioritize one over the other. However, the gains in confidence in your test code cannot be understated here. Using memoization techniques from rspec such as `let` or `subject` will let us be more concise in our tests and works if the test code is less that 2 pages long. Once it gets overly nested and longer, debugging it becomes a huge pain and our confidence in changing the test code may drop.
