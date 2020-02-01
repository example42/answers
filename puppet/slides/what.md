## Essential concepts

When you run the Puppet command, **facter** is executed.
Facter collects **facts**, information about different
elements of the local system.

In a standard setup, on a managed node runs **Puppet agent**,
which sends the node's certificate and facts to the server.

Using these and its local Puppet code and data,  the Puppet server
generates and returns a **catalog** of resources to apply on the node.

The catalog is based on code in the Puppet language,
saved in files called **manifests**, with .pp extension.

In the Puppet code we declare **resources** that manage
elements of the system (files, packages, services ...).

---

Resources are grouped in **classes** which
may have parameters that affect their behaviour.

The values of the **class parameters** can be configured via Hiera.

**Hiera** is a pluggable key-pair lookup tool,
based on hierarchies we can customise.

Classes, new resources, custom facts and any relevant
configuration file for a specific purpose are organised in **modules**.

The **Puppet Forge** is a public collection of reusable modules.

All our Puppet code (local and public modules) and Hiera data
can be stored or referred to in the **control-repo**.

---


## Puppet infrastructure

A typical Puppet infrastructure is composed by:

### Puppet clients

Our managed nodes, where Puppet agent client is installed:

- Connect to server via https to port 8140 TCP
- Are identified by a ssl certificate, with the node name
- Runs Puppet agent as root (or Administrator)
- Collect local facts and send them to server
- Apply the catalog received from the server

---

### Puppet servers

One or more Puppet servers, our friendly Puppet Masters.

- Our Puppet code and data is here
- Compiles a catalog of resources to apply on eacht client
- Use client certname and facts, and our code and data
  to define what resources are expected on each node
- Act as Certification Authority for clients and servers certificates
- Can store clients' catalogs, facts and reports to PuppetDB



---

### Puppet used locally without servers

You can still use Puppet on a single node, even your workstation,
without servers, in master less mode, to:

- Get the system facts
- Apply directly local Puppet code
- Investigate how Puppet represents system resources.

---

## How to run Puppet

A Puppet run on a client can be triggered in different ways:

- As a service (default configuration), which polls the server every 30 minutes (default)
- Via a cron job (typically with random delays to avoid requests congestion)
- Manually from the client node
- In a centralized way via MCollective or Bolt
- From the Puppet Enterprise (PE), via Orchestration features, based on MCollective, in older versions, or Bolt, starting from PE version 2016.x


---

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

---

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

