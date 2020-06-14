## Classification
<img src="gfx/junior.png" class="skill">

Classification consists in assigning Puppet classes to the managed nodes.

Each class contains the resources that are included in the nodes' catalog.

Class parameters can be set via Hiera or explicitly declared, like normal resources.

We can have classes that can just include and group other classes.

There're different ways to manage classification in Puppet.

They can be interoperable and cohexist, even if it's better to stay with a single approach.

---

## Classification via the node  statement
<img src="gfx/junior.png" class="skill">

It's the original approach, not to much used  in these days, if not for the defaul node.

It uses the node statement which contains a list of Puppet code to apply to a given node.

Nodes inheritance was a method used in the past to group different nodes, now is more available.

For each node name we can declare resources, set node variables, or, more commonly just include classes:

    node 'web01.example.com' {
      include ::general
      include ::apache
    }

RegExps can be used as well:

    node /^web/ {
      include ::general
      include ::apache
    }

---

## Classification via an External Node Classifier (ENC)
<img src="gfx/intermediate.png" class="skill">

Via an [ENC](https://puppet.com/docs/puppet/latest/nodes_external.html) we can define the classes (and parameters) that each node should have in a totally separated tool.

To configure a Puppet server to use an ENC add lines like these to `puppet.conf`:

    [master]
      node_terminus = exec
      external_nodes = /usr/local/bin/enc

Puppet runs the command specified via `external_nodes` and it passes as argument the client's certname.

The executed command can do anything with any language and has to return a YAML output as follows:

    ---
    environment: production
    classes:
      - general:
      - apache:
    parameters:
      role: 'web'

Here are defined the classes to include for that node, the global parameters (top scope variables) and the Puppet environment to use for the given node.

[Puppet Enterprise](https://puppet.com/products/puppet-enterprise), [The Foreman](https://www.theforeman.org),  [Puppet Dashboard](https://github.com/sodabrew/puppet-dashboard) can all act as ENC.

Example of an [essential ENC](https://github.com/example42/psick/blob/production/bin/enc_cat.sh) which uses files in [this directory](https://github.com/example42/psick/tree/production/bin/enc_cat).

---

## Classification via hiera
<img src="gfx/intermediate.png" class="skill">

We can specify the list of classes to include on a node via Hiera.

The legacy function `hiera_include` can look for a Hiera key (here 'classes' ) and get an array of classes to include:

    hiera_include('classes').

The current alternative is the lookup function, used as follows:

    lookup('classes',Array,'unique',[]).include

Which is a fancy and condensed way of writing:

    $classes = lookup('classes',Array,'unique',[])
    $classes.each | $class | {
      include $class
    }

With the above functions, we define in our hierarchy under the key named `classes`, what to include for each node, the result is merged across the hierarchies:

    classes:
      - general
      - apache

---

## Nodeless Classification
<img src="gfx/intermediate.png" class="skill">

We can include classes directly in the main manifest `manifests/site.pp`:

    include general

if we have a $role variable some way, and we use the roles and profiles pattern, we can classify the relevant role class with:

    include "role::${::role}"

Or we can include different common classes according the OS we are dealing with:;

    include downcase("base::${::kernel}")

Generally, we can manage any logic of what classes to include in nodes according to any variable we can have at disposal.

For complex cases, however is usually better to have these defining variables in Hiera and use it for classification.

