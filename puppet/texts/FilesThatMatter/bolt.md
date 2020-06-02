# Files that matter: Bolt

## Beginner 

## Junior

A Bolt project directory contains all the configuration, code and data used by Bolt.

Three types of project directory:

- Local project directory. Any directory containing a bolt.yaml file.
   
- Embedded project directory. Any directory containing subdirectory called Boltdir.

- User project directory. If no bolt.yaml or Boltdir is found in bolt cwd, the user's
  ~/.puppetlabs/bolt is used as project directory

A project directory contains: 

bolt.yaml	      # General configuration options for Bolt.
hiera.yaml	    # Hiera config to use when using bolt apply.
inventory.yaml	# The list of known targets and relevant specific data like credentials.
Puppetfile	    # List of external modules to install with "bolt puppetfile install".
modules/	      # The directory where modules from the Puppetfile are installed. To .gitignore and not edit.
site-modules/   # Local modules that are edited and versioned with the project directory.
data/           # Default path where Hiera data files are stored.

Hint: Looks similar to a Puppet control-repo? Yes it does.
ProTip: Place bolt.yaml and an inventory.yaml in your control-repo and you
have made it a fully integrated (same hiera data, same modules) Bolt project.

Project dir can be specified in this order:

- Manually, with the command line option --boltdir <DIRECTORY_PATH>
- Bolt traverses parents of the current directory until it finds a directory containing a Boltdir or bolt.yaml.
- If no directory is specified manually or found in a parent directory, the user project directory is used.


## Intermediate

### Bolt configuration file

Contains global configuration options, most useful are:

- concurrency: The number of parallel actions to perform on remote targets. Default: 100.
- format: The output format. Possible values: human and json. Default: human.
- hiera-config: The path to Hiera config . Default is hiera.yaml inside the Bolt project directory.
- inventoryfile: The path to the inventory file where groups of targets are defined. Default is inventory.yaml inside the Bolt project directory.
- modulepath: The module path where modules with tasks and plans live. Default path is modules:site-modules:site inside the Bolt project directory.
- puppetfile: The path of Puppetfile
- save-rerun: Specify whether to update .rerun.json in the Bolt project directory. 
- transport: Specify the default transport to use when the transport for a target is not specified in the URL or inventory. Options are docker, local, pcp, ssh, and winrm.

SSH transport configuration options:

- connect-timeout: How long Bolt waits when establishing connections.
- disconnect-timeout: How long Bolt waits to force-close an SSH connection.
- host-key-check: Whether to perform host key validation when connecting over SSH. Default is true.
- password: Login password.
- port: Connection port. Default is 22.
- private-key: Either the path to the private key file to use for SSH authentication, or a hash with key key-data and the contents of the private key.
- proxyjump: A jump host to proxy SSH connections through, and an optional user to connect with, for example: jump.example.com or user1@jump.example.com.
- run-as: A different user to run commands as after login.
- run-as-command: The command to elevate permissions. Bolt appends the user and command strings to the configured run as a command before running it on the target. This command must not require an interactive password prompt.
- sudo-executable: The executable to use when escalating to the configured run-as user. This is useful when you want to escalate using the configured sudo-password, since run-as-command does not use sudo-password or support prompting. The command executed on the target is <sudo-executable> -S -u <user> -p custom_bolt_prompt <command>. Note: This option is experimental.

    sudo-password: Password to use when changing users via run-as.

    tmpdir: The directory to upload and execute temporary files on the target.

    script-dir: The subdirectory of the tmpdir to use in place of a randomized subdirectory for uploading and executing temporary files on the target. It's expected that this directory already exists as a subdir of tmpdir, which is either configured or defaults to /tmp.

    tty: Request a pseudo tty for the SSH session. This option is generally only used in conjunction with the run_as option when the sudoers policy requires a tty. Default is false.

    user: Login user. Default is root.
modulepath: "modules:site-modules:site"
inventoryfile: "~/.puppetlabs/bolt/inventory.yaml"
concurrency: 10
format: human
ssh:
  host-key-check: false
  port: 22
  private-key: ~/.ssh/id_rsa
  user: foo
  run-as-command: ['sudo', '-k', '-n']



## Advanced

After every execution, Bolt writes information about the result of that run to a .rerun.json file inside the Bolt project directory.



version: 2
targets:
  - _plugin: puppetdb
    query: "inventory[certname] { facts.osfamily = 'RedHat' }"
    target_mapping:
      name: certname
      config:
        ssh:
          hostname: facts.networking.interfaces.en0.ipaddress
          
## Senior

