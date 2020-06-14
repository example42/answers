## Resource Types (Types)
<img src="gfx/junior.png" class="skill">

Resources (also called Resource Types and Types) are single **units of configuration** that potentially can manage any "object" in an Operating System or more widely in an IT infrastructure.

A resources is composed by:

- A **type** (package, service, file, user, mount, exec, interface, ec2_instance ...)
- A **title** (how the resource is called and referred in Puppet code)
- Zero or more **arguments**

The syntax is always as follows:

    type { 'title':
      argument  => value,
      other_arg => value,
    }

Example for a **file** resource type:

    file { 'motd':
      ensure  => file,
      path    => '/etc/motd',
      content => 'Tomorrow is another day',
    }

---

## Resource Types documentation
<img src="gfx/junior.png" class="skill">

Information about resource types and their documentation can be found in different places:

  - Online is published the complete [Type Reference](http://docs.puppetlabs.com/references/latest/type.html) for the latest or earlier versions of Puppet.

  - The `puppet describe` command:

        puppet describe <resource_type>
      
    for example, to find from the command line information about the file resource:

        puppet describe file

    To list the description of ll the available types, run:
    
        puppet describe --list

  - Give a glance to Puppet code for the list of **native** resource types (the ones written in Ruby language, based on the types and providers pattern):

        ls $(facter rubysitedir)/puppet/type

---

## Simple samples of resources
<img src="gfx/junior.png" class="skill">

The most common native resources, shipped with Puppet by default are: package, service, file, user, group, cron, exec, mount. Here are some simple examples.

Installation of OpenSSH package:

    package { 'openssh':
      ensure => present,
    }

Creation of /etc/motd file:

    file { 'motd':
      ensure => file,
      path   => '/etc/motd',
    }

Start of httpd service:

    service { 'httpd':
      ensure => running,
      enable => true,
    }

Creation of oscar user:

    user { 'oscar':
      ensure => present,
      uid    => 1024,
    }

Each of these resources have several other arguments that allows to define every characteristic of the resource.

---

## More complex examples
<img src="gfx/junior.png" class="skill">

Here are some more complex examples with usage of variables, resource references, arrays and relationships.

Management of nginx service with parameters defined in module's variables

    service { 'nginx':
      ensure     => $::nginx::manage_service_ensure,
      name       => $::nginx::service_name,
      enable     => $::nginx::manage_service_enable,
    }

Creation of nginx.conf with content retrieved from different sources (first found is served)

    file { 'nginx.conf':
      ensure  => file,
      path    => '/etc/nginx/nginx.conf',
      source  => [
          "puppet:///modules/site/nginx.conf--${::fqdn}",
          "puppet:///modules/site/nginx.conf"
      ],
    }

Installation of the Apache package triggering a restart of the relevant service:

    package { 'httpd':
      ensure => $ensure,
      name   => $apache_package,
      notify => Class['Apache::Service'],
    }


---

## Resource Abstraction Layer (RAL)
<img src="gfx/junior.png" class="skill">

Resources are abstracted from the underlying OS, this is achieved via the **Resource Abstraction Layer** (RAL) composed of resource **types** that can have different **providers**.

A type specifies the attributes that a given resource may have, a provider implements the relevant type on the underlying Operating System.

For example the ```package``` type is known for the great number of providers (yum, apt, msi, gem ... ).

    ls $(facter rubysitedir)/puppet/provider/package

With the command ```puppet resource``` we can represent the current status of a system's resources in Puppet language (note this can be done for any resource, even the ones not managed by Puppet):

To show all the existing users on a system (or only the root user):

    puppet resource user
    puppet resource user root

To show all the installed packages:

    puppet resource package

To show all the system's services:

    puppet resource service

It's also possible to directly modify them with ```puppet resource``` (note that this is not generally the way Puppet is used to manage the system's resources):

    puppet resource service httpd ensure=running enable=true


---

## Resources defaults

It's possible to set default argument values for a resource in order to reduce code duplication. The syntax is:

    Type {
      argument => value,
    }

Common examples:

    Exec {
      path => '/sbin:/bin:/usr/sbin:/usr/bin',
    }

    File {
      mode  => 0644,
      owner => 'root',
      group => 'root',
    }

Resource defaults can be overriden when declaring a specific resource of the same type.

Note that the "Area of Effect" of resource defaults might bring unexpected results. The general suggestion is:

Place **global** resource defaults in manifests/site.pp outside any node definition.

Place **local** resource defaults at the beginning of a class that uses them (mostly for clarity sake, as they are parse-order independent).

---

## Resource references

In Puppet any resource is uniquely identified by its type and its name.
We can't have 2 resources of the same type with the same name in a catalog.

We have seen that we declare resources with a syntax like:

    type { 'name':
      arguments => values,
    }

When we need to reference to them in our code the syntax is like:

    Type['name']

Some examples:

    file { 'motd': ... }
    apache::virtualhost { 'example42.com': .... }
    exec { 'download_myapp': .... }

are referenced, respectively, with

    File['motd']
    Apache::Virtualhost['example42.com']
    Exec['download_myapp']

---

## Resource Metaparameters

Metaparameters are parameters available to any resource type, they can be used for different purposes:

- Manage dependencies (**before**, **require**, **subscribe**, **notify**, **stage**)

- Manage resources' application policies (**audit**, **noop**, **schedule**, **loglevel**)

- Add information to a resource (**alias**, **tag**)

[Official documentation on Metaparameters](http://docs.puppetlabs.com/puppet/latest/reference/metaparameter.html)

The metaparameters that manage dependencies are widely used to apply the resources in a catalog following the expected order.