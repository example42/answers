## Puppet Enterprise troubleshooting

In order to have the basic information for troubleshooting Puppet Enterprise components first read the sections about [PE services](pe_services.md) amd [PE Logs](pe_logs.md).

They contain useful information about location of log files and configurations, ports used, service processes and relations between PE components.

Here we are going to provide some further details on troubleshooting the various PE components.

Upstream documentation is available at https://puppet.com/docs/pe/2019.0/troubleshooting.html

### Puppet Server

This service is essential for Puppet operations, its main log file, **/var/log/puppetlabs/puppetserver/puppetserver.log**, is the first place to look for when we have issues.

Generally they might be related to:

  - pe-puppetserver service startup
  - reachability problems between clients and server
  - problems in deploying Puppet code and data
  - problems in compiling catalogs and applying them

Before getting into each service troubleshooting details, in case of problems check also common possible system's problems:

  - Disk space available on all the system's mount points (**df**, **mount** ...)
  - Systems has enough memory (**free**, **top**... )
  - System CPU usage is sustainable (**top**, **vmstat**...)
  - If SELinux is enabled check its logs for eventual blocked actions (**sestatus**, **tail -f /va/log/audit/audit.log**)
  - Networking and name resolution works correctly (**telnet**, **ping**, **dig**...)
  - Local or remote firewall rules preventing access to services (**iptables-save**, remote **telnet**)

#### pe-puppetserver service startup

If Puppet server has problems in starting up and is not able to reply to clients, check the above log during a service startup. As root on the Puppet Server run:

    tail -f /var/log/puppetlabs/puppetserver/puppetserver.log

On another terminal, for better visibility on what happens in what moment, restart the puppetserver service (note that during restart, which may take a few seconds, the service is not available to clients):

    systemctl stop pe-puppetserver
    systemctl start pe-puppetserver

Possible things to do:

  - Check output of systemctl status pe-puppetserver
  - If there are stack traces in the logs, look at the beginning of the trace message for having an idea on where the problem originates from.
  - Check if file referenced in configurations exist and are readable by the pe-puppetserver user
  - Improve logs verbosity by editing the Logback xml files referenced below.

