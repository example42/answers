## Introduction
<img src="gfx/beginner.png" class="skill">

Puppet is a software that **automates** the configurations of computers.

With its **language** it can natively **manage resources** like:  
packages, services, files, users, mounts, crons...  

Thanks to its extensible design, using dedicated **modules**
it can configure virtually any kind of IT resource of any OS, like:  
ec2_instances, docker_networks, mysql::users, gcompute_firewalls 

Typical Puppet users are **Sysadmins** and **Site Reliability Engineers** who
manage hundreds or thousands of servers.

Puppet is developed by a company called... **[Puppet](https://puppet.com/)**

---

## Puppet in Action
<img src="gfx/beginner.png" class="skill">

This is a sample output of the execution of **puppet agent** on a node.

A **node** is a computer managed by Puppet.

What you see is a list of configured resources on a node:

<asciinema-player cols="200" src="casts/puppet_run3.cast" idle-time-limit="1" autoplay="4"></asciinema-player>

---

## Puppet software
<img src="gfx/junior.png" class="skill">

Puppet is an [Open Source](https://github.com/puppetlabs/puppet) Server automation framework and application.

It's a **Configuration Management Tool** (as Ansible, CFEngine, Chef, Salt...).

It allows to configure, with a **declarative** Domain Specific **Language** (DSL)
potentially any kind of resource on any IT device.

Native **Puppet agent** can run on Linux, Unix (Solaris, AIX, *BSD), MacOS, Windows and,
typically, receives a list of the resources to apply local from a server.

The **Puppet Server** is the central place where is stored all the code and data needed
to configure clients.

---

## Configuration Management advantages
<img src="gfx/junior.png" class="skill">

Configuration management tools allow Sys Admins to **configure servers with code**.

Code is centralized and can be managed with a Version Control System like [git](https://git-scm.com/).

Servers and devices managed in this way have been described as **Infrastructure as Code**.

[Versioned] code brings terrific consequences on different topics:

- **Track**: Commits log shows the history of change on the infrastructure (who did what when)
- **Test**: Code can be tested and its effect verified before production rollout
- **Reproduce**: Coherent and consistent server setups with aligned environments for devel, test, qa, prod nodes
- **Scale**: What's done once, for one, can be automated always for many servers