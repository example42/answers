# Commands That Matter: Puppet

## Beginner 


## Junior

### Puppet sub commands

Help Yourself

    puppet help

Show system facts (equivalent of facter)

    puppet facts
    
Run agent in enforcing or safe mode

    puppet agent -t
    puppet agent -t --noop
    
Install a module (author-module_name)

    puppet module install puppetlabs-chocolatey
    
Apply Puppet code present in the given manifest file

    puppet apply manifest.pp

Apply directly the Puppet code  argument

    puppet apply -e "include chocolatey"
    
Show the available system resources which can be represented in Puppet language

    puppet resource --types

Query a specific resource, showing in Puppet language the current status of all or a specific one

    puppet resource package
    puppet resource package nginx

Show the puppet configuration for the user

    puppet config print all

Commonly used arguments for most of these commands are the ones that permits to specify where to look for modules or the usage of a specific environment:

    --modulepath=modules:site
    --environment=testing




### Puppet module

    # Help Yourself
    puppet help module

    # Search for something on the Puppet Modules Forge
    puppet module search 

    # Install a module
    puppet module install puppetlabs-docker

    # List where modules are installed
    puppet module list

    # Upgrade a module
    puppet module upgrade puppetlabs-docker

    # Remove a module
    puppet module uninstall puppetlabs-docker 


### Using PDK

    # Help Yourself
    pdk help

    # Create a new Puppet module
    pdk new module profile

    # Create a new Puppet module based on custom templates
    pdk new module profile --template-url=value

    # Always work in the module dir, create the main class
    cd profile
    pdk new class profile

    # Validate module syntax, any time you want
    pdk validate

    # Run unit tests on the module
    pdk test unit 


### Certificate signing

By default the first Puppet agent run on a client fails because its SSL certificate (used for all the HTTPS communications with the server) is not yet signed on the server's Certication Authority (CA):

    client # puppet agent -t
        # "Exiting; no certificate found and waitforcert is disabled"

The server has received the client's Certificate Signing Request (CSR) has to be manually signed:

    # On Puppet version less than 6
    server # puppet cert sign <certname>  
     
    # On Puppet version le6 or more
    server # puppetserver ca sign --certname <certname> 
    
Once signed on the Master, the client can connect and receive its catalog:

    client # puppet agent -t


## Intermediate

### Puppet certificates signing

A SSL Certificate Signing Request is automatically created on each Puppet client the first time Puppet agent is run.

This has to be signed on the Puppet CA server (the Puppet Master).

Once signed the certificate univocally identifies the node and can't be changed (unless cleaned and recreated).

    # Find the certificate files are stored
    puppet config print ssldir

Client commands (Puppet >= 6)

    # Initial CSR creation (done automatically when puppet agent is run first)
    puppet ssl bootstrap

    # Verify certs status (both client and CA server ones)
    puppet ssl verify

    # Clean up a local certificate (NOTE: You must clean the same cert also on the server)
    puppet ssl clean

    # Clean up local certificate AND CA certificate and CRL bundle
    puppet ssl clean --localca

Server commands (both for Puppet < and >= 6)

    # List the (client) certificates to sign:
    puppet cert list                            # Puppet <  6
    puppetserver ca list                        # Puppet >= 6

    # List all certificates: signed (+), revoked (-), to sign ( ):
    puppet cert list --all                      # Puppet <  6
    puppetserver ca list --all                  # Puppet >= 6

    # Sign a client's certificate:
    puppet cert sign <certname>                 # Puppet <  6
    puppetserver ca sign --certname <certname>  # Puppet >= 6

    # Remove a client certificate:
    puppet cert clean <certname>                # Puppet <  6
    puppetserver ca clean --certname <certname> # Puppet >= 6
