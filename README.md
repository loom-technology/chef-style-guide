# Chef Style Guide

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

The Mash class exists so you can call node attributes in the following bad ways. Don't use them.

```ruby
node["with"]["any"]["double"]["quotes"]
node.with.any.accessor.calls
node[:with][:any][:symbols]
```

### 4.2 Use #tap to access nested attributes

If you are getting or setting many node attributes at the same nested level, use Ruby's #tap method to simplify your code.

#### Discussion

Prefer using `#tap` if you have long lines.

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

# Improve your experience

Here are some notes you can refer to. They will help make the best of your experience in designing and building things with Chef.

## Ruby

Understanding Ruby is incredibly helpful. Encourage your team to be opinionated and to agree on a style works for the team.

Increasing readability will reduce software defects. Follow the community's Ruby Style guide.

- https://github.com/bbatsov/ruby-style-guide

## Testing

Tests are an essential part of our prescribed workflow.

Previously, we've recommended Chefspec and ServerSpec. Our new recommendation for testing methodology is to use our new framework called InSpec.

- http://betterspecs.org/
- https://github.com/sethvargo/chefspec
- http://serverspec.org/
- https://github.com/chef/inspec

## Cookbook Design

I emphasize good cookbook design. Chef is incredibly flexible, but enforcing design choices will decrease defects and result in better code that is easier to maintain.

### Resource cookbooks

[You'll see the phrase "don't repeat yourself" fairly often, and you'll see the acronym DRY][dry].

Many cookbooks are recipe-centric, and contain a set of recipes. These recipes are added to a run list, and execute linearly interpereted Ruby instructions, there are also plenty of design patterns you can put to use. One I like a lot is the "library-based cookbook" pattern.

C calls them libraries, Python calls them packages, and Ruby calls them gems. With Chef, a similar notion is to author library-based cookbooks. Library-based cookbooks contain core, reusable functionality to configure something in a specific way. Then you can write a "wrapper cookbook" to consume the library-based cookbook. This allows for separation of logical concerns, and isolated testing—this results in safer code with fewer defects!

Good examples of real life library-based cookbooks are the mysql cookbook and the httpd cookbook.

- https://supermarket.chef.io/cookbooks/mysql
- https://supermarket.chef.io/cookbooks/httpd

# Author

Created and maintained by [Kevin J. Dickerson][kevin]. <kevin.dickerson@loom.technology>

# Semver

This guide follows [Semantic Versioning 2.0][semver].

# Changes

## 0.1.0 (03/21/2016)

- Begin versioning this thing

[mash]: http://www.rubydoc.info/gems/chef/Mash
[mashdef]: https://github.com/chef/chef/blob/master/lib/chef/mash.rb
[snake_case]: https://en.wikipedia.org/wiki/Snake_case
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[kevin]: http://kevinjdickerson.com
[semver]: http://semver.org/
