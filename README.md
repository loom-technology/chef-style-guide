# Chef Style Guide

Follow this guide to produce more readable code with fewer errors.

## Metadata

Prefer a single line for each dependency, especially if you have a long list of dependencies.

Specify versions with pessimistic versioning.

**Worse**

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

**Preferred**

```ruby
depends apt
depends git, '~> 4.2'
depends jenkins '2.3.1'
…
```

If you use percent strings, you cannot specify version numbers.

Using the preferred style, changes in git are easier to read.

Percent string arrays can also grow very long, creating a long line of code. Lines should be short since GitHub cuts off lines at 80 characters.

Multiline percent strings still have the problem that you cannot specify cookbook version numbers.


The `%w(…)` [percent string syntax](http://www.ruby-doc.org/core-2.2.0/doc/syntax/literals_rdoc.html#label-Percent+Strings) may be difficult to understand if you or another developer are new to Ruby. Don't assume a high level of familiarity with Ruby.

## Cookbook names

The cookbook name is the GitHub repository name, using Ruby underscore style.

**Incorrect**

```ruby
logstash_cookbook
logstash-cookbook
long-cookbook-name
```

**Correct**

```ruby
logstash
long_cookbook_name
```

## Attribute notation

Use the style, `node['with']['single']['quotes']`. For more background see this post, [Remove contentious rule FC001](https://github.com/acrmp/foodcritic/issues/86).

**Correct**

```ruby
node['with']['single']['quotes']
```

**Incorrect**

```ruby
node["with"]["any"]["double"]["quotes"]
node.with.any.accessor.calls
node[:with][:any][:symbols]
```

## Tap

If you are configuring node attributes, you can use the `#tap` method. This can improve readability if you have long lines caused by lots of nested attributes.

**Incorrect**

This line is too long.

```ruby
default['an_application']['settings']['very_long_set_of_attributes']['even_more_configuration']['this_is_very_long'] = true
```

**Short, single-line attributes**

Or, more simply,

```ruby
default['java']['install_flavor'] = 'oracle'
default['java']['jdk_version'] = '8'
default['java']['oracle']['accept_oracle_download_terms'] = true
…
```

**Using tap**

```ruby
default['java'].tap do |java|
  java['install_flavor'] = 'oracle'
  java['jdk_version'] = '8'
  java['oracle']['accept_oracle_download_terms'] = true
  …
end
```

Prefer using `#tap` if you have long lines. Long lines are cut off in GitHub. This will improve the readability of your attribute settings.

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

### Library-based cookbooks

[You'll see the phrase "don't repeat yourself" fairly often, and you'll see the acronym DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

Although many cookbooks are a set of linear interpereted Ruby instructions, there are also plenty of design patterns you can put to use. One I like a lot is the "library-based cookbook" pattern.

C calls them libraries, Python calls them packages, and Ruby calls them gems. With Chef, a similar notion is to author library-based cookbooks. Library-based cookbooks contain core, reusable functionality to configure something in a specific way. Then you can write a "wrapper cookbook" to consume the library-based cookbook. This allows for separation of logical concerns, and isolated testing—this results in safer code with fewer defects!

A good example of a library-based cookbook is the mysql cookbook, authored largely by Sean Omeara (GitHub username "someara"). He and I strongly advocate for this pattern.

- https://supermarket.chef.io/cookbooks/mysql
- https://github.com/someara
