# What you need to know about Puppet

## Beginner 

## Junior

### Puppet infrastructure

In a typical setup we have:

Clients, our managed nodes, where Puppet agent is installed:
  - Connect to server via https to port 8140 TCP
  - Have a local certificate, with the node name
  - Puppet agent runs as root (or Administrator)
  - Collect local facts and send them to server
  - Apply the catalog received from the server

Server, our friendly Puppet Master
  - Contains our Puppet code and data
  - It generates a catalog or resources to apply on the client
  - Catalog is based on the code and data on server and the client certificate and facts
  - Acts as Certification Authority for Puppet certificates
  - Can store clients' catalogs, facts and reports to PuppetDB

Puppet can also be used on a node without servers, to query or change local resources.

### How to run Puppet

A Puppet run on a client can be triggered in different ways:

- As a service (default configuration), which polls the server every 30 minutes (default)
- Via a cron job (typically with random delays to avoid requests congestion)
- Manually from the client node
- In a centralized way via MCollective or Bolt
- From the Puppet Enterprise (PE), via Orchestration features, based on MCollective, in older versions, or Bolt, starting from PE version 2016.x




### Essential concepts

The catalog is based on Puppet code, which is written in manifests (files with .pp extension)

In the Puppet code we declare resources that affect elements of the system (files, packages, services ...)

Resources are grouped in classes which may expose parameters that affect their behavior.

The values of the class parameters can be set, with different values for different nodes, via Hiera

Hiera is a key-pair lookup tool, based on hierarchies we can define

Classes and the configuration files that are shipped to nodes are organized in modules.

The Puppet Forge is a public collection of reusable modules.

All our Puppet code (local and public modules) and Hiera data can be stored or referred to in the control-repo.


### Anatomy of a Puppet Run

Execute Puppet on the client

    Client shell # puppet agent -t

If pluginsync = true (default from Puppet 3.0) the client retrieves all extra plugins (facts, types and providers) present in modules on the server's $modulepath

    Client output # Info: Retrieving plugin

The client runs facter and send its facts to the server

    Client output # Info: Loading facts in /var/lib/puppet/lib/facter/... [...]

The server looks for the client's hostname (or certname, if different from the hostname) and looks into its nodes list

The server compiles the catalog for the client using also client's facts.

    Server's logs # Compiled catalog for <client> in environment production in 8.22 seconds

If there are not syntax errors in the processed Puppet code, the server sends the catalog to the client, in PSON format.

    Client output # Info: Caching catalog for <client>

The client receives the catalog and starts to apply it locally If there are dependency loops the catalog can't be applied and the whole tun fails.

    Client output # Info: Applying configuration version '1355353107'

All changes to the system are shown here. If there are errors (in red or pink, according to Puppet versions) they are relevant to specific resources but do not block the application of the other resources (unless they depend on the failed ones).

At the end ot the Puppet run the client sends to the server a report of what has been changed

    Client output # Finished catalog run in 13.78 seconds

The server eventually sends the report to a Report Collector.

### Ways to setup Puppet

Puppet can be used in multiple ways:

- Master / Client - puppet agent

    It's the typical Puppet setup

    We have clients, our managed nodes, where Puppet agent is installed.

    We have one or more Masters where Puppet server runs as a service

    Client/Server communication is via https (port 8140)

    Clients certificates have to be accepted (signed) on the Master

    Command used on the client: puppet agent (generally as root)

    Command used on the server:
        Legacy Ruy based master in old Puppet versions: puppet master (generally as puppet)
        "New" separated PuppetServer Clojure application puppetserver

- Masterless - puppet apply

    Master less mode doesn't use a client-server infrastructure.

    Our Puppet code (written in manifests) is applied directly on the target system.

    No need of a Puppet Master

    We have to distribute our modules and data to the managed nodes.

    Command used: puppet apply (generally as root)

- Agentless mode (Puppet without Puppet (agent))

    It's possible to apply Puppet manifests on a node where Puppet is not installed using bolt.

    The command bolt apply <manifest> will act on a remote node, accessible via ssh or winrm, and apply there the local manifest containing Puppet code.

    No Puppet package is needed on the remote node (bolt will install it under the hoods) and no service is needed.

## Intermediate

## Advanced

## Senior

