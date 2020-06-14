## Introduction to Hiera

Hiera is the **key/value lookup tool** of reference where to store Puppet user data.

It provides an highly customizable way to lookup for parameters values based on a custom hierarchy using many different backends for data storage.

It provides a command line tool ```hiera``` that we can use to interrogate direclty the Hiera data and functions to be used inside Puppet manifests: ```hiera()``` , ```hiera_array()``` , ```hiera_hash()``` , ```hiera_include()```

Hiera is installed by default with Puppet version 3 and is available as separated download on earlier version ([Installation instructions](http://docs.puppetlabs.com/hiera/1/installing.html)).

We need Hiera only on the PuppetMaster (or on any node, if we have a masterless setup).

Currently version 1 is available, here is its [official documentation](http://docs.puppetlabs.com/hiera/1/).

---

## Introduction to Hiera

Hiera is the most used solution to manage the data we need to configure our systems according to the logic we need.

It allows to separate data from code, allowing us to avoid to place inside our Puppet manifests information which is strictly related to our infrastructure and its settings.

Hiera has been integrated in starting from Puppet 3: for each class parameter Puppet does an automatic lookup on a corresponding Hiera value.

Since Puppet 4.9 a totally new version of Hiera (5) has been introduced, which introduces the possibility to use Hiera data directly in modules.

Hiera can use different backends to store it's data. The default one is Yaml (all data is placed in simple Yaml files). Other backend are Json, Mysql, Redis and so on.

Data is looked on Hiera following a hierarchy based on Puppet variables, in this way we can attribute to an Hiera Key (for example ```ntp::server```) different values according to different conditions (for example the role, the environment or the location of a server).

Inside Puppet code with can use the ```hiera()``` function (obsoleted by the ```lookup()``` function from Puppet 4.9) to look for a given Hiera key. For example the following code assign to the variable ```$server``` the value of the Hiera key called ```ntp_server```. An optional parameter can be added to set the default value (here ```ntp.pool.org```) to return if the key is not found on Hiera:

    $server = hiera('ntp_server','ntp.pool.org')

On modern Puppet versions this is equivalent to:

    $server = lookup('ntp_server',String,'first','ntp.pool.org')


Since Puppet 3 Puppet does an automatic lookup on Hiera for each parameter of a class. In the following example the parameter ```server``` (whose value can be referred, inside the same class, using the ```$server``` variable and, inside any other class with its *fully qualified name*: ```$::ntp::server```) can be changed by setting, on Hiera a value for the key called ```ntp::server```:

    class ntp (
      server = 'ntp.pool.org',
    ) { }

---

## Hiera configuration (old, pre Hiera 5)

Hiera 's configuration file for Puppet is ```/etc/puppet/hiera.yaml``` (On Puppet 4 ```/etc/puppetlabs/puppet/hiera.yaml```). It's a Yaml file which looks like this:

    ---
      :backends:
        - yaml

      :hierarchy:
         - "%{::environment}/nodes/%{::fqdn}"
         - "%{::environment}/roles/%{::role}"
         - "%{::environment}/zones/%{::zone}"
         - "%{::environment}/common"
       :yaml:
         :datadir: /etc/puppet/hieradata

In the above sample it's configured the **backend(s)** to use, the **hierarchy** based on Puppet variables (inside ```%{}```) and the configuration specific to the used backend (here is set the **datadir** when Yaml files containing Hiera keys are placed, named acconding to the defined hierarchy).

---

## Hierarchies

With the ```:hierarchy``` global setting we can define a string or an array of data sources which are checked in order, from top to bottom.

When the same key is present on different data sources by default is chosen the top one. We can override this setting with the ```:merge_behavior``` global configuration. Check [this page](http://docs.puppetlabs.com/hiera/1/lookup_types.html#deep-merging-in-hiera--120) for details.

In hierarchies we can interpolate variables with the %{} notation (variables interpolation is possible also in other parts of hiera.yaml and in the same data sources).

This is an example Hierarchy:

        ---
        :hierarchy:
          - "nodes/%{::clientcert}"
          - "roles/%{::role}"
          - "%{::osfamily}"
          - "%{::environment}"
          - common

Note that the referenced variables should be expressed with their fully qualified name. They are generally facts or Puppet's top scope variables (in the above example, **$::role** is not a standard fact, but is generally useful to have it (or a similar variable that identifies the kind of server) in our hierarchy).

If we have more backends, for each backend is evaluated the full hierarchy.

We can find some real world hierarchies samples in this [Puppet Users Group post](https://groups.google.com/forum/?hl=it#!topic/puppet-users/cCfimbolUio)

More information on hierarchies [here](http://docs.puppetlabs.com/hiera/1/hierarchy.html).

---

## Puppet data bindings
Starting from Puppet version 3 Hiera is shipped directly with Puppet and an automatic hiera lookup is done for each class' parameter using the key **$class::$argument**: this functionality is called data bindings or automatic parameter lookup.

For example in a class definition like:

    @@@puppet
    class openssh (
      template = undef,
    ) { . . . }

Puppet 3 automatically looks for the Hiera key `openssh::template` if no value is explicitly set when declaring the class.

To emulate a similar behaviour on pre Puppet 3 we should write something like:

    @@@puppet
    class openssh (
      template = hiera("openssh::template"),
    ) { . . . }

If a default value is set for an argument, that value is used only when user has not explicitly declared avalue for that argument and Hiera automatic lookup for that argument doesn't return any value.

In modern Puppet setups data bindings is widely used to configure the classes included (without parameters declaration) via any classification method.

---

## Hiera configuration (new, starting from Hiera 5 - Puppet 4.9)

Starting from Hiera version 5 (available from Puppet version 4.9) hiera.yaml configuration file can be present in [3 different layers](https://puppet.com/docs/puppet/5.5/hiera_intro.html#ariaid-title3):

- Global layer: ```/etc/puppetlabs/puppet/hiera.yaml```. Equivalent to the single hiera.yaml in older versions.
- Environment layer: ```/etc/puppetlabs/code/environments/<environment>/hiera.yaml```. Inside each Puppet environment directory
- Module layer ```hiera.yaml```inside each module.

The hierarchies defined in each of these files are traversed in the same order: first all the ones in the global layer, then the ones in the environment layer and then the ones in module layer.

This means that a common.yaml defined in ```/etc/puppetlabs/puppet/hiera.yaml``` would override any more specific, variable dependent, hierarchy level in environment or module layers.

For this reason when using Hiera 5 layes are geenrally used as follows:

- Global layer is disabled (empty or missing hiera.yaml), used to set to global overrides, which "win" over everything, or used for special backends (like, as default on PE, the PE console)

- Environment layer is used for the most common data, where infrastructure related settings are placed

- Module layer is used generally by module authors, for OS related settings.

The format of hiera.yaml is slightly different, a sample one might look as follows:

    version: 5
    hierarchy:
      - name: "Hierarchy"
        paths:
        - "nodes/%{::fqdn}.yaml"
        - "roles/%{::role}.yaml"
        - "zones/%{::zone}.yaml"
        - "common.yaml"
    defaults:
      data_hash: yaml_data
      datadir: hieradata

If the above were an environment hiera.yaml, the data directory would be relative to the environment: ```/etc/puppetlabs/code/environments/<environment>/hieradata/```.

---

## Hiera Backends

One powerful feature of Hiera is that the actual key-value data can be retrieved from different backends.

With the ```:backends``` global configuration we define which backends to use, then, for each used backend we can specify backend specific settings.

### Build it backends:

**yaml** - Data is stored in yaml files (in the ```:datadir``` directory)

**json** - Data is stored in json files (in the ```:datadir``` directory)

**puppet** - Data is defined in Puppet (in the ```:datasouce``` class)

### Extra backends:
Many additional backends are available, the most interesting ones are:

[**gpg**](https://github.com/crayfishx/hiera-gpg) - Data is stored in GPG encripted yaml files

[**http**](https://github.com/crayfishx/hiera-http) - Data is retrieved from a REST service

[**mysql**](https://github.com/crayfishx/hiera-mysql) - Data is retrieved from a Mysql database

[**redis**](https://github.com/reliantsecurity/hiera-redis) - Data is retrieved from a Redis database

### Custom backends
It's relatively easy to write custom backends for Hiera. Here are some [development instructions](http://docs.puppetlabs.com/hiera/1/custom_backends.html)

---

## Using Hiera in Puppet

The data stored in Hiera can be retrieved by the PuppetMaster while compiling the catalog using the legacy (now deprecated) `hiera()` function or the more modern `lookup()` one.

In our manifests we can have something like this:

    # Deprecated Hiera function for direct lookup
    $my_dns_servers = hiera('dns_servers')

    # Current lookup function alternative
    $my_dns_servers = lookup('dns_servers')

Both of them assign to the variable ***$my_dns_servers*** (can have any name) the first value retrieved by Hiera for the key ***dns_servers***

We may prefer, in some cases, to retrieve all the values retrieved in the hierarchy's data sources of a given key and not the first use. If the expected data is an array we can use hiera_array() for that (deprecated) or specify the unique merge method in the lookup() function.

    # Deprecated Hiera function for merging arrays lookup
    $my_dns_servers = hiera_array('dns_servers')

    # Current lookup function alternative with unique merge method
    $my_dns_servers =  lookup('ntp::servers', {merge => unique})

If we expect an hash as value for a given key we can use the hiera() legacy function or lookup() with the default "first" lookup method to retrieve the first value found in the hierarchies, but, if we want to merge the hash values found in all the hierarchy levels in a single hash  we can use the legacy hiera_hash() function or the deep merge method in lookup():

    # Deprecated Hiera function for merging hashes lookup
    $openssh_settings = hiera_hash('openssh_settings')

    # Current lookup function alternative with merge methods hash or deep
    $openssh_settings = lookup('openssh_settings', {merge => hash}) # Normal hash merge
    $openssh_settings = lookup('openssh_settings', {merge => deep}) # Deep hash merge

The difference between hash and deep merge methods is that, when exist the same subkeys in hash looked up via Hiera, with deep merge the subkeys values are merged recursively, with normal hash merge the first one is the returned value.

Finally it's possible to include classes via Hiera, this is the so called Hiera driven classification.

In these examples, the key looked for ('classes') contain an array of the names of the classes to include (merged across the hierarchies):

    # Deprecated Hiera_ function for including classes
    hiera_include('classes')
    
    # Alternative in modern Puppet
    lookup('classes', {merge => unique}).include

---

## Using hiera from the command line

Hiera can be invoken via the command line to interrogate the given key's value:

    hiera dns_servers

This will return the default value as no node's specific information is provided. More useful is to provide the whole facts' yaml of a node, so that the returned value can be based on the dynamic values of the hierarchy.
On the Pupppet Masters the facts of all the managed clients are collected in $vardir/yaml/facts so this is the best place to see how Hiera evaluates keys for different clients:

    hiera dns_servers --yaml /var/lib/puppet/yaml/facts/<node>.yaml

We can also pass variables useful to test the hierarchy, directly from the command line:

    hiera ntp_servers operatingsystem=Centos
    hiera ntp_servers operatingsystem=Centos hostname=jenkins

To have a deeper insight of Hiera operations use the debug (-d) option:

    hiera dns_servers -d

To make an hiera array lookup (equivalent to hiera_array()):

    hiera dns_servers -a

To make an hiera hash lookup (equivalent to hiera_hash()):

    hiera openssh::settings -h

---

