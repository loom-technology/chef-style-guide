# Chef Style Guide

[![Chef Style Guide](https://img.shields.io/badge/Chef%20Style%20Guide-v0.3.0-blue.svg)](https://github.com/loom-technology/chef-style-guide)

Follow this style guide to write more readable code with fewer errors.

## 1. Metadata

### 1.1. Use a single line per cookbook dependency

Use a single line for each cookbook dependency, especially if you have many dependencies.

### 1.2 Use exact or pessimistic versioning

Specify versions with [pessimistic versioning constraints] or exact version constraints.

[pessimistic versioning constraints]: http://guides.rubygems.org/patterns/#pessimistic-version-constraint

### 1.3 Discussion

Don't use percent strings or iterators to declare dependencies, since you can't specify cookbook version constraints.

Using the preferred style, changes in GitHub are easier to read. Instead of having one or more dependency changes per line, GitHub will only show one change per line.

Percent string arrays can grow very long, creating long lines of code. Lines should be short since GitHub cuts off lines that are longer than 120 characters, decreasing readability.

Multiline percent strings also don't allow authors to specify cookbook version constraints.

The `%w(…)` [percent string syntax] may be difficult to understand if you or another developer are new to Ruby. Don't assume a high level of familiarity with Ruby.

[percent string syntax]: http://www.ruby-doc.org/core-2.2.0/doc/syntax/literals_rdoc.html#label-Percent+Strings

##### Incorrect

```ruby
%w(apt git jenkins …).each do |cookbook|
  depends cookbook
end

%w(apt
   git
   jenkins …).each do |cookbook|
     depends cookbook
end
```

##### Correct

```ruby
depends apt '4.0.2'
depends git, '~> 5.0'
depends jenkins '2.6.0'
…
```

## 2. Cookbook names

### 2.1 Use underscores and lowercase letters to name cookbooks

#### Discussion

The cookbook name is the GitHub repository name. Use lowercase names with underscores separating words. This is called "[snake case]."

Since Chef is written in Ruby, do not use characters that are invalid for Ruby methods. In some versions of Ruby, hyphens are invalid characters for symbol names—e.g. `:this-is-invalid` but `:"this-is-valid"` in Ruby >= 2.0.0. Do not use hyphens in cookbook names.

Keeping cookbooks in their own GitHub organization helps to separate concerns. This is analogous to achieving [first normal form] in a relational database. For example, this document is in the GitHub organization `loom-technology`, but Loom cookbooks are stored in the GitHub organization `loom-cookbooks`.

It may also be desirable to prefix cookbooks names with your an abbreviation of your organization—e.g. `myorg_logstash`. This is not an uncommon pattern and can help clarify which repository is which if you have a cluttered namespace— lots of things that have similar names.

[snake case]: https://en.wikipedia.org/wiki/Snake_case
[first normal form]: https://en.wikipedia.org/wiki/First_normal_form

##### Incorrect

```ruby
logstash_cookbook
logstash-cookbook
long-cookbook-name
```

##### Correct

```ruby
logstash
long_cookbook_name
myorg_logstash
```

## 3. Attributes

### 3.1 Use strings to access node attribute keys

Use the style, `node['with']['single']['quotes']`.

#### Discussion

For more background see this post, [Remove contentious rule FC001].

Each node attribute is a [Mash] (defined in [lib/chef/mash.rb]). Using Mash can lead to problems. A valid node attribute is not necessarily a valid Ruby symbol or a valid Ruby method—so just skip the Mash magic and stick to strings.

[Remove contentious rule FC001]: https://github.com/acrmp/foodcritic/issues/86
[Mash]: http://www.rubydoc.info/gems/chef/Mash
[lib/chef/mash.rb]: https://github.com/chef/chef/blob/master/lib/chef/mash.rb

##### Correct

```ruby
node['with']['single']['quotes']
```

It's acceptable to use interpolation or variables in node attribute keys if it's absolutely required.

```ruby
node["maybe_with_#{interpolation}_but_only_if_necessary"]

some_fancy_key = "may_be_clearer_to_#{interpolate}_here"
node[some_fancy_key]
```

##### Incorrect

The Mash class allows you to call node attributes in the following bad ways.

```ruby
node["with"]["any"]["double"]["quotes"]
node.with.any.accessor.calls
node[:with][:normal][:symbols]
node[:"with"][:"string"][:"literal"][:"symbols"]
```

### 3.2 Use #tap to access nested attributes

If you are getting or setting many node attributes at the same nested level, use Ruby's [#tap] method to simplify your code.

[#tap]: http://ruby-doc.org/core-2.3.0/Object.html#method-i-tap

#### Discussion

Prefer using #tap if you have long lines.

Using #tap will improve readability if you have long lines caused by lots of nested attributes. Since lots of nested attributes can make your code bloat out with lots of long lines, it will improve the readability of such code to use #tap.

Long lines are cut off in GitHub. This will improve the readability of your attribute settings.

##### Incorrect

This line is too long.

```ruby
default['an_application']['settings']['very_long_set']['more_longer_attributes']['omg']['even_more_configuration']
```

##### Correct

You can read the attributes without scrolling horizontally.

```ruby
default['an_application']['settings']['very_long_set'].tap do |very_long_set|
  very_long_set['more_longer_attributes']['omg'].tap do |omg|
    do_something_with omg['even_more_configuration']
  end
end
```

##### Incorrect

In this example, all attribute keys are nested below `['java']`.

```ruby
default['java']['install_flavor'] = 'oracle'
default['java']['jdk_version'] = '8'
default['java']['oracle']['accept_oracle_download_terms'] = true
…
```

##### Correct

```ruby
default['java'].tap do |java|
  java['install_flavor'] = 'oracle'
  java['jdk_version'] = '8'
  java['oracle']['accept_oracle_download_terms'] = true
  …
end
```

## 4. Templates

### 4.1 Do not use Ruby logic in ERB templates

Do not use Ruby logic in ERB templates. Instead, use the `variables` property when using Chef's `template` resource.

#### Discussion

When using the Chef `template` resource, do not put logic in your ERB (Embedded Ruby) template files.

The Chef `template` resource has an optional `variables` property, which takes a hash. The keys of the hash are used to create variables that are made available in the scope of the ERB template.

Perform all necessary logic in your recipes, custom resource actions, or helper methods to create a hash. Then pass the hash to the `variables` property on your `template` resource call.

Separating logic from templates allows you to isolate concerns. For example, you may want to test a helper method to verify that it provides the correct logical value to a template.

##### Incorrect

In the following scenario, the code uses logic in an ERB template to determine the contents of a hypothetical configuration file.

In the hypothetical configuration file, `an_important_config_key` is set to a value determined by simple Ruby logic within the configuration file's ERB template.

In an ERB template:

```erb
<% if node['some_key'] == 'some value' -%>
an_important_config_key "<%= node['some_other_key'] %>"
<% else -%>
an_important_config_key "<%= node['some_third_key'] %>"
<% end -%>
…
```

##### Correct

In the following scenario, no logic is contained in the ERB template.

In a Chef recipe or custom resource action:

```ruby
config_value = # do some logic and set a value, possibly with a helper method
some_template_variables = { an_important_config_value: config_value }
```

Then, in the ERB file:
```erb
an_important_config_key "<%= an_important_config_value %>"
…
```

Pass the variables in your call on `template` resource:
```ruby
template '/etc/motd' do
  variables(some_template_variables)
end
```

# 5. Improve your experience

Here are some notes you can refer to. They will help make the best of your experience in designing and building things with Chef.

## Ruby

Understanding Ruby is helpful. Encourage your team to be opinionated and to agree on style and convention.

Increasing readability will reduce software defects. Follow [The Ruby Style Guide].

[The Ruby Style Guide]: https://github.com/bbatsov/ruby-style-guide

### Less well-known Ruby features you should use

Ruby [keyword arguments] are a step forward in the endless march against unnecessary punctuation. Use keyword arguments to make method definitions and method calls easier to understand.

```ruby
def gather_arguments(first: nil, **rest)
  p first, rest
end
```

Note: The following call does not need parentheses!

```ruby
gather_arguments first: 1, second: 2, third: 3
# prints 1 then {:second=>2, :third=>3}
```

This now has the improved clarity of explicitly naming the parameters—the reader knows which value is assigned to which variable being sent to the method.

Additionally, it is less visually cluttered since parenthesis are not required.

[keyword arguments]: http://ruby-doc.org/core-2.1.8/doc/syntax/methods_rdoc.html#label-Keyword+Arguments

### Ruby style caveats

Understand that there are certain recommendations in The Ruby Style Guide that may affect the logical operation of your software.

#### Logical order of precedence

For example, `and` and `&&` have a different [order of precedence]. Purposefully following an order of precedence may be functionally significant and more important than following the style guide.

```ruby
puts 'hi' and 'goodbye'
hi
=> nil

puts 'hi' && 'goodbye'
goodbye
=> nil

'hi' && 'goodbye'
=> "goodbye"

'hi' and 'goodbye'
=> "goodbye"
```

[order of precedence]: http://ruby-doc.org/core-2.2.0/doc/syntax/precedence_rdoc.html

#### Symbols

Here are a few notes about [symbols in Ruby]. Most of this section is straight from the Ruby documentation.

Understanding Ruby symbols—how they work, how to declare them, and so on—will simplify the inevitable discussions you will have about them.

[symbols in Ruby]: https://ruby-doc.org/core-2.3.0/doc/syntax/literals_rdoc.html#label-Symbols

##### String literal symbols

You _can_ use _string literal symbols_ and _symbols by interpolation_. (Ruby >= 2.0.0.)

```ruby
:"my_symbol1"
:"my_symbol#{1 + 1}"
```

However, this style decreases readability. Knowing that you _can_ do this will put to rest the discussion that symbols are immutable strings, or that their declaration style is exclusive to Ruby's `String`.

-

##### Symbols and speed

Ruby 2.3 introduced immutable strings, which you can enable by setting an option. Ruby 3.0 will by default enable immutable strings.

With immutable strings, you can specify the value of a string `'a'` and another string `'a'` and they will reference the same `'a'`. This saves memory and makes the code significantly more performant. You can read a [discussion on immutable string literals].

So, are symbols faster than strings? Not really!

[immutable string literals]: https://bugs.ruby-lang.org/issues/11473

## Testing

Tests are an essential part of any team's workflow. Don't let the notion of testing intimidate you! Try starting with integration tests, and then add unit tests later after you've "tuned" your code.

Then, when you perform tests, run your unit tests first. Unit tests are faster, which allows you to fail faster, which is good!

Chef runs in two phases: Compile, and execute.

- Unit tests with ChefSpec verify code at compile time.
- Integration tests with InSpec verify machine configuration at execute time.

### Unit testing with ChefSpec

Unit tests verify Chef's compile phase. Unit tests verify that certain Chef resources were called with certain properties. Unit tests are used to ensure that the code you've written is going to do what you thought it was going to do.

Since unit tests run before the Chef execute phase, unit tests perform quickly, allowing you to fail faster.

Unit tests are written in [ChefSpec].

Take a look at my example cookbooks for reference on unit testing.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

[ChefSpec]: https://github.com/sethvargo/chefspec

### Integration testing with InSpec

Integration tests run against a converged virtual machine. Use integration tests to verify that Chef recipes configure a virtual machine to a desired state.

Integration tests are written in [InSpec].

Take a look at my example cookbooks for reference on integration testing.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

[InSpec]: https://github.com/chef/inspec

### Betterspecs.org

[Betterspecs.org] will help you learn core concepts about testing.

[Beterspecs.org]: http://betterspecs.org/

#### Caveats

Some concepts at Betterspecs.org don't translate exactly to InSpec—for example, Betterspecs.org recommends using "expect" syntax instead of "should" syntax. However, if you're writing tests in InSpec, you *should* use "should" syntax since that's how it was designed to be used.

## Cookbook design

I emphasize good cookbook design. Chef is incredibly flexible, but enforcing design choices will increase readability, decrease defects and result in better code that is easier to maintain.

### Resource cookbooks

[You'll see the phrase "don't repeat yourself" fairly often, and you'll see the acronym DRY][dry].

Many cookbooks are recipe-centric, and contain a set of recipes. These recipes are added to a run list, and execute linearly interpreted Ruby instructions, there are also plenty of design patterns you can put to use. One I like a lot is what I'm calling the "resource cookbook" pattern.

C calls them libraries, Python calls them packages, and Ruby calls them gems. With Chef, a similar notion is to author **resource cookbooks**. Resource cookbooks define core, reusable functionality in the form of Chef resources to configure a machine in a specific way.

After writing a resource cookbook, you can then write a "wrapper cookbook" which consumes the resource cookbook. This allows for separation of logical concerns, and isolated testing—this results in safer code with fewer defects.

Take a look at my example cookbooks for reference on wrapping resource cookbooks and creating custom resources.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself

# Author

Created and maintained by [Kevin J. Dickerson][kevin]. <kevin.dickerson@loom.technology>

# Acknowledgement

Many of the thoughts here are a result of discussions with my friend and colleague Aaron Lane ([@aaron-lane], [@ncs-alane]) :maple_leaf:.

[@aaron-lane]: https://github.com/aaron-lane
[@ncs-alane]: https://github.com/ncs-alane

# References

Look at the [RAW view of this document][raw] to see the references and links in this document.

# Changes

This guide follows [Semantic Versioning 2.0].

[Semantic Versioning 2.0]: http://semver.org/

## 0.3.0 (09/25-2016)

- Adds section on Ruby symbols
- Adds additional clarification on several topics
- Updates formatting
- Updates several typos
- Refactors named Markdown links

## 0.2.1 (04/16/2016)

- Updates URL for GitHub

## 0.2.0 (03/30/2016)

- Expands and enumerates every section
- Adds section on testing
- Adds section on templates
- Adds various subheadings to improve organization
- Updates code examples
- Refactors markdown links to use named links
- Adds version shield
- Adds references

## 0.1.0 (03/21/2016)

- Begins versioning

[kevin]: https://loom.technology
[example]: https://supermarket.chef.io/cookbooks/example
[example_resources]: https://supermarket.chef.io/cookbooks/example_resources
[raw]: https://raw.githubusercontent.com/kevindickerson/chef-style-guide/master/README.md
