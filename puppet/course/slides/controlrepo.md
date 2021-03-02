## Control repo
<img src="gfx/junior.png" class="skill">

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
<img src="gfx/junior.png" class="skill">

The control repo has the content of a Puppet environment.

Starting from Puppet 4 Puppet environments are placed under **$environmentpath**: 

    /etc/puppetlabs/code/environments/$environment

On Puppet 3 with directory environments path was:

    /etc/puppet/environments/$environment

---

## Puppetfile

This file contains a list of external modules to deploy with the control-repo.

The source of these modules can be the Puppet Forge or a git repository.

Tools like [r10k](https://github.com/puppetlabs/r10k) take care of deploying these modules when the control-repo is deployed.

Syntax is as follows:

    # Latest version of module mount_core from puppetlabs author from the Forge
    mod 'puppetlabs/mount_core', :latest

    # Version 2.2.1 of module tp from example42
    mod 'example42/tp', '2.2.1'

    # Module network from a git source (author name here is redundant)
    mod 'example42/network',
    :git => 'https://github.com/example42/puppet-deployments'

    # Special syntax that defines a git source and uses the same branch of the control-repo (or 'production' if there's no branch on the module with the same name)
    mod 'example42/hieradata',
    :git => 'https://github.com/example42/psick-hieradata',
    :branch => :control_branch,
    :default_branch => 'production'

    # Specify a custom install directory where to deploy the module (Default is **modules/** relative to the main control-repo dir)
    mod 'secrets',
    :git => 'git@gitlab.internal:example/secrets.git',
    :install_path => 'private'

---

## hiera.yaml

This is the Hiera 5 configuration file for the environment layer.

This is usually the Hiera configuration file of reference and here we typically place the hierarchies and the backends where we want to store the data of our Puppet infrastructure.

Starting from Puppet 4.9, which introduces Hiera 5, Hiera has indeed 3 different **layers** where Hiera can be configured:

- **Global**, configured in **$confdir/hiera.yaml** (/etc/puppetlabs/puppet/hiera.yaml or C:\ProgramData\PuppetLabs\puppet\etc\hiera.yaml)
- **Environment**, this example, configured in **<ENVIRONMENT>/hiera.yaml** (/etc/puppetlabs/code/environments/production/hiera.yaml or C:\ProgramData\PuppetLabs\code\environments\production\hiera.yaml)
- **Module**. configured inside a module, **<MODULE>/hiera.yaml** (/etc/puppetlabs/code/environments/production/modules/ntp/hiera.yaml or C:\ProgramData\PuppetLabs\code\environments\production\modules\ntp\hiera.yaml)

Remember that Hiera parses these 3 different configuration files and the relevant Hierarchies in the above order, so if a value is define in a Hierarchy defined at the global level, even if a common.yaml one, this has precedence on any other hierarchy defined in the environment or module layer.

For this reason, usually the Global layer has an empty hiera.yaml (or, in PE, a configuration which uses PE as backend), the environment layers is where wee configure the hierarchies that match our specific infrastructure, and the module layer usually (at least in public modules) contains just hierarchies related to Operating systems and versions.

---

## hieradata or data

These are popular directory where Hiera data is usually placed. This has to be configured in the control repo's hiera.yaml and can be included in the same git repository or be placed in a separated one, configured in the Puppetfile.yaml

In the default hiera.yaml the datadir is configured as follows:

    ---
    version: 5
    defaults:
      datadir: data
      
    hierarchy:
    - name: "Yaml backend"
        data_hash: yaml_data
        paths:
        - "nodes/%{trusted.certname}.yaml"
        - "common.yaml"

The above configuration expects Hiera data files in **<ENVIRONMENT>/data/nodes/<NODENAME>.yaml** for each managed node, and **<ENVIRONMENT>/data/common.yaml** for common default settings.

---

## environment.conf


---

## modules