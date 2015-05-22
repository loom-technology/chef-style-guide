# Chef Style Guide

Follow this guide to produce readable code with fewer errors.

## Metadata

Say you have a long list of cookbooks you depend on. Which style is better?

**A**

    %w(apt git jenkins …).each do |cookbook|
      depends cookbook
    end

**B** (preferred)

    depends apt
    depends git
    depends jenkins
    …

If you use **A**, then you won't be able to specify version numbers.

The list can also grow very long, creating a very long line of code. Lines should be short. Github cuts off lines at 80 characters.

Using **B**, we also get clearer changes in git.

The `%w(…)` [percent string syntax](http://www.ruby-doc.org/core-2.2.0/doc/syntax/literals_rdoc.html#label-Percent+Strings) may be difficult to understand if you or another developer are new to Ruby. Don't assume a high level of familiarity with Ruby.

In short, the benchmark should be to reach towards higher readability. Simple is good!

Use **B**.

## Cookbook names

The cookbook name is the Github repository name.

**Correct**

    logstash
    long_cookbook_name

**Incorrect**

    logstash_cookbook
    logstash-cookbook
    long-cookbook-name

## Node attribute notation

Use the style, `node['with']['single']['quotes']`. For more background see this post, [Remove contentious rule FC001](https://github.com/acrmp/foodcritic/issues/86).

**Correct**

    node['with']['single']['quotes']

**Incorrect**

    node["with"]["any"]["double"]["quotes"]
    node.with.any.accessor.calls
    node[:with][:any][:symbols]
