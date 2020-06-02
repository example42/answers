## Modules

Puppet code is distributed and reused via **modules**.

There are modules to manage virtually any kind of software, systems' components, network device or cloud object.

The [Puppet Forge](https://forge.puppet.com) is the official repository of modules, from Puppet company, its partners and the community.

Most of the public modules are published on [GitHub](https://github.com/search?q=puppet+modules).

Any module added on Puppet's **modulepath** is available for usage.

---

### Modules conventions: class autoloading

A module is a directory. The name of the module is the name of the directory.
Puppet has powerful naming conventions for modules.

A mysql module, for example, stays in the directory: 

    mysql/

The main module class, called mysql, is always in a manifest called

    mysql/manifests/init.pp

and can be declared with just:

    include mysql

Other classes are named and placed in well defined files. Class mysql::server will be defined in:

    mysql/manifests/server.pp

Class mysql::server::ha will be in:

    mysql/manifests/server/ha.pp

---

### Modules conventions: templates

Templates are files in Puppet epp or Ruby erb format which are typically passed to the epp() and template() functions inside Puppet code.

They are placed in the directory:

    mysql/templates/

In our code, where we have:

    file { '/etc/my.cnf':
      content => template('mysql/my.cnf.erb'),
    }

We interpolate the erb template placed in

    mysql/templates/my.cnf.erb

If we have:

    file { '/etc/my.cnf':
      content => epp('mysql/ha/my.cnf.epp'),
    }

We interpolate the epp template placed in

    mysql/templates/ha/my.cnf.epp

---

### Modules conventions: files


Static files are placed in:

    mysql/files/

We can serve them as is with the source argument of the file resource:

    file { '/etc/my.cnf':
      source => 'puppet:///modules/mysql/my.cnf',
    }

The actual file used will be

    mysql/files/my.cnf

### Modules conventions: tasks and plans

Bolt Tasks are the "new" thing. They can be scripts in any language which can be executed via Bolt:

    mysql/tasks/*

Bolt plans may have space in a modern module as well. They can orchestrate tasks and puppet code:

    mysql/plans/*.pp *.yaml

Plugins directory with custom facts, types, providers, functions  in Ruby code et here.

    mysql/lib/*.rb


### Where modules are placed: modulepath

Modulepath (comma separated directories where modules are stored):

Modules are looked in a list of directories, as configured via the $modulepath
config entry in global puppet.conf and in the environment's environment.conf.

On old Puppet 3 default module path was:

    /etc/puppet/modules:/usr/share/puppet/modules

and, when using directory environments:

    /etc/puppet/environments/$environment/modules
