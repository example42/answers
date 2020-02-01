# Commands That Matter: Bolt

## Beginner 



## Junior

### Bolt Installation

Installation on Linux

Install the release package to enable the Puppet Tools repository, then Bolt can be installed as a normal package.

On Debian and Ubuntu commands are as follows. Change <dist_codename> with the Distribution codename
(quick reminders: Debian8=jessie , Debian9=stretch , Debian10=buster, Ubuntu16.04=xenial , Ubunut18.04=bionic)

    wget https://apt.puppet.com/puppet-tools-release-<dist_codename>.deb
    sudo dpkg -i puppet-tools-release-<dist_codename>.deb
    sudo apt-get update 
    sudo apt-get install puppet-bolt

On RedHat and derivatives (Centos, Scientific Linux...) (change <v> with RHEL major version: 6, 7, 8)

    sudo rpm -Uvh https://yum.puppet.com/puppet-tools-release-el-6.noarch.rpm
    sudo yum install puppet-bolt	

On Fedora (change <v> with Fedora version (supported as on 20200111: 28, 29)):

    sudo rpm -Uvh https://yum.puppet.com/puppet-tools-release-fedora-<v>.noarch.rpm
    sudo dnf install puppet-bolt

On Suse Enterprise server 12

    sudo rpm -Uvh https://yum.puppet.com/puppet-tools-release-sles-12.noarch.rpm
    sudo zypper install puppet-bolt

Installation on MacOS

- Download DMG image with installer from [Puppet site](https://puppet.com/docs/bolt/latest/bolt_installing.html#install-bolt-on-macos)
- If you have Homebrew:

        brew cask install puppetlabs/puppet/puppet-bolt

Installation on Windows

- Download msi installer from [Puppet site](https://downloads.puppet.com/windows/puppet6/puppet-bolt-x64-latest.msi)
- If you have Chocolatey: choco install puppet-bolt

Download and run from a Docker image

    docker pull puppet/puppet-bolt
    docker run puppet/puppet-bolt <subcommand> [action] [options]

# Bolt is Ruby code, can be installed as gem, as least recommended way.

gem install bolt



### Bolt subcommands

bolt --help

bolt <SUBCOMMAND> [ACTION]
bolt <SUBCOMMAND> <ACTION> <COMMAND> --targets winrm://<WINDOWS.TARGET> --user <USERNAME> --password <PASSWORD>

Run a command on the defined targets. Command must be already present on the target host.
bolt command run <COMMAND> --targets <TARGET NAME>,<TARGET NAME>,<TARGET NAME>

bolt command run <COMMAND> --targets winrm://<WINDOWS.TARGET> --user <USERNAME> --password <PASSWORD>
bolt command run <COMMAND> -n <NODE1,NODE2,NODE3,NODE4>
bolt script run <PATH/TO/SCRIPT> --nodes <NODE1,NODE2,NODE3,NODE4>
bolt task run package action=status name=vim --nodes <NODE1,NODE2,NODE3,NODE4>
bolt file upload <SOURCE> <DESTINATION> --nodes <NODE1>
bolt file upload <SOURCE> <DESTINATION> --targets <TARGET NAME>,<TARGET NAME>

bolt file upload my_file.txt /tmp/remote_file.txt --targets web5.mydomain.edu,web6.mydomain.edu

bolt command run <COMMAND> --targets @targets.txt
bolt command run --targets '@targets.txt' # On Windows PowerShell quoting is needed
cat targets.txt | bolt command run --targets -
bolt command run <COMMAND> --targets ssh://user:password@[fe80::34eb:ff1:b584:d7c0]:22,
ssh://root:password@hostname, pcp://host01, winrm://Administrator:password@hostname

bolt command run <COMMAND> --targets elastic_search,web_app


version: 2
groups:
  - name: elastic_search
    targets:
      - elasticsearch1.subdomain.mydomain.edu
      - elasticsearch2.subdomain.mydomain.edu
      - elasticsearch3.subdomain.mydomain.edu
  - name: web_app
    targets:
      - web5.mydomain.edu
      - web6.mydomain.edu
      - web7.mydomain.edu

## Intermediate

bolt command run 'hostname' --targets <LINUX_TARGETS> --user <USER> --password <PASSWORD>
bolt command run 'hostname' --targets <LINUX_TARGETS> --user <USER> --password-prompt
bolt command run 'hostname' --targets <LINUX_TARGETS> --user <USER> --private_key <PATH_TO_PRIVATE_KEY>

Available transports are: ssh, winrm, local, docker, pcp

### Running Bolt via Docker image



### [Bolt plugins](https://puppet.com/docs/bolt/latest/using_plugins.html)


## Advanced


## Senior
