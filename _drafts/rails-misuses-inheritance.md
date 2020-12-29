---
layout: post
title: Rails Views are Inadequate
description: ""
categories:
- Rails
---

Rails Views are inadequate

Rails views do not subscribe to the "view" definition of MVC, so here's what I mean: they're the objects whose bindings are used for template renderings.
In the following code, the view is the object which responds to the `hello` method, and has the `@world` instance variable.

```erb
<html>
  <%= hello(@world) %>
</html>
```

### Elements of the View Context

Views in Rails, called [`view_context_class`](https://github.com/rails/rails/blob/d36991147863a95bea93506dc5411d00043790f5/actionview/lib/action_view/rendering.rb#L64),
are composed of different parts, all defined by the controllers that uses them. 
The first and most common part are the instance variables. When a controller [instantiates a view context](https://github.com/rails/rails/blob/d36991147863a95bea93506dc5411d00043790f5/actionview/lib/action_view/rendering.rb#L92), 
it creates a [hash of its instance variables](https://github.com/rails/rails/blob/b2eb1d1c55a59fee1e6c4cba7030d8ceb524267c/actionpack/lib/abstract_controller/rendering.rb#L66-L71), which are then [bound as instance variables on the view context](https://github.com/rails/rails/blob/d36991147863a95bea93506dc5411d00043790f5/actionview/lib/action_view/base.rb#L205). 
Roughly speaking, the following code illustrates what's going on:

```ruby
class GreetingView < ActionView::Base
  def initialize(variables)
    variables.each { |name, value| instance_variable_set("@#{name}", value) }
  end
end

class GreetingController < ActionController::Base
  def view_context
    variables =
     instance_variables.each_with_object({}) do |name, hash| 
       hash[name] = instance_variable_get(name) 
     end
     
    View.new(variables)
  end
end
```

Instance Variables are the prescribed way to share state from the Controller to the View Context. 

The second part are view helpers, called with the `helper` singleton method on controllers:

```ruby
module GreetingHelper
  def hello(name)
    "Hello #{name}" 
  end
end

class GreetingController < ActionController::Base
  helper(GreetingHelper)
end
```

View helpers are modules that get [included by the View Context Classes](https://github.com/rails/rails/blob/d36991147863a95bea93506dc5411d00043790f5/actionview/lib/action_view/rendering.rb#L59).
Roughly speaking, this is what happens:

```ruby
class GreetingController < ActionController::Base
  def self.helper(mod)
    GreetingView.include(mod)
  end
end
```

Finally, there are Helper Methods, used by calling `helper_method` on the controller like so:

```ruby
class GreetingController < ActionController::Base
  helper_method(:name)
  
  def name
    @name
  end
end
```

[Helper Methods](https://github.com/rails/rails/blob/b13a5cb83ea00d6a3d71320fd276ca21049c2544/actionpack/lib/abstract_controller/helpers.rb#L60-L71) are useful when the controller has a method which the View Context requires. 
Defining a Helper Method will define the method of the same name on the View Context Class, delegating to the controller.

### Complexity-

- before|after|around action
- code sharing via modules

### Views & Helpers

Views are another area which


