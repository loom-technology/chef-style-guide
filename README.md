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

The cookbook name is the Github repository name, using Ruby underscore style.

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
