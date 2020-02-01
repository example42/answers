## Control repo

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

---

### Puppet environments and control repo

The control repo has the content of a Puppet environment.

Starting from Puppet 4 Puppet environments are placed under **$environmentpath**: 

    /etc/puppetlabs/code/environments/$environment

On Puppet 3 with directory environments path was:

    /etc/puppet/environments/$environment

---

## Puppetfile

---

## hiera.yaml

---

## hieradata

---

## environment.conf

---

## modules