Relevant configuration files:

  - **/etc/puppetlabs/puppet/puppet.conf** (Main Puppet config file, both for agent and master. Note that the master section here which was used by the old puppet master component in ruby is honoured by the Puppet Server application and coupled with what is configured under **/etc/puppetlabs/puppetserver/**)
  - **/etc/puppetlabs/puppetserver/bootstrap.cfg** (the list of subservices that are loaded with Puppet Server, this is NOT supposed to be changed, as some functionality might be lost, but can be useful to isolate the service that creates blocking startup problems by commenting the relevant line and trying a restart)
  - **/etc/puppetlabs/puppetserver/*.xml** files define how logs are configured via Logback, we may edit them to improve debug levels on logs
  - **/etc/puppetlabs/puppetserver/conf.d/global.conf** (the global conf file, with paths for certs)
  - **/etc/puppetlabs/puppetserver/conf.d/pe-puppet-server.conf** (other general settings for the Puppet server)
  - **/etc/puppetlabs/puppetserver/conf.d/webserver.conf** (configuration of the webserver part)

#### Code deployment problems

Code Manager is the subsystem of the Puppet Server service that takes care of deploying Puppet Code from the Git server to the directory **/etc/puppetlabs/code/environments** on the Puppet Server (here stays the code and the data used by Puppet when compiling catalogs for clients).

If it fails code changes pushed to the Git Server are not correctly deployed on the Puppet Servers.

Possible things to do:

  - Always verify the actual status of the deployed code on the **/etc/puppetlabs/code/environments** directory of the Puppet Servers. Here there is a different subdirectory (with the name of the relevant Puppet environment) which matches the branches of the control-repo. Check if contents of deployed files are current.
  - Trigger a manual code deployment, while keeping an eye to the puppetserver.log, with a command like:

        /opt/puppetlabs/bin/puppet code deploy <environment>

    Replace <environment> with the name of the Puppet environment (and git branch on the control-repo) to deploy.
    For example to deploy the production environment in the /etc/puppetlabs/code/environments/production directory run

        /opt/puppetlabs/bin/puppet code deploy production

  - Check code manages and file sync services functionality with the command:

        puppet-code status

Relevant configuration files:

  - **/etc/puppetlabs/puppetserver/conf.d/code-manager.conf** (Code Manager configuration)
  - **/etc/puppetlabs/r10k/r10k.yaml** (configuration of r10k, the tool used, under the hoods, by Code Manager to deploy a control repo)
  - **/etc/puppetlabs/puppetserver/conf.d/file-sync.conf** (configures the File Sync component, which ensures that code is correctly deployed on Master and Replica servers)

Refer to https://puppet.com/docs/pe/2019.0/codemgrtroubleshoot-1.html for more details.

#### Problems in accessing Puppet Server

Puppet agents may have problems in reaching the Puppet Server or in obtaining a valid catalog check.

Possible things to do:

  - Check if the puppet server(s) configured on clients (entry server or server_list in /etc/puppetlabs/puppet/puppet.conf) is resolved correclty from the clients

  - Check if client(s) can reach the server names on port 8140 from the client. If not check:
    
    - If port 8140 is listening on the Server on the interface used for clients' access
    - That server has no blocking local inbound firewall rules for port 8140
    - That the client no blocking local outbound firewall rules to server's port 8140
    - Routing between client and server
    - Ensure that any eventual firewall between client(s) and server allow TCP port 8140 traffic (also port 8142 from clients to server should be open, it's used for Puppet orchestration, strictly speaking is not required but is nice to have to be able to trigger remote jobs)

  - If port 8140 is reachable but the client can't fetch a catalog for SSL certificates issues check:
    
    - That times are aligned to the second on Master and clients
    - If the client certificate has been accepted on the server
    - If the client certificate is not valid (it might have been recreated and the new one doesn't match the one accepted on the server)

Note that starting from Puppet version 6 CA is managed via the puppetserver ca command, a lot of reference online still mentions the old puppet cert commands.

You can use puppetserver ca commands to sign, clean or list client certificates.

Refer to https://puppet.com/docs/puppet/6.1/ssl_certificates.html for more details on Puppet Certificates management.

Refer to https://puppet.com/docs/pe/2019.0/troubleshootingcommunicationsbetween_components.html for more details on thoubleshooting communication between Puppet components.

Relevant configuration files:

  - **/etc/puppetlabs/puppetserver/conf.d/auth.conf** (defines access lists to the Puppet Server API endpoints)
  - **/etc/puppetlabs/puppetserver/conf.d/ca.conf** (the CA conf file)
  - **/etc/puppetlabs/puppet/autosign.*** (autosign files in case autosign is configured)

#### Errors while trying to fetch or apply a catalog

The catalog is a list of resources to be applied on a client. It's generated on the Server according to the classes included for that client, their Puppet code, the relevant Hiera data and the clients' facts.

When a client is able to communicate with the server (port 8140 is reachable and SSL certs are signed) different kind of errors may take place when trying to fetch its catalog and applying it:

  - Catalog compilation errors
  - Dependency loops in catalogs
  - Errors in applying specific resources of a catalog

These errors may take place for all the clients or only a subset of them, according to the number of nodes that use the classes which create problems.

To fix them we have to work on the Puppet code and Hiera data, not on the Puppet Server services.

Generally we can see evidence of such problems:

  - In puppetserver.log
  - On clients, when running puppet agent -t from the command line
  - On the PE Console web interface

The files to start to analyse in order to fix such errors are, in most of the cases, not anymore Puppet Enterprise configuration files but rather the files that the Puppet Server uses to compile a catalog.

So, to have an approximate idea, in order to sort out issues related to missing or wrong Hiera data, we can check syntax and content of:

  - **/etc/puppetlabs/puppet/hiera.yaml** (The global hiera configuration file. It should have default contents with the classifier_data backend, which allows to set Hiera values on PE Web Console)
  - **/etc/puppetlabs/code/environments/<environment>/hiera.yaml** (the environment Hiera configration file, where our hierarchies are defined)
  - **/etc/puppetlabs/code/environments/<environment>/modules/%{product_module}/hiera** (hiera data set in product modules)
  - **/etc/puppetlabs/code/environments/<environment>/modules/hiera** (hiera data set in hiera module)
  - Eventual configurations done on the PE Console Nodes Classification interface

In order to troubleshoot errors in Puppet code check:

  - **/etc/puppetlabs/code/environments/<environment>/manifests/site.pp** (the very first manifest parsed by thew Puppet Server when it starts to compile a client's catalog. Consider it the init of Puppet, everything starts from there.)
  - All the classes included by a node, as set via the classes Hiera key. These classes are all placed in modules which may stay in one of these directories:
    
    - **/etc/puppetlabs/code/environments/<environment>/modules**
    - **/etc/puppetlabs/code/environments/<environment>/site**
    - **/etc/puppetlabs/code/environments/<environment>/legacy/puppet/modules/shared**
    - **/etc/puppetlabs/code/environments/<environment>/legacy/puppet/modules/products/\***

Of course we don't need to check all these files every time, we can refer to the specific error messages for insights on where's the problem.

#### Catalog compilation errors

They may be due to various reasons:

  - Syntax errors in manifests used by classes included by the client (such errors should not pass CI tests and can be tested locally with the puppet parser validate <manifest.pp> command)
  - Syntax errors in Hiera yaml files
  - Missing data in Hiera (required parameters value are not found on Hiera)
  - Duplicated resource errors (the same resource is declared more than once in different manifests used by a client)
  - Wrong reference to file sources and paths

In Puppet logs and reports it should be clearly defined the file, the line number and even the column where there's the problem, refer to them to identify and fix the issue.


#### Dependency loops in catalogs

These errors happen when a Puppet catalog can't be applied because there's a look in the resources dependency graph in the catalog.

Note that such an errors appears on the client when it tries to apply a catalog successful compiled and retrieved from the Master.

In order to troubleshoot them can be useful the "View Node Graph" link in the top left corner or a client detail page on the PE console.

#### Errors in applying specific resources of a catalog

There errors happen when Puppet tries, and fails, to apply one or more resources defined in the catalog for a client.

Contrary to the previous categories of errors, which totally prevent Puppet from applying a catalog, these errors involve just a subset of the managed resources: the failing ones and the ones which depend on them.

Also in these cases the solution involves working on the Puppet code and data, and not on the Infrastructure.

It's relatively normal (even if far from being optimal), especially in the current Puppet setup, to have some resources failing on some nodes. To have better insights on the number of nodes impacted and the source of the issue check:

  - The error messages in the Puppet reports (either on the client or on the web console)
  - The Inspect / Events interface on the Web console where it's possible to have an overview of classes / nodes and resources which fail or have changes in the entire infrastructure.


### Puppet DB and PostgreSQL

To troubleshoot errors with PuppetDB service start from **/var/log/puppetlabs/puppetdb/puppetdb.log**, eventually increase logging level in **/etc/puppetlabs/puppetdb/logback.xml**:

    tail -f /var/log/puppetlabs/puppetdb/puppetdb.log

On another terminal restart the puppetdb service (note that during restart, which may take a few seconds, the Puppet Server can't compile catalogs for clients or submit their facts and reports):

    systemctl stop pe-puppetdb
    systemctl start pe-puppetdb

Relevant configuration files:

  - **/etc/puppetlabs/puppetdb/bootstrap.cfg** (the list of subservices that are loaded with PuppetDB, this is NOT supposed to be changed)
  - **/etc/puppetlabs/puppetdb/*.xml** files define how PuppetDB logs are configured via Logback
  - **/etc/puppetlabs/puppetdb/conf.d/config.ini** (some global settings)
  - **/etc/puppetlabs/puppetdb/conf.d/database.ini** (database connection settings, be sure they are correct to access the PostgreSQL server)
  - **/etc/puppetlabs/puppetdb/conf.d/jetty.ini** (the configuration of PuppetDB web server (used for API access))
  - **/etc/puppetlabs/puppetdb/certificate-whitelist** (the list of nodes who can connect to PuppetDB)
  - **/etc/puppetlabs/puppet/puppetdb.conf** (the configuration for Puppet Server where is defined how to reach PuppetDB)

For more info on troubleshooting PostgreSQL database check online documentation (PostgreSQL Version used on PE 2019.0 is 9.6) or this Puppet specific notes: https://puppet.com/docs/pe/2019.0/troubleshootingthedatabases.html


### Console Services

To troubleshoot errors with Console Services service start from /var/log/puppetlabs/console-services/console-services.log, eventually increase logging level in /etc/puppetlabs/console-services/logback.xml :

    tail -f /var/log/puppetlabs/console-services/console-services.log

On another terminal restart the console-services service (note that during restart, which may take a few seconds, the PE Console web interfaace is not visible to users):

    systemctl stop pe-console-services
    systemctl start pe-console-services

The web server of Console services is proxied by an Nginx service, to check its logs:

    tail -f /var/log/puppetlabs/nginx/error.log
    tail -f /var/log/puppetlabs/nginx/access.log

To restart the NGINX reverse proxy:

    systemctl stop pe-nginx
    systemctl start pe-nginx

Relevant configuration files:

  - **/etc/puppetlabs/console-services/bootstrap.cfg** (the list of subservices that are loaded with console-services , this is NOT supposed to be changed)
  - **/etc/puppetlabs/console-services/*.xml** files define how ogs are configured via Logback
  - **/etc/puppetlabs/console-services/conf.d/cowebserver.conf** (configurations of the jetty web server)
  - **/etc/puppetlabs/console-services/conf.d/*-database.conf** (configuration files for the 3 databases used by the console services (activity, classifier and rbac)
  - **/etc/puppetlabs/nginx/conf.d/proxy.conf** (the nginx configuration for the reverse proxy)

### Reset admin password

PE Console authenticates users using local or Active Directory users, but there's a local admin account with administrative privileges.

In case its password is lost, it can be reset by running on the system:

    sudo /opt/puppetlabs/server/bin/set_console_admin_password.rb <NEW_PASSWORD>

For more information about Console Services check the online documentation

### Installation

The outputs of the installation process are placed in **/var/log/puppetlabs/installer/**

The file **/etc/puppetlabs/enterprise/conf.d/pe.conf** contains the parameters defined by users during the installation. It may be used to reinstall the system in an automated way and it's used in the upgrade process.

NOTE: It contains in clear text the admin password set during installation, if it hasn't been changed it's still valid.

For details on how to troubleshoot the installation and upgrade process refer to https://puppet.com/docs/pe/2019.0/troubleshootingthepe_installer.html

