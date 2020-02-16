## Introduction
<img src="gfx/beginner.png" class="skill">

Puppet **automates** the configurations of computers.

With its language it can **manage resources** like:
packages, services, files, users.

Thanks to its modular design, using dedicated **modules**
it can configure virtually any kind of IT resource of any OS.

Typical Puppet users are **Sysadmins** who
manage hundreds or thousands of servers,

Puppet is developed by a company called... [Puppet](https://puppet.com/)

<asciinema-player cols="200" src="casts/puppet_facts.cast" autoplay="4"></asciinema-player>

---

## Where Puppet is used
<img src="gfx/junior.png" class="skill">

Puppet is an Open Source **Configuration Management Tool**.

It allows to configure, with a **declarative** Domain Specific **Language** (DSL)
potentially any kind of resource on any IT device.

Native Puppet clients can run on Linux, Unix (Solaris, AIX, *BSD), MacOS, Windows, Supported Platforms).

Puppet can also configure, network and storage devices,
cloud resources and anything which may be managed via an API.

---

## Configuration Management advantages
<img src="gfx/junior.png" class="skill">

Configuration management changes the way we manage servers,
allowing us to configure them via code.

**Infrastructure as Code** implies: Track, Test, Deploy, Reproduce, Scale

Benefits and consequences are:

- Code commits log shows the history of change on the infrastructure
- Reproducible setups: Do once, repeat forever
- Scale quickly: Done for one, use on many
- Coherent and consistent server setups
- Aligned Environments for devel, test, qa, prod nodes

Alternatives to Puppet are Chef, CFEngine, Salt, Ansible.
