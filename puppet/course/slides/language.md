## Conditionals

Puppet provides different constructs to manage conditionals inside manifests:

**Selectors** allows to set the value of a variable or an argument inside a resource declaration according to the value of another variable.
Selectors therefore just returns values and are not used to manage conditionally entire blocks of code.

**case statements** are used to execute different blocks of code according to the values of a variable. It's possible to have a default block for unmatched entries.
Case statements are NOT used inside resource declarations.

**if elsif else** conditionals, like case, are used to execute different blocks of code and can't be used inside resources declarations.
We can can use any of Puppet's comparison expressions and we can combine more than one for complex patterns matching.

**unless** is somehow the opposite of **if**. It evaluates a boolean condition and if it's *false* it executes a block of code. It doesn't have elsif / else clauses.

---

## Variables

Variables is Puppet codes are basically **constants**: once defined in a class we can't change them.

We can set variables in our Puppet code with this syntax:

    # Normal variable assignment
    $role = 'mail'

    # The value of a variable is based on another variable (here used the **selector** costruct)
    $package = $::operatingsystem ? {
      /(?i:Ubuntu|Debian|Mint)/ => 'apache2',
      default                   => 'httpd',
    }

Puppet automatically provides also some **internal** variables, the most common are:

- The name of the node (the certname setting in its puppet.conf)

        $clientcert # Default is the client's Fully Qualified Domain Name)

- The Puppet's environment where the Master looks for the code to compile

        $environment # Default is "production"

- The Master's FQDN and IP address

        $servername $serverip

-  Any configuration setting of the Puppet Master's puppet.conf

        $settings::<setting_name>:

-  The name of the module that contains the current resource's definition

        $module_name

---

## Facter and facts

**Facter** is a tools shipped with Puppet. It runs on clients and collects **facts** which are sent to the server. In our code we can use facts to manage resources in different ways or with different arguments.


Here follows a list of the most common and useful facts:

    al$ facter

    architecture => x86_64
    fqdn => Macante.example42.com
    hostname => Macante
    ipaddress_eth0 => 10.42.42.98
    macaddress_eth0 => 20:c9:d0:44:61:57
    operatingsystem => Centos
    operatingsystemrelease => 6.3
    osfamily => RedHat
    virtual => physical

It's easy to create custom facts. They can be of 2 types:

- **Native facts** written in ruby and shipped with modules (in the ```lib/facter``` directory)
- **External facts** can be simple ini-file like texts (with ```.txt``` extension), Yaml files or even commands in any language, which returns a fact name and its value. External faccts are located in the nodes' ```/etc/facter/facts.d``` directory and can be shipped also from modules (in the ```facts.d``` directory)

---

## Built-in variables

Puppet provides some useful built in variables, they can be:

#### Set by the client (agent)

- ```$clientcert``` - the name of the node's certificate. By default its ```$::fqdn```
- ```$clientversion``` - the Puppet version on the client

#### Set by the server (master)

- ```$environment``` (default: production) - the Puppet environment where are placed modules and manifests.
- ```$servername```, ```$serverip``` - the Puppet Master FQDN and IP
- ```$serverversion``` - the Puppet version on the server
- ```$settings::<name>``` - any configuration setting on the Master's ```puppet.conf```

#### Set by the server during catalog compilation

- ```$module_name``` - the name of the module that contains the current resource's definition
- ```$caller_module_name``` - the name of the module that contains the current resource's declaration
