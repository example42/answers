# Puppet changes impact overview

We share here information useful to understand what can be the impact of a change in Puppet code or Hiera data in a standard Puppet infrastructure.

Take this as a general reference to use and adapt to each use case, without forgetting that the best way to assess the impact of a change is to test it in as many different conditions as needed.

We identify the following risk levels, from lower to higher:

- **[safe]** Changes done here are totally safe in terms of impact on running servers
- **[limited]** Changes here impact a limited number of servers or not critical elements
- **[warning]** Changes may impact several servers and should be considered with care
- **[danger]** Changes may have a very large impact. Be sure to be aware of what's the impact

Let's have a quick overview of the risk level related to different kind of files.

Needless to say that they refer to actual changes in Puppet code and data, if we are just adding a comment we can be confident that the change won't have any effect on live systems.

## Control-repo

Most of our Puppet code and data is outside of the control-repo and placed in dedicated git repos, still a few key files are present here (paths are relative to the control-repo's main directory): 

- **[safe]** `README.md`, `docs/`, `LICENSE` contain documentation and general information. Changes done here won't have any impact on our servers
- **[danger]** `data/` directory contains Hiera data by default, as configured in `hiera.yaml`. In some cases Hiera data files can be under `hieradata/` or in a module deployed via `Puppetfile` or not even used, where an Hiera backend not based on files is used. In any case any change to Hiera data can affect more or less widely (according to the Hiera layer where the change occurs and the nodes matching that layer) the infrastructure.
- **[danger]** `hiera.yaml` is the Hiera configuration file for the environment, changes here (for in the hierarchy or the used backends) may affect several systems in more or less unpredictable ways. Edit it only if we know what we are doing. Be aware of and sure to understand the contents of this file, as they affect the whole logic of the used hierarchies and the relevant area of impact.
- **[danger]** `manifests/` contains the first manifests parsed by Puppet server when compiling a new catalog. Files here impact all the nodes. Usually just one file is present here, `manifests/site.pp`, which can be considered the single place from where everything begins. Handle with care.
- **[warning]** `Puppetfile` contains the list of the modules to add to the control-repo. If we add a new module we won't have any effect on nodes (except for the automatic pluginsync of the lib directory of the module and the local execution of its eventual facts) until we actually start to use its classes and defines. If we remove a module we'll break Puppet runs in all the nodes that eventually use it. When we add or remove modules, we may see on our nodes files changing at the first Puppet run: these are due the contents of module's plugins being synced to the clients (pluginsync feature) they are normal and won't affect our servers operations.
- **[warning]** `site/` or `site-modules/` (according to the modulepath configured in `environment.conf`)  contain modules that are directly in the control repo (and not in separated git repos). 

## Hiera data

Hiera data, wherever it's place, contains the files, typically in yaml format, with all the data which configures out infrastructure. The impact of changes in these files depends directly on the hierarchies configured in `hiera.yaml`.

Given a sample content as follows:

  hierarchy:
    - name: "Hierarchy"
      paths:
        - "nodes/%{trusted.certname}.yaml"
        - "role/%{::role}-%{::env}.yaml"
        - "role/%{::role}.yaml"
        - "zone/%{::zone}.yaml"
        - "common.yaml"

we can expecte these kind of impact:

- **[danger]** `/common.yaml` contain Hiera data which is used for all the nodes (unless overwritten in higher levels), so any change here may impact several servers. Be aware.
- **[danger]** `zone/*.yaml` could contain data for entire network zones, datacenters or mayor sets of nodes. Changes in any of these files are likely to impact a wide number of nodes.
- **[warning]** `role/*.yaml` are files which configure specific roles, or specific environments for a role. Changes here affect all the nodes of the given role.
- **[limited]** `nodes/*.yaml` file contain Hiera data for single specific nodes. Here we can place nodes specific settings, which are easy to test (directly on the involved node) and have a limited impact (only the node having the name of the file we change.

## Additional notes:

The impact of the changes in a specific module, depends on factors like:

  - the potential number of nodes they impact (which basically depends on how many nodes use a specific class).
    This information can be searched in different places:
      - on the PE Console in the node graph detail page (for each single node) (on PE versions older than 2019.4)
      - on the PE Console in the Run Puppet or Task pages, selecting under Inventory and then a PQL Query on nodes with a specific class
  - the resources managed and the effect of their change. For example the worst thing that can happen when changing the contents  of the file /etc/motd is a different text when logging to a node, without any real effect on the node's service. On the other hand changing a configuration file like /etc/ssh/sshd_config, which will eventually trigger a restart of the ssh service, may, in case of wrong configurations, prevent ssh access to nodes in case of restart failures.

In case some refactoring is needed in a class' code, consider the following:

  - If the default value for a class parameter is changed, this will apply to all the nodes which use that class and don't have a relevant value set via Hiera. In general, rather than changing defaults (unless they are really wanted and expected potentially everywhere) for class parameters is better to override them via Hiera (with the option to better test and control their propagation by working on different hiera files)
  - When an hard coded setting has to be changed, it's better to expose that setting as a class parameter (having as default the previously hardcoded value) and change via Hiera the relevant value where needs. For example, a class like this, where we want to customize the value of the package version:

        class openssh {
          package { openssh:
            ensure => '1.1.0',
          }
        }

  can be safely refactored to something like:

        class openssh ( 
          $package_ensure = '1.1.0',
        ) {
          package { openssh:
            ensure => $package_ensure,
          }
        }

  Such a change would not affect in any way existing nodes, but allows, where needed, to configure a custom version via the Hiera key: openssh::package_version.

  - When changing Hiera data files, we can set any key value, if that key is used or not it depends if in a given node it's included the class which uses that key. By default hiera keys are looked up with "first found first returned" method. That is, if a key is looked for on, Hiera first looks for it in the first layer, that is the single host specific file. If the key is found in that file, the relevant value is returned, if not, the next line in the hierarchy is looked for, and so on until the default "hiera/common.yaml" file.
  In some cases, anyway, an Hiera key might be looked with using the "merge" method, which works with arrays or hashes, and instead of having Hiera stopping at the first occurrence of a key, while traversing its hierarchy, it keeps on gathering the values for that key across the whole hierarchy level, and then merges the relevant values (of course the one found first have precedence, in case of "conflict").

Don't be to worried about the [warning]  and [danger] tags, they are used as a reference but should not block us from doing changes, if they are needed.

The keys to work with Puppet and be confident what with it are: knowledge and testing.

Knowing what we are doing and its potential impact is fundamental, feel free to ask to the Puppet experts in the company for advice, in case of doubt.

Testing the changes before pushing them is important as well. Tests should be done where it matters, possibly on all the different conditions / nodes where the things we changes may have an impact. We are working on automating some of these tests, in the mean time, follow the procedure we've always followed for testing Puppet code, adapted to the new infrastructure.