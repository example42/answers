## Managing files

---

## Managing packages

Installation of packages is managed by the **package** type.

The main arguments:

    package { 'apache':
      name      => 'httpd',  # (namevar)
      ensure    => 'present' # Values: 'absent', 'latest', '2.2.1'
      provider  => undef,    # Force an explicit provider
    }

---

## Managing services

Management of services is via the **service** type.

The main arguments:

    service { 'apache':
      name      => 'httpd',  # (namevar)
      ensure    => 'running' # Values: 'stopped', 'running'
      enable    => true,     # Define if to enable service at boot (true|false)
      hasstatus => true,     # Whether to use the init script' status to check
                             # if the service is running.
      pattern   => 'httpd',  # Name of the process to look for when hasstatus=false
    }


---

## Executing commands

We can run plain commands using Puppet's **exec** type. Since Puppet applies it at every run, either the command can be safely run multiple times or we have to use one of the **creates**, **unless**, **onlyif**, **refreshonly** arguments to manage when to execute it.

    exec { 'get_my_file':
        command => 'wget http://mysite/myfile.tar.gz -O /tmp/myfile.tar.gz',
        path    => '/sbin:/bin:/usr/sbin:/usr/bin',
        creates => '/tmp/myfile.tar.gz', 
        onlyif  => 'ls /tmp/myfile.tar.gz && false',
        unless  => 'ls /tmp/myfile.tar.gz',
        user    => 'my_user',
    }

The parameters are as follows:

- **command**, (namevar, if not specified, title is used) The actual command to execute
- **path**, the search paths where to look for the command, if not set, command must have an absolute path
- **creates**, a method to test if to run command, it refers to a file created by the command. It if exists, the command is not executed
- **onlyif**, alternative test method, if command here  returns an error the main command IS NOT executed
- **unless**, alternative test method, if command here  returns an error the main command IS executed
- **user**, which user the command should be run as (default root)

---

## Managing files: Template + Options hash

Check this [blog post](http://www.example42.com/2014/10/29/reusability-features-every-module-should-have/) for details.

Classes and defines should expose parameters that allow to override the used templates and set a custom hash of configurations.

    class redis (
      $config_file_template = 'redis/redis.conf.erb',
      $options_hash         = {},
    ) {
      file { '/etc/redis/redis.conf':
        content => template($config_file_template),
      }
    }

Given the previous class definition, we can configure it with this sample Hiera data, in YAML format:

    redis::config_file_template: 'site/redis/redis.conf.erb'
    redis::options_hash:
      port: '12312'
      bind: '0.0.0.0'
      masterip: '10.0.42.50'
      masterport: '12350'
      slave: true

The referenced template stays in our site module, in ```$modulepath/site/templates/redis/redis.conf.erb``` and may look like:

    port <%= @options_hash['port'] %>
    bind <%= @options_hash['bind'] %>
    <% if @options_hash['slave'] == true -%>
    slaveof <%= @options_hash['masterip'] %> <%= @options_hash['masterport'] %>
    <% end -%>
