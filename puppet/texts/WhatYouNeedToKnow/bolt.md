# What you need to know about Bolt

## Noop

Hashtag #puppetbolt

## Beginner 

Search: puppet bolt

Quote:

"Bolt is Open Source Agentless Workflow Orchestration tool"
Lucy Wyman

## Junior

### Basics

Bolt is an Open Source Agent-less Workflow Orchestration tool.

It's similar to Ansible as it doesn't need an agent running on the target node. 

It can execute different activities on one or more target nodes,
reaching them via ssh, winrm, local, docker, pcp transport protocols.

The most common activities which Bolt can perform are:

- Run a command existing on remote systems
- Upload and run on remote systems a local script
- Upload a file to remote targets
- Apply local Puppet code on a remote target (where Puppet is not installed)
- Run a Puppet Task (can be a script in any language, distributed in Puppet modules)
- Run a Puppet Plan (a more complex collection of tasks and scripts which be run on different targets) 

Credentials and connection settings to targets can be passed as arguments or stored in an inventory file.

In the inventory.yaml file it's possible to define different groups of targets and relevant options.


## Intermediate

## Advanced

## Senior

