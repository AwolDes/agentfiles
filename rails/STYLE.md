# Style

We aim to write code that is a pleasure to read, and we have a lot of opinions about how to do it well. Writing great code is an essential part of our programming culture, and we deliberately set a high bar for every code change anyone contributes. We care about how code reads, how code looks, and how code makes you feel when you read it.

We love discussing code. If you have questions about how to write something, or if you detect some smell you are not quite sure how to solve, please ask away to other programmers. A Pull Request is a great way to do this.

When writing new code, unless you are very familiar with our approach, try to find similar code elsewhere to look for inspiration.

## Core Philosophy

- **Radical Simplicity**: Every line must justify its existence. Delete aggressively.
- **Idiomatic Over Clever**: Use the language's natural patterns. Flow with the framework, never against it.
- **Self-Documenting Code**: Comments are code smell. Let method names and structure tell the story.
- **DRY to the Extreme**: No duplication ever. Extract shared logic immediately.
- **Programmer Happiness**: Code should spark joy. Write code you'd want to maintain at 2am.

## Ruby/Rails

- Follow Rails conventions: fat models, skinny controllers
- Use scopes, callbacks, and concerns appropriately
- Declarative over imperative: tell what, not how
- Extract complex logic to well-named private methods
- Question all metaprogramming: is it truly necessary?
- Leverage Active Support extensions idiomatically
- Trust Rails' built-in methods - don't reinvent

## Helpers

Helper methods should live in view-specific helper files, not `ApplicationHelper`. The `ApplicationHelper` should remain empty or contain only truly global helpers used across the entire application.

```ruby
# Bad - Adding view-specific logic to ApplicationHelper
module ApplicationHelper
  def gift_swap_status_badge(gift_swap)
    # ...
  end
end

# Good - Using the appropriate view-specific helper
module GiftSwapsHelper
  def gift_swap_status_badge(gift_swap)
    # ...
  end
end
```

This keeps helpers organized, makes their scope clear, and prevents `ApplicationHelper` from becoming a dumping ground for unrelated methods.

## Coding Style & Naming Conventions
Write as though you are DHH shipping code into Rails core: choose the boring, conventional solution, prefer readability over cleverness, and rely on Rails helpers before building abstractions. Ruby follows RuboCop (`bin/rubocop`), two-space indent, snake_case methods, PascalCase classes. Keep models lean; push orchestration into POROs under `app/models` or concerns.

Use tailwind classes in views.

## DHH Mode Checklist
1. Ask “How would Rails solve this today?” before adding gems or custom JS.
2. Pretend future maintainers are Rails core reviewers—ship code they would merge.
3. If a solution feels clever, rewrite it straighter and document any intentional divergence from convention.

## Testing Guidelines

Minitest lives in `test/`; mirror Rails naming such as `accounts_controller_test.rb` and lean on fixtures in `test/fixtures`.

### Use Fixtures, Not Mocks or Stubs

**Always use real ActiveRecord objects from fixtures instead of mocks or stubs like OpenStruct.**

```ruby
# Bad - Using OpenStruct to stub associations
gift_swap.define_singleton_method(:participants) do
  OpenStruct.new(count: 5)
end

# Good - Using fixtures with real ActiveRecord objects
gift_swap = gift_swaps(:christmas_swap)
assert_equal 3, gift_swap.participants.count
```

### Why Fixtures?

1. **Test real behavior**: Fixtures use actual ActiveRecord associations and database queries, catching bugs that mocks miss
2. **Consistency**: Matches the pattern used throughout Rails and this codebase
3. **Maintainability**: Changes to model behavior are automatically tested
4. **Clarity**: No need to understand mocking frameworks - just reference fixtures

### Fixture Best Practices

- Create fixtures for common test scenarios in `test/fixtures/`
- Reuse fixtures across tests when possible
- Use `travel_to` for date-dependent tests
- Follow the patterns in existing model tests (e.g., `test/models/gift_swap_test.rb`)

```ruby
# Example: Testing date-dependent behavior
test "shows exchange_day status on day of exchange" do
  travel_to gift_swaps(:exchange_day_swap).day_of_exchange do
    assert_equal "exchange_day", gift_swaps(:exchange_day_swap).status
  end
end
```

## Conditional returns

In general, we prefer to use expanded conditionals over guard clauses.

