## Essential concepts
<img src="gfx/junior.png" class="skill">

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

## Essential concepts
<img src="gfx/junior.png" class="skill">

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

