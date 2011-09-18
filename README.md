# Abstract

    ... nearly everybody is convinced that every style but their own is
    ugly and unreadable. Leave out the "but their own" and they're
    probably right...
    --Jerry Coffin (on indentation)

This document was created when I, as the Technical Lead of the company
which I work for, was asked by the CTO to create some internal documents
describing good style and best practices for Ruby programming. I started off
by building upon
[this existing style guide](https://github.com/chneukirchen/styleguide),
since I concurred with most of the points in it. I hope it will be useful to
other people as well and I hope that I'll get a lot of feedback and
suggestions on how to improve the guide for the benefit of the entire Ruby
community.

## Formatting

* Use UTF-8 as the source file encoding.
* Use two space indent, no tabs. (Your editor/IDE should have a setting to
  help you with that.)
* Use Unix-style line endings. (Linux/OSX users are covered by default,
  Windows users have to be extra careful.)
    * If you're using Git you might want to do `$ git config --global
      core.autocrlf true` to protect your project from Windows line endings
      creeping into your project.
* Use spaces around operators, after commas, colons and semicolons, around {
  and before } (Exception: Interpolation).

    ```Ruby
    sum = 1 + 2
    a, b = 1, 2
    1 > 2 ? true : false; puts "Hi"
    [1, 2, 3].each { |e| puts e }
    "Hello #{name}!"
    ```

* No spaces after (, [ or before ], ).

    ```Ruby
    some(arg).other
    [1, 2, 3].length
    ```

* Indent `when` as deep as `case`. (As suggested in the Pickaxe.)

    ```Ruby
    case
    when song.name == "Misty"
      puts "Not again!"
    when song.duration > 120
      puts "Too long!"
    when Time.now.hour > 21
      puts "It's too late"
    else
      song.play
    end

    kind = case year
           when 1850..1889 then "Blues"
           when 1890..1909 then "Ragtime"
           when 1910..1929 then "New Orleans Jazz"
           when 1930..1939 then "Swing"
           when 1940..1950 then "Bebop"
           else "Jazz"
           end
    ```

* Use one empty line between `def`s. 

    ```Ruby
    def some_method
      do_something
      do_something_else
      result
    end

    def some_method
      result
    end
    ```
    
* Use RDoc and its conventions for API documentation. (Really RDoc?) Don't put an
  empty line between the comment block and the `def`.
* If you feel you should partition parts of your method with empty lines, consider splitting it into smaller methods before doing so.
* Do not pollute the file with trailing whitespaces.

## Syntax

* Use `def` with parentheses when there are arguments. Omit the
  parentheses when the method doesn't accept any arguments.

     ```Ruby
     def some_method
       # body omitted
     end

     def some_method_with_arguments(arg1, arg2)
       # body omitted
     end
     ```

* Never use `for`, unless you know exactly why. Most of the time iterators
  should be used instead. (Why?)

    ```Ruby        
    arr = [1, 2, 3]
        
    # bad
    for elem in arr do
      puts elem
    end

    # good
    arr.each { |elem| puts elem }
    ```
    
* Never use `then` for multiline `if/unless`.

    ```Ruby
    # bad
    if some_condition then
      # body omitted
    end

    # good
    if some_condition
      # body omitted
    end
    ```

* Use `&&/||` for boolean expressions, `and/or` for control flow. The latter pair has low operator precedence

    ```Ruby
    # boolean expression
    if some_condition && some_other_condition
      do_something
    end

    # control flow
    document.saved? or document.save!
    ```

* Avoid multiline ?: (the ternary operator), use `if/unless` instead.
* Favor modifier `if/unless` usage when you have a single-line body.

    ```Ruby
    # bad
    if some_condition
      do_something
    end

    # good
    do_something if some_condition
    ```

* Favor `unless` over `if` for negative conditions.
       
    ```Ruby
    # bad
    do_something if !some_condition

    # good
    do_something unless some_condition
    ```
    
* Favor `unless` over `if` in order to express edge case character (if applicable)

    # bad (if missing address is edge case)
    process address if address.present?
    
    # good (if missing address is edge case)
    process address unless address.blank?
    
* Favor `unless` over `if` in order to focus on important operations instead of edge case treatment.
  If edge case treatment is as simple as raising an Exception or returning something, do it directly.
       
    ```Ruby
    # bad
    if edge_case
      treat edge_case
    else
      process
    end

    # good
    unless edge_case
      process
    else
      treat_edge_case
    end
    
    # good alternative 1
    raise Exception if unprocessable
    process
    
    # good alternative 2
    return nil if unprocessable
    process
    ```
    
* Suppress superfluous parentheses when calling methods, but keep them
  when calling "functions", i.e. when you use the return value in the
  same line.
  
    ```Ruby
    x = Math.sin(y)
    array.delete e
    ```
        
* Prefer {...} over do...end for single-line blocks.
  Avoid using {...} for multi-line blocks.
  Always use do...end for "control flow" and "method definitions" (e.g. in Rakefiles and certain DSLs).
  Avoid do...end when chaining.

* Avoid `return` where not required.

    ```Ruby
    # bad
    def some_method(some_arr)
      return some_arr.size
    end

    # good
    def some_method(some_arr)
      some_arr.size
    end
    ```

* Avoid line continuation (\\).

* Using the return value of = is ok.

    ```Ruby
    if v = array.grep(/foo/) ...
    ```

* Use ||= freely.

    ```Ruby
    # set name to Bozhidar, only if it's nil or false
    name ||= "Bozhidar"
    ```

* Make use of blank?, present? and presence.

    ```Ruby
    # bad
    unless address.nil? || address.empty?
    
    # good
    unless address.blank?
    if address.present?
    
    # bad
    state   = params[:state]   if params[:state].present?
    country = params[:country] if params[:country].present?
    region  = state || country || 'US'

    # good
    region = params[:state].presence || params[:country].presence || 'US'
    ```

* Avoid using Perl-style global variables (like $0-9, $`, ...).

