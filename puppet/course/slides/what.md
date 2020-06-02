## Essential concepts 1/2
<img src="gfx/junior.png" class="skill">

Puppet uses a custom **declarative language** which describes
the **resources** to manage on a system:
files, packages, services, users...

Puppet language is written in files called **manifests**, which 
have a **.pp** extension.

When we run Puppet, a companion tool, **facter** is executed.
Facter collects **facts**, information about different
elements of the local system.

Based on facts and the manifests a **catalog** of the resources to manage
is generated and enforced locally.

Puppet can run in two main modes:
- **apply** where it uses a catalog based on local manifests 
- **agent** where it receives a catalog from the Puppet server

When the agent interacts with the server it uses an encrypted connection
based on a x509 **certificate** generated on the client and
signed by the Puppet server's internal Certification Authority.

---

## Essential concepts 2/2
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

