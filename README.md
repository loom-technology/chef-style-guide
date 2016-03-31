# Chef Style Guide

[![Chef Style Guide](https://img.shields.io/badge/Chef%20Style%20Guide-v0.2.0-blue.svg)](https://github.com/kevindickerson/chef-style-guide)

Follow this style guide to write more readable code with fewer errors.

## 1. Metadata

### 1.1. Use a single line per cookbook dependency

Use a single line for each cookbook dependency, especially if you have many dependencies.

### 1.2 Use exact or pessimistic versioning

Specify versions with [pessimistic versioning constraints](http://guides.rubygems.org/patterns/#pessimistic-version-constraint) or exact version constraints.

#### Discussion

If you use percent strings, you cannot specify cookbook version numbers. Since you should be specifying version numbers, you're already off to a bd start.

Using the preferred style, changes in git are easier to read.

Percent string arrays can also grow very long, creating a long line of code. Lines should be short since GitHub cuts off lines at 80 characters.

Multiline percent strings still have the problem that you cannot specify cookbook version numbers.

The `%w(…)` [percent string syntax](http://www.ruby-doc.org/core-2.2.0/doc/syntax/literals_rdoc.html#label-Percent+Strings) may be difficult to understand if you or another developer are new to Ruby. Don't assume a high level of familiarity with Ruby.

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
depends apt
depends git, '~> 4.2'
depends jenkins '2.3.1'
…
```

## 3. Cookbook names

### 3.1 Use underscores and lowercase letters to name cookbooks

#### Discussion

The cookbook name is the GitHub repository name. Use lowercase names with underscores separating words. This is often called "[snake case][snake_case]."

Since Chef is written in Ruby, do not use symbols that are invalid for Ruby methods. Hyphens are invalid symbols for Ruby method names, so don't use hyphens in cookbook names.

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
```

## 4. Attributes

### 4.1 Use strings to access node attribute keys

Use the style, `node['with']['single']['quotes']`.

#### Discussion

For more background see this post, [Remove contentious rule FC001](https://github.com/acrmp/foodcritic/issues/86).

Each node attribute is a [Mash][mash] (defined in [lib/chef/mash.rb][mashdef]).  It's inefficient and can lead to problems. A valid node attribute is not necessarily a valid Ruby symbol or a valid Ruby method—so just skip the magic and stick to strings.

##### Correct

```ruby
node['with']['single']['quotes']
```

You may be forced to use variables as key names:
```ruby
node["maybe_with_#{interpolation}_but_only_if_necessary"]

some_fancy_key = "may_be_clearer_to_#{interpolate}_here"
node[some_fancy_key]
```

##### Incorrect

The Mash class exists so you can call node attributes in the following bad ways. Don't use Mash.

```ruby
node["with"]["any"]["double"]["quotes"]
node.with.any.accessor.calls
node[:with][:any][:symbols]
```

### 4.2 Use #tap to access nested attributes

If you are getting or setting many node attributes at the same nested level, use Ruby's [#tap][#tap] method to simplify your code.

#### Discussion

Prefer using #tap if you have long lines.

Using #tap will improve readability if you have long lines caused by lots of nested attributes. Since lots of nested attributes can make your code bloat out with lots of long lines, it will improve the legibility of such code to use #tap.

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

## 5. Templates

### 5.1 Do not use Ruby logic in ERB templates

Do not use Ruby logic in ERB templates. Instead, use the `variables` property when using Chef's `template` resource.

#### Discussion

When using the Chef `template` resource, do not put logic in your ERB (Embedded Ruby) template files.

The Chef `template` resource has an optional `variables` property, which takes a hash. The keys of the hash are used to create variables that are made available in the scope of the ERB template.

Perform all necessary logic in your recipes, custom resource actions, or helper methods to create a hash. Then pass the hash to the `variables` property on your `template` resource call.

Separating logic from templates allows you to isolate concerns. For example, you may want to test a helper method to verify that it provides the correct logical value to a template.

##### Incorrect

In the following scenario, the code uses logic in an ERB template to determine the contents of a hypothetical configuration file.

In the hypothetical configuration file,  `an_important_config_key` is set to a value determined by simple Ruby logic within the configuration file's ERB template.

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

# Improve your experience

Here are some notes you can refer to. They will help make the best of your experience in designing and building things with Chef.

## Ruby

Understanding Ruby is incredibly helpful. Encourage your team to be opinionated and to agree on style and convention.

Increasing readability will reduce software defects. Follow [The Ruby Style Guide][ruby-style].

### Ruby style caveats

Understand that there are certain recommendations in The Ruby Style Guide that may affect the logical operation of your software.

#### Logical order of precedence

For example, `and` and `&&` have a different [order of precedence][precedence]. Purposefully following an order of precedence may be functionally significant and more important than following the style guide.

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

## Testing

Tests are an essential part of any team's workflow. Don't let the notion of testing intimidate you! I recommend starting with integration tests, and then adding unit tests.

### Unit testing with ChefSpec

Unit tests verify Chef's compile phase. Unit tests verify that certain Chef resources were called with certain properties. Unit tests are used to ensure that the code you've written is going to do what you thought it was going to do.

Since unit tests run before the Chef execute phase, unit tests perform quickly, allowing you to fail faster.

Unit tests are written in [ChefSpec][chefspec].

Take a look at my example cookbooks for reference on unit testing.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

### Integration testing with InSpec

Integration tests run against a converged virtual machine. Use integration tests to verify that Chef recipes configure a virtual machine to a desired state.

Integration tests are written in [InSpec][inspec].

Take a look at my example cookbooks for reference on integration testing.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

### Betterspecs.org

[Betterspecs.org][betterspecs] will help you learn core concepts about testing.

#### Caveats

Some concepts at Betterspecs.org don't translate exactly to InSpec—for example, Betterspecs.org recommends using "expect" syntax instead of "should" syntax. However, if you're writing tests in InSpec, you *should* use "should" syntax since that's what it was designed for.

## Cookbook Design

I emphasize good cookbook design. Chef is incredibly flexible, but enforcing design choices will decrease defects and result in better code that is easier to maintain.

### Resource cookbooks

[You'll see the phrase "don't repeat yourself" fairly often, and you'll see the acronym DRY][dry].

Many cookbooks are recipe-centric, and contain a set of recipes. These recipes are added to a run list, and execute linearly interpreted Ruby instructions, there are also plenty of design patterns you can put to use. One I like a lot is what I'm calling the "resource cookbook" pattern.

C calls them libraries, Python calls them packages, and Ruby calls them gems. With Chef, a similar notion is to author **resource cookbooks**. Resource cookbooks define core, reusable functionality in the form of Chef resources to configure a machine in a specific way.

After writing a resource cookbook, you can then write a "wrapper cookbook" which consumes the resource cookbook. This allows for separation of logical concerns, and isolated testing—this results in safer code with fewer defects.

Take a look at my example cookbooks for reference on wrapping resource cookbooks and creating custom resources.

- [https://supermarket.chef.io/cookbooks/example][example]
- [https://supermarket.chef.io/cookbooks/example_resources][example_resources]

# Author

Created and maintained by [Kevin J. Dickerson][kevin]. <kevin.dickerson@loom.technology>

# References

References are listed at the bottom of this markdown document. Look at the bottom of the [RAW view of this document][raw] to see the references.

# Changes

This guide follows [Semantic Versioning 2.0][semver].

## In progress

- Updates language syntax highlighting tags in markdown

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

[mash]: http://www.rubydoc.info/gems/chef/Mash
[mashdef]: https://github.com/chef/chef/blob/master/lib/chef/mash.rb
[snake_case]: https://en.wikipedia.org/wiki/Snake_case
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[kevin]: http://kevinjdickerson.com/
[semver]: http://semver.org/
[#tap]: http://ruby-doc.org/core-2.3.0/Object.html#method-i-tap
[ruby-style]: https://github.com/bbatsov/ruby-style-guide
[chefspec]: https://github.com/sethvargo/chefspec
[inspec]: https://github.com/chef/inspec
[betterspecs]: http://betterspecs.org/
[example]: https://supermarket.chef.io/cookbooks/example
[example_resources]: https://supermarket.chef.io/cookbooks/example_resources
[precedence]: http://ruby-doc.org/core-2.2.0/doc/syntax/precedence_rdoc.html
[raw]: https://raw.githubusercontent.com/kevindickerson/chef-style-guide/master/README.md