* Never put a space between a method name and the opening parenthesis.

    ```Ruby
    # bad
    f (3 + 2) + 1

    # good
    f(3 + 2) + 1
    ```

## Naming

* Use `snake_case` for methods and variables.
* Use `CamelCase` for classes and modules.  (Keep acronyms like HTTP,
  RFC, XML uppercase.)
* Use `SCREAMING_SNAKE_CASE` for other constants.
* The names of predicate methods (methods that return a boolean value)
  should end in question mark.
  (i.e. `Array#empty?`).
* The names of potentially "dangerous" methods (i.e. methods that modify `self` or the
  arguments, `exit!`, etc.) should end with exclamation marks.

* When defining binary operators, name the argument `other`.

    ```Ruby
    def +(other)
      # body omitted
    end
    ```

* Prefer `collect` over `map` if you are just collecting attributes without transformations

    ```Ruby
    # good
    user_names = users.collect { |u| u.name }
    downcased_user_names = user_names.map { |n| n.downcase }
    ```

## Comments

* Write self documenting code and ignore the rest of this section. _"Good
  code is its own best documentation. As you're about to add a comment, ask
  yourself, ‘How can I improve the code so that this comment isn't needed?’
  Improve the code and then document it to make it even clearer."_ (Steve
  McConnell)
* If your comment is a sentence, start it capitalized and use punctuation.
* Avoid superfluous comments.

    ```Ruby
    # bad
    counter += 1 # increments counter by one
    ```

* Keep existing comments up-to-date. No comment is better than an outdated
  comment.
* Avoid writing comments to explain bad code. Try to refactor the code to
  make it self-explanatory.

## Classes

* Always supply a proper `to_s` method.
* Use the `attr` family of functions to define trivial accessors or
  mutators.
* Consider adding factory methods to provide additional sensible ways
  to create instances of a particular class.
* Prefer duck-typing over inheritance.
* Avoid the usage of class (@@) variables due to their "nasty" behavior.
* Assign methods proper visibility levels (`private`, `protected`)
in accordance with their intended usage. Don't go off leaving
everything `public` (which is the default).

## Exceptions

* Don't suppress exceptions without sending out a Airbrake notification manually.
* Avoid rescuing more than you expect to fail (e.g. begin .. rescue - without a specific Exception class)

## Misc

* Avoid methods longer than 10 LOC (lines of code). Ideally most methods will be shorter than
  5 LOC.
* Try to keep the argument list small. Consider the use of opts = {} as last parameter to include further parameters as options.
* Use `def self.method` to define singleton methods. This makes the methods
  more resistant to refactoring changes.

    ```Ruby
    class TestClass
      # bad
      def TestClass.some_method
        # body omitted
      end

      # good
      def self.some_other_method
        # body omitted
      end
    end
    ```

* Avoid `alias` when `alias_method` will do.
* Write for Ruby 1.9. Don't use legacy Ruby 1.8 constructs.
    * Use the new JavaScript literal hash syntax.
    * Use the new lambda syntax.
    * Methods like `inject` now accept methods names as arguments.
    
      ```Ruby
      [1, 2, 3].inject(:+)
      ```

* Avoid needless metaprogramming.

## Design

* Code in a functional way, avoiding mutation when it makes sense.
* Do not mutate arguments unless that is the purpose of the method.
* Do not mess around in core classes when writing libraries. (Do not monkey
  patch them.)
* [Do not program
  defensively.](http://www.erlang.se/doc/programming_rules.shtml#HDR11)
* Keep the code simple and subjective. Each method should have a single,
  well-defined responsibility.
* Avoid more than three levels of block nesting.
* Don't overdesign. Overly complex solutions tend to be brittle and hard to
  maintain.
* Don't underdesign. A solution to a problem should be as simple as
  possible, but no simpler than that. Poor initial design can lead to a lot
  of problems in the future.
* Be consistent. In an ideal world, be consistent with these guidelines.
* Use common sense.

# Contributing

Feel free to open tickets or send pull requests with improvements.
