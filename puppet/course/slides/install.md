## Puppet installation
<img src="gfx/junior.png" class="skill">

Puppet packages are available natively on most Linux distros, just run:

    yum install puppet # On RedHat derivatives
    apt install puppet # On Debian derivatives

We recommend however to use the upstream repositories from  the [Puppet site](https://puppet.com/docs/puppet/latest/install_puppet.html) with updated versions available.

The packages available on official Puppet repos are:

- **puppet-release**, the package to install and configure the Puppet repository (for Debian and Redat derivatives)
- **puppet-agent**, the base package with Puppet, Facter, Hiera, the PXP agent and all the necessariy prerequisites (recent Ruby version included)
- **puppetserver**, the Puppet server software, written in Clojure
- **puppetdb**, the PuppetDB component, used to store all Puppet related data
- **pdk**, the Puppet Development Kit, useful when writing modules
- **puppet-bolt**, Bolt, the agentless remote execution tool to run commands, tasks, and plans on remote nodes

---

## Using Puppet from the command line
<img src="gfx/junior.png" class="skill">

Puppet agent can be installed and used on a node (maybe our own workstation) even without having a server.

We can use it to:

- Get the system facts (**puppet facts**)
- Apply directly local Puppet code without the need of a server (**puppet apply**)
- Investigate how Puppet represents system resources (**puppet resource**)
- Install locall puppet modules from the Modules Forge (**puppet module**)

<asciinema-player cols="200" src="casts/puppet_facts.cast" autoplay="4"></asciinema-player>


---

## How to run Puppet
<img src="gfx/junior.png" class="skill">

A Puppet run on a client can be triggered in different ways:

- As a **service** (default configuration), which polls the server every **30 minutes** (default)
- Via a **cron job** (typically with random delays to avoid requests congestion)
- **Manually** from the client's command line
- In a **centralized way** via MCollective, Bolt or any other remote execution tool
- From the **Puppet Enterprise (PE) console**, via the Orchestration feature


---

### Anatomy of a Puppet Run 1/2
<img src="gfx/intermediate.png" class="skill">

**Run Puppet** agent on the client (usually as root):

    Client shell # puppet agent -t

First, the client retrieves **plugins** (facts, types and providers...) present in modules on the server:

    Client output # Info: Retrieving plugin

Then the client runs **facter** to collect facts and send them to the server:

    Client output # Info: Loading facts in /opt/puppetlabs/puppet/lib/facter/... [...]

The server **compiles the catalog** for the client:

    Server's logs # Compiled catalog for <client> in environment production in 8.22 seconds

If there are not syntax errors in the processed Puppet code, the server sends the catalog to the client:

    Client output # Info: Caching catalog for <client>

If there are syntax errors, missing references or other hard errors on our code, here we get an error instead.


---

### Anatomy of a Puppet Run 2/2
<img src="gfx/intermediate.png" class="skill">

Once the client has received the catalog, it resolves the dependencies order of the resources (it fails here if there are depenencies loops) and it starts to **apply** it:

    Client output # Info: Applying configuration version '1355353107'

All **changes** to the system are shown starting from here. New resources might be added, removed or modified, if there are errors in applying specific resources the whole relevant line is in red.

For example the following line shows that the content of the file **/etc/motd**, declared in the class **psick::motd**, has been changed:

    Client output # Notice: /Stage[main]/Psick::Motd/File[/etc/motd]/content: content changed '{md5}092f5fa021be46e167c4d653f25cce64' to '{md5}7f5dcc5a6be4613d14997ee23f6e7dd1'

a diff of the changed lines is also displayed if puppet agent has been run with the *-t* argument.

At the end of the Puppet run the client sends to the server a report of what has been changed

    Client output # Finished catalog run in 13.78 seconds

The server eventually sends the report to a Report handler (which can, for example senda n email or add a message in a chat) and PuppetDB.

---

### Ways to setup Puppet
<img src="gfx/junior.png" class="skill">

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

