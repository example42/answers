# Introduction to Puppet

## Beginner 

What is Puppet?

A plain, basic, answer for absolute beginners is:

Puppet is a tool used tp manages computers.
Many computers.

It permits to automate the configuration of One or Thousands machines: the automation of automation!

## Junior

Puppet is an Open Source Configuration Management software, it's used to centralize and automate the configuration of systems.

Puppet can manage components of an Operating System like packages, services, files, users and so on, but can also be used to manage network devices, like routers, switches, load balancers and firewalls, or basically any kind of computing resources, like virtual machines, cloud instances and docker containers.

It uses a declarative language to describe the expected status of a system: with this language we DECLARE what resources we want on the nodes we manage, and how these resources should be configured, then it's up to Puppet to actually make this possible, abstracting from the underlying Operating System.

Puppet always uses the same layout and syntax for resources:

    <resource_type> { '<resource title>':
      parameter  => 'value',
      parameter2 => 'other value',
    }

This pattern is named “resource type declaration”.
