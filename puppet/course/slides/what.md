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


---

## Puppet infrastructure 1/2
<img src="gfx/junior.png" class="skill">

A typical Puppet infrastructure is composed by:

### Puppet clients

The **nodes we manage**, where we install the Puppet agent component.

These nodes, typically Linux or Windows servers:

- Connect to Puppet server via **https** to **port 8140** TCP
- Are identified by an **ssl certificate**, with the node name
- Run Puppet agent as **root** (or **LocalSystem** on Windows)
- Run **facter**, which collects local **facts** we can use in our Puppet codedir
- **Apply the catalog** received from the server
- Each **resource** of the catalog is something to manage on the client

---

## Puppet infrastructure 2/2

The second component are the:
### Puppet servers
<img src="gfx/junior.png" class="skill">

One or more Puppet servers, our friendly Puppet Masters, which:

- Host our **Puppet code and data**
- **Compile a catalog** for each client
- Use clients' **certificate name** and **facts**, and our **code** and **data**
- Act as a **Certification Authority** (CA) for clients and servers certificates
- Can send to **PuppetDB** clients' catalogs, facts, and reports
- PuppetDB contains information about the Puppet managed infrastructure

