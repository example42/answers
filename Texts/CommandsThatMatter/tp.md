# example42 Puppet works

## Junior

### Install Tiny Puppet

    # Install example42 Tiny Puppet module
    puppet module install example42-tp
    
    # Install tp command on local node(*)
    puppet tp setup
    
    # List available applications
    tp list
    
    # Install an application (any APP on any OS!) (*) (**)
    tp install app
    
    # Test all applications installed with tp
    tp test
    
    # Show logs of one or all the applications
    tp log [app]

    (*)  Need superpowers
    (**) May need updated tinydata