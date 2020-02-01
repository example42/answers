# Commands That Matter: Puppet

## Beginner 


## Junior

### Puppet sub commands

    Help Yourself
    puppet help

    # Show system facts
    # Equivalent of facter
    puppet facts
        
    # Run agent in enforcing or safe mode
    puppet agent -t
    puppet agent -t --noop
        
    # Search and then install a module (author-modulename)
    puppet module search chocolatey
    puppet module install puppetlabs-chocolatey
        
    # Execute directly Puppet code in the argument or from a manifest file
    puppet apply -e "include chocolatey"
    puppet apply manifest.pp

    # Show system resources which can be represented in Puppet language

    puppet resource --types

    # Query all or a specific type of resource
    
    puppet resource package
    puppet resource package nginx

    # Show the puppet configuration for local user
    puppet config print all

# Commonly used arguments for most of these commands
# permits to specify where to look for modules or what
# Puppet environment to use

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


## Advanced

### Puppet sub command

    Usage: puppet <subcommand> [options] <action> [options]

    # Show system facts and upload them to Puppet Master
    puppet facts
    puppet facts upload

    # Run puppet agent in debug mode
    puppet agent -t --debug

    # Testing Puppet agent on a node where it was disabled, and keep it disabled
    puppet agent --enable ; puppet agent -t --noop ; puppet agent --disable "Puppet disabled $(date) for Manual maintenance"
        
       
    # Execute directly Puppet code 
    puppet apply -e "include chocolatey"

    # Apply the main site.pp manifest of a control-repo, using its modulepath and hiera config
    puppet apply manifests/site.pp --modulepath "site:modules" --environmentpath "$(dirname $0)/.." --hiera_config="$(dirname $0)/hiera.yaml"

    # Apply a JSON catalog, previously generated by a Master
    puppet apply --catalog $(puppet config print client_datadir)/catalog/$(facter fqdn).json
    
    # Interact with system resources. Show and change single resources
    puppet resource --types
    puppet resource package
    puppet resource package nginx
    puppet resource package nginx ensure=absent

    # Show the content of running user's puppet.conf
    cat "$(puppet config print confdir)/$(puppet config print config_file_name)"

    # Show and set a specific config entry, in the given puppet.conf's section
    puppet config print noop --section agent
    puppet config set noop true --section agent




### Puppet module

    # List installed modules, search, install, upgrade and remove a module
    puppet module list
    puppet module search chocolatey
    puppet module install puppetlabs-chocolatey
    puppet module upgrade puppetlabs-chocolatey
    puppet module uninstall puppetlabs-chocolatey
