# Files that Matter: Puppet

## Beginner 

## Junior

### Modules conventions

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

    Puppet has powerful naming conventions for modules.

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

Static files are placed in:

    mysql/files/

We can serve them as is with the source argument of the file resource:

    file { '/etc/my.cnf':
      source => 'puppet:///modules/mysql/my.cnf',
    }

The actual file used will be

    mysql/files/my.cnf


### Control repo

A control repo has a configuration files where is defined the modulepath to use, if to use caching and what configuration version to show on Puppet runs output:

    environment.conf
    
Hiera configuration, with the backends to use, the directory twhere data is stored and the hierarchy layers:

    hiera.yaml
    
A directory where to place Hiera data files (typically in yaml format, if default Yaml backed in used). Usually (can be configued in hiera.yaml) one of these:

    data/
    hieradata/

The file that defines the public external modules to install:

    Puppetfile

The first directory where the servers looks for Puppet code, it contains manifests outside modules with "global" code like definition of common resources, top scope variables and resource defaults. 

    manifests/

A site directory which is added to the modulepath and contains local custom modules. In this directory you have, for example the role and the profile modules and any other custom module created for local needs. Maybe be called like:

    site/
    site-modules/

A directory where modules defined in Puppetfile are copied. This is actually expected ot be empty in the control-repo, as it's populated wuding deployment:

    modules/


## Intermediate

### Module paths


Bolt Tasks are the "new" thing. They can be scripts in any language which can be executed via Bolt:

    mysql/tasks/*

Bolt plans may have space in a modern module as well. They can orchestrate tasks and puppet code:

    mysql/plans/*.pp *.yaml

Plugins directory with custom facts, types, providers, functions  in Ruby code et here.

    mysql/lib/*.rb


###

## Advanced


### Main files locations

Old and New Puppet files layouts (note this applies to Puppet official packages, distros may have custom layouts in their own packages)

Configuration directory ($confdir):

/etc/puppet/            # Puppet 3 or earlier
/etc/puppetlabs/puppet/ # Puppet 4 and PE

Log directory ($logdir):

/var/log/puppet # Puppet 3 or earlier
/var/log/puppetlabs/puppet # Puppet 4

Lib directory ($libdir) (contains Puppet operational data such as catalog, backup of files...):

/var/lib/puppet # Puppet 3 or earlier
/opt/puppetlabs/puppet/lib # Puppet 4

SSL directory ($libdir) (contains all the SSL certificates)

$logdir/ssl  # Puppet 3 or earlier
$confdir/ssl # Puppet 4

Manifest file (the first manifest parsed by the Master when compiling the catalog

/etc/puppet/manifests/site.pp # Puppet 3 with config-file environments
/etc/puppet/environments/$environment/manifests/site.pp # Puppet 3 with directory environments
/etc/puppetlabs/code/environments/$environment/manifests/site.pp # Puppet 4

Modulepath (comma separated directories where modules are stored):

/etc/puppet/modules:/usr/share/puppet/modules # Puppet 3
/etc/puppet/environments/$environment/modules # Added modules dir in Puppet 3 when using **directory environments**

Code directory in Puppet 4:

/etc/puppetlabs/code # $codedir
/etc/puppetlabs/code/environments/$environment # $environmentpath



## Advanced



## Senior