```ruby
# Bad
def todos_for_new_group
  ids = params.require(:todolist)[:todo_ids]
  return [] unless ids
  @bucket.recordings.todos.find(ids.split(","))
end

# Good
def todos_for_new_group
  if ids = params.require(:todolist)[:todo_ids]
    @bucket.recordings.todos.find(ids.split(","))
  else
    []
  end
end
```

This is because guard clauses can be hard to read, especially when they are nested.

As an exception, we sometimes use guard clauses to return early from a method:

* When the return is right at the beginning of the method.
* When the main method body is not trivial and involves several lines of code.

```ruby
def after_recorded_as_commit(recording)
  return if recording.parent.was_created?

  if recording.was_created?
    broadcast_new_column(recording)
  else
    broadcast_column_change(recording)
  end
end
```

## Methods ordering

We order methods in classes in the following order:

1. `class` methods
2. `public` methods with `initialize` at the top.
3. `private` methods

## Invocation order

We order methods vertically based on their invocation order. This helps us to understand the flow of the code.

```ruby
class SomeClass
  def some_method
    method_1
    method_2
  end

  private
    def method_1
      method_1_1
      method_1_2
    end
  
    def method_1_1
      # ...
    end
  
    def method_1_2
      # ...
    end
  
    def method_2
      method_2_1
      method_2_2
    end
  
    def method_2_1
      # ...
    end
  
    def method_2_2
      # ...
    end
end
```

## To bang or not to bang

Should I call a method `do_something` or `do_something!`?

As a general rule, we only use `!` for methods that have a correspondent counterpart without `!`. In particular, we don’t use `!` to flag destructive actions. There are plenty of destructive methods in Ruby and Rails that do not end with `!`.

## Visibility modifiers

We don't add a newline under visibility modifiers, and we indent the content under them.

```ruby
class SomeClass
  def some_method
    # ...
  end

  private
    def some_private_method_1
      # ...
    end

    def some_private_method_2
      # ...
    end
end
```

If a module only has private methods, we mark it `private` at the top and add an extra new line after but don't indent.

```ruby
module SomeModule
  private
  
  def some_private_method
    # ...
  end
end
```

## CRUD controllers

We model web endpoints as CRUD operations on resources (REST). When an action doesn't map cleanly to a standard CRUD verb, we introduce a new resource rather than adding custom actions.

```ruby
# Bad
resources :cards do
  post :close
  post :reopen
end

# Good
resources :cards do
  resource :closure
end
```

## Controller and model interactions

In general, we favor a [vanilla Rails](https://dev.37signals.com/vanilla-rails-is-plenty/) approach with thin controllers directly invoking a rich domain model. We don't use services or other artifacts to connect the two.

Invoking plain Active Record operations is totally fine:

```ruby
class Cards::CommentsController < ApplicationController
  def create
    @comment = @card.comments.create!(comment_params)
  end
end
```

For more complex behavior, we prefer clear, intention-revealing model APIs that controllers call directly:

```ruby
class Cards::GoldnessesController < ApplicationController
  def create
    @card.gild
  end
end
```

When justified, it is fine to use services or form objects, but don't treat those as special artifacts:

```ruby
Signup.new(email_address: email_address).create_identity
```

## Run async operations in jobs

As a general rule, we write shallow job classes that delegate the logic itself to domain models:

* We typically use the suffix `_later` to flag methods that enqueue a job.
* A common scenario is having a model class that enqueues a job that, when executed, invokes some method in that same class. In this case, we use the suffix `_now` for the regular synchronous method.

```ruby
module Event::Relaying
  extend ActiveSupport::Concern

  included do
    after_create_commit :relay_later
  end

  def relay_later
    Event::RelayJob.perform_later(self)
  end

  def relay_now
    # ...
  end
end

class Event::RelayJob < ApplicationJob
  def perform(event)
    event.relay_now
  end
end
```

## Quality Gates

**The Rails Core Test**: Would this be accepted into Rails itself?
- Is this exemplar code worthy of documentation?
- Does it demonstrate mastery of the language/framework?
- Does it feel natural within the codebase?

## Red Flags

- Unnecessary complexity or "cleverness"
- Fighting framework conventions
- Non-idiomatic patterns
- Comments explaining what (not why)
- Any duplication
- Configuration when convention would work
- Metaprogramming without clear benefit

## The Standard

Code quality isn't "does it work?" but "is it exemplary?"

The bar is: would this be used as a teaching example?