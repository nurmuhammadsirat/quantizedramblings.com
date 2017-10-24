---
title: Writing (Good) Tests With RSpec
asset_path: /assets/writing-good-tests-with-rspec
updated: 2017-04-01 23:24
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

        it 'is expected not to change an immutable input field' do
          expect(form.person.name).to eq('Jill Oldname')
        end
      end
    end
  end
end
```

Let's look at the test. The way I break it down is:

```Ruby
it 'is expected not to change an immutable input field' do   # Description of the test
  expect(form.person.name).to eq('Jill Oldname')             # Actual test
end
```

The actual test consists of just one line. Whats `form`? Why does it have a `person`? Why is that person's name supposed to be "Jill Oldname"? The test code may be DRY and concise, but it conveys very little inside it. We have to jump around all over the place to figure out what the test is trying to do and trying to convey.

