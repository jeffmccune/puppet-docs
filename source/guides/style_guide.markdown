---
layout: default
title: Style Guide
---

Style Guide
===========

* * *

### Style Guide Metadata

Version 1.0

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://www.faqs.org/rfcs/rfc2119.html).

## Puppet Version

This style guide is largely specific to Puppet versions 2.6.x; some of its
recommendations are based on some language features that became
available in version 2.6.0 and later.

## Why a Style Guide?

Puppet Labs develops modules for customers and the community, and these modules
should represent the best known practice for module design and style. Since
these modules are developed by many people across the organisation, a central
reference was needed to ensure a consistent pattern, design, and style.

## General Philosophies

No style manual can cover every possible circumstance. When a judgement call
becomes necessary, keep in mind the following general ideas:

1.  **Readability matters.**
    If you have to choose between two equally effective alternatives, pick the
    more readable one. This is, of course, subjective, but if you can read your
    own code three months from now, that's a great start.
2.  **Inheritance should be avoided.**
    In general, inheritance leads to code that is harder to read.
    Most use cases for inheritance can be replaced by exposing class
    parameters that can be used to configure resource attributes. See
    the [Class Inheritance](#class-inheritance) section for more details.
3.  **Modules must work with an ENC without requiring one.**
    An internal survey yielded near consensus that an ENC should not be
    required.  At the same time, every module we write should work well
    with an ENC.
4.  **Classes should generally not declare other classes.**
    Declare classes as close to node scope as possible. Classes which require
    other classes should not directly declare them and should instead allow the
    system to fail if they are not declared by some other means. (Although the
    include function allows multiple declarations of classes, it can result in
    non-deterministic scoping issues due to the way parent scopes are assigned.
    We might revisit this philosophy in the future if class multi-declarations
    can be made deterministic, but for now, be conservative with declarations.)

## Module Metadata

Every module must have Metadata defined in the Modulefile data file
and outputted as the metadata.json file.  The following Metadata
should be provided for all modules.

    name 'myuser-mymodule'
    version '0.0.1'
    author 'Author of the module - for shared modules this is Puppet Labs'
    summary 'One line description of the module'
    description 'Longer description of the module including an example'
    license 'The license the module is release under - generally GPLv2 or Apache'
    project_page 'The URL where the module source is located'
    dependency 'otheruser-othermodule', '1.2.3'

### Style Versioning

This style guide must be version controlled, which will allow modules to comply
with a specific version of the style guide.

The relevant style guide version must be embedded as metadata in the
Modulefile.  Use the `style_version` field:

    style_version "1.0"

This style metadata information may be used for automated linting at some later
date.

## Spacing, Indentation, & Whitespace

Module manifests complying with this style guide:

* Must use two-space soft tabs
* Must not use literal tab characters
* Must not contain trailing white space
* Should not exceed an 80 character line width
* Should align fat comma arrows (`=>`) within blocks of attributes

## Comments

Although the Puppet language allows multiple comment types, we prefer
hash/octothorpe comments (`# This is a comment`) because they're generally the
most visible to text editors and other code lexers.

1.  Should use `# ...` for comments
2.  Should not use `// ...` or `/* ... */` for comments

## Quoting

All strings that do not contain variables should be enclosed in
single quotes.  Double quotes should be used when variable interpolation is
required.  Quoting is optional when the string is an alphanumeric
bare word and is not a resource title.

All variables should be enclosed in braces when interpolated in a
string.  For example:

**Good:**

{% highlight ruby %}
    "/etc/${file}.conf"
    "${operatingsystem} is not supported by ${module_name}"
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    "/etc/$file.conf"
    "$operatingsystem is not supported by $module_name"
{% endhighlight %}

Variables standing by themselves should not be quoted.  For example:

**Good:**

{% highlight ruby %}
    mode => $my_mode
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    mode => "$my_mode"
    mode => "${my_mode}"
{% endhighlight %}

## Resources

### Resource Names

All resource titles should be quoted. (Puppet
supports unquoted resource titles if they do not contain spaces or
hyphens, but you should avoid them in the interest of consistent look-and-feel.)

**Good:**

{% highlight ruby %}
    package { openssh: ensure => present }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    package { 'openssh': ensure => present }
{% endhighlight %}

### Arrow Alignment

All of the fat comma arrows (`=>`) in a resource's attribute/value list should
be aligned. The arrows should be placed one space ahead of the longest
attribute name.

**Good:**

{% highlight ruby %}
    exec { 'blah':  
      path => '/usr/bin',  
      cwd  => '/tmp',
    }

    exec { 'test':  
      subscribe   => File['/etc/test'],  
      refreshonly => true,
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    exec { 'blah':  
     path => '/usr/bin',  
     cwd => '/tmp',
    }

    exec { 'test':  
      subscribe => File['/etc/test'],  
      refreshonly => true,
    }
{% endhighlight %}

### Attribute Ordering

If a resource declaration includes an `ensure` attribute, it should be the
first attribute specified.

**Good:**

{% highlight ruby %}
    file { '/tmp/foo':
      ensure => file,
      owner  => '0',
      group  => '0',
      mode   => '0644',
    }
{% endhighlight %}

(This recommendation is solely in the interest of readability, as Puppet
ignores attribute order when syncing resources.)

### Compression

Within a given manifest, resources should be grouped by logical relationship to
each other, rather than by resource type. Use of the semicolon syntax to
declare multiple resources within a set of curly braces is not recommended,
except in the rare cases where it would improve readability.

**Good:**

{% highlight ruby %}
    file { '/tmp/a':
      content => 'a',
    }

    exec { 'change contents of a':
      command => 'sed -i.bak s/a/A/g /tmp/a',
    }

    file { '/tmp/b':
      content => 'b',
    }

    exec { 'change contents of b':
      command => 'sed -i.bak s/b/B/g /tmp/b',
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    file {
      "/tmp/a":
        content => "a";
      "/tmp/b":
        content => "b";
    }

    exec {
      "change contents of a":
        command => "sed -i.bak s/b/B/g /tmp/a";
      "change contents of b":
        command => "sed -i.bak s/b/B/g /tmp/b";
    }
{% endhighlight %}

### Symbolic Links

In the interest of clarity, symbolic links should be declared by using an
ensure value of `ensure => link` and explicitly specifying a value for the
`target` attribute. Using a path to the target as the ensure value is not
recommended.

**Good:**

{% highlight ruby %}
    file { '/var/log/syslog':
      ensure => link,
      target => '/var/log/messages',
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    file { '/var/log/syslog':
      ensure => '/var/log/messages',
    }
{% endhighlight %}

### File Modes

File modes should be represented as 4 digits rather than 3, to explicitly show
that they are octal values.

In addition, file modes should be specified as single-quoted strings instead of
bare word numbers.

**Good:**

{% highlight ruby %}
    file { '/var/log/syslog':
      ensure => present,
      mode   => '0644',
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    file { '/var/log/syslog':
      ensure => present,
      mode   => 644,
    }
{% endhighlight %}

### Resource Defaults

Resource defaults should be used in a very controlled manner, and should only
be declared at the edges of your manifest ecosystem.  This is due to the way
resource defaults propagate through dynamic scope, which can have unpredictable
effects far away from where the default was declared.

**Good:**

{% highlight ruby %}
    # /etc/puppetlabs/puppet/manifests/site.pp:
    File {
      mode   => '0644',
      owner  => 'root',
      group  => 'root',
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    # /etc/puppetlabs/puppet/modules/ssh/manifests/server/uk/foo.pp
    File {
      mode   => '0600',
      owner  => 'nobody',
      group  => 'nogroup',
    }
{% endhighlight %}

## Conditionals

### Keep Resource Declarations Simple

You should not intermingle conditionals with resource declarations. When using
conditionals for data assignment, you should separate conditional code from the
resource declarations.

**Good:**

{% highlight ruby %}
    $foo_mode = $operatingsystem ? {
      debian => '0007',
      redhat => '0776',
      fedora => '0007',
    }

    file { '/tmp/foo':
       mode => $foo_mode,
    }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    file { '/tmp/foo':
      mode => $operatingsystem ? {
        debian => '0777',
        redhat => '0776',
        fedora => '0007',
      }
    }
{% endhighlight %} 

### Defaults for Case Statements and Selectors

Case statements should have default cases.

Hence both of these are valid.  In addition, the default case should fail the
catalog compilation when the resulting behavior cannot be predicted on the
majority of platforms the module will be used on.

For selectors, the omission of a default selection should only be used if
catalog compilation failure is explicitly desired in this case.

The following example follows the recommended style:

{% highlight ruby %}
    case $operatingsystem {
      centos: {
        $version = '1.2.3'
      }
      solaris: {
        $version = '3.2.1'
      }
      default: {
        fail("Module $module_name is not supported on $operatingsystem")
      }
    }
{% endhighlight %}

## Classes

### Separate Files

All classes and resource type definitions must be in separate files in the
`manifests` directory of their module. For example:

{% highlight ruby %}
    # /etc/puppetlabs/puppet/modules/foo/manifests
    
    # init.pp
      class foo { }
    # bar.pp
      class foo::bar { }
    # dostuff.pp
      define foo::dostuff () { }
{% endhighlight %}

This is functionally identical to declaring all classes and defines in init.pp, but highlights the structure of the module and makes everything more legible.

### Internal Organization of a Class

Classes should be organised with a consistent structure and style.  In the
below list there is an implicit statement of "should be at this relative
location" for each of these items.  The word "may" should be interpreted as "If
there are any X's they should be here".

1.  Should define the class and parameters
2.  Should validate any class parameters and fail catalog compilation if any
    parameters are invalid
3.  Should default any validated parameters to the most general case
4.  May declare local variables
5.  May declare relationships to other classes `Class['foo'] -> Class['bar']`
6.  May override resources
7.  May declare resource defaults
8.  May declare defined resource types
9.  May declare native types (Users, Groups, Services...)
10. May declare other resources
11. May declare Resource relationships inside of conditionals

The following example follows the recommended style:

{% highlight ruby %}
    class myservice($ensure='running') {

      if $ensure in [ running, stopped ] {
        $ensure_real = $ensure
      } else {
        fail('ensure parameter must be running or stopped')
      }

      case $operatingsystem {
        centos: {
          $package_list = 'openssh-server'
        }
        solaris: {
          $pacakge_list = [ SUNWsshr, SUNWsshu ]
        }
        default: {
          fail("module $module_name does not support $operatingsystem")
        }
      }

      $variable = 'something'

      Package { ensure => present, }

      File { owner => '0', group => '0', mode => '0644' }
  
      package { $package_list: }

      file { "/tmp/${variable}":
        ensure => present,
      }

      service { 'myservice':
        ensure    => $ensure_real,
        hasstatus => true,
      }
    }
{% endhighlight %}

### Relationship Declarations

Relationship declarations with the chaining syntax should only be used in the
"left to right" direction.

**Good:** 

    Package[bar] -> Service[foo]

**Bad:**

    Service[foo] <- Package[bar]

When possible, you should prefer metaparameters to relationship declarations.
One example where metaparameters aren't desirable is when subclassing would be
necessary to override behavior; in this situation, relationship declarations
inside of conditionals should be used.

### Classes and Defined Resource Types Within Classes

Classes and defined resource types must not be defined within other classes.

**Bad:**

{% highlight ruby %}
    class foo {
      class bar { ... }
    }
{% endhighlight %}

**Also bad:**

{% highlight ruby %}
    class foo {
      define bar() { ... }
    }
{% endhighlight %}

### Class Inheritance

Inheritance may be used within a module, but must not be used across module
namespaces. Cross-module dependencies should be satisfied in a more portable
way that doesn't violate the concept of modularity, such as with include
statements or relationship declarations.

**Good:**

{% highlight ruby %}
    class ssh { ... }

    class ssh::client inherits ssh { ... }

    class ssh::server inherits ssh { ... }

    class ssh::server::solaris { ... }
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    class ssh inherits server { ... }

    class ssh:client inherits workstation { ... }

    class wordpress inherits apache { ... }
{% endhighlight %}

Inheritance in general should be avoided when alternatives are viable.  For
example, instead of using inheritance to override relationships in an existing
class when stopping a service, consider using a single class with an ensure
parameter and conditional relationship declarations:

{% highlight ruby %}
    class bluetooth($ensure=present, $autoupgrade=false) {
       # Validate class parameter inputs. (Fail early and fail hard)

       if ! ($ensure in [ "present", "absent" ]) {
         fail("bluetooth ensure parameter must be absent or present")
       }

       if ! ($autoupgrade in [ true, false ]) {
         fail("bluetooth autoupgrade parameter must be true or false")
       }

       # Set local variables based on the desired state

       if $ensure == "present" {
         $service_enable = true
         $service_ensure = running
         if $autoupgrade == true {
           $package_ensure = latest
         } else {
           $package_ensure = present
         }
       } else {
         $service_enable = false
         $service_ensure = stopped
         $package_ensure = absent
       }

       # Declare resources without any relationships in this section

       package { [ "bluez-libs", "bluez-utils"]:
         ensure => $package_ensure,
       }

       service { hidd:
         enable         => $service_enable,
         ensure         => $service_ensure,
         status         => "source /etc/init.d/functions; status hidd",
         hasstatus      => true,
         hasrestart     => true,
      }

      # Finally, declare relations based on desired behavior

      if $ensure == "present" {
        Package["bluez-libs"]  -> Package["bluez-utils"]
        Package["bluez-libs"]  ~> Service[hidd]
        Package["bluez-utils"] ~> Service[hidd]
      } else {
        Service["hidd"]        -> Package["bluez-utils"]
        Package["bluez-utils"] -> Package["bluez-libs"]
      }
    }
{% endhighlight %}

(This example makes several assumptions and is based on an example provided in
the Puppet Master training for managing bluetooth.)

In summary:

* Class inheritance is only useful for overriding resource attributes; any
  other use case is better accomplished with other methods.
* The most commonly overridden parameters are relationship metaparameters.
* Other parameters, e.g. ensure and enable may have behavior changed through
  the use of variables and conditional logic.

### Namespacing Variables

When using top-scope variables, including facts, Puppet modules should
explicitly specify the empty namespace (i.e., `$::operatingsystem`, not
`$operatingsystem`) to prevent accidental scoping issues.

### Display Order of Class Parameters

In parameterized class and defined resource type declarations, parameters that
are required should be listed before optional parameters (i.e. parameters with
defaults).

**Good:**

{% highlight ruby %}
    class foo (
      $two,
      $one = "default",
      $three = "default"
    ) {}
{% endhighlight %}

**Bad:**

{% highlight ruby %}
    class foo (
      $one = 'default',
      $two,
      $three = 'default'
    ) {}
{% endhighlight %}

## Tests

All manifests should have a corresponding test manifest in the module's `tests`
directory.

    modulepath/foo/manifests/{init,foo}.pp
    modulepath/foo/tests/{init,foo}.pp

## Puppet Doc

Classes and defined resource types should be documented inline using the
following conventions:

For classes:

{% highlight ruby %}
    # Full description of class here.
    #
    # == Parameters
    #
    # Document parameters here
    #
    # [*sample_bar*]
    #   Description of this variable.
    #
    # == Variables
    #
    # Here you should define a list of variables that this module would require.
    #
    # [*$foo_var*]
    #     Description of this variable
    #
    # == Examples
    #
    # Put some examples on how to use your class here.
    #
    #   $example_var = "blah"
    #   include example_class
    #
    # == Authors
    #
    # Author Name <author@domain.com\>
    #
    # == Copyright
    #
    # Copyright 2011 Company Name Inc, unless otherwise noted.
    #
    class example_class {
      ...
    }
{% endhighlight %}

For defined resources:

{% highlight ruby %}
    # Description of resource here
    #
    # == Parameters
    #
    # Document parameters here
    #
    # [*namevar*]
    #   Always document namevar
    #
    # [*sample_bar*]
    #   Description of this variable.
    #
    # == Examples
    #
    # Provide some examples on how to use this type:
    #
    #   example_class::example_resource{
    #     "namevar":
    #       sample_param => "value",
    #   }
    #
    define example_class::example_resource($example_var) {
      ...
    }
{% endhighlight %}

This will allow documentation to be automatically extracted with the puppet doc
tool.

## The extlookup() Function

Modules should avoid the use of extlookup() in favor of ENCs or other
alternatives.
