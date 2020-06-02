# Puppet Enterprise


## Puppet Enterprise (PE) Features

The most interesting and useful features of Puppet Enterprise (PE) are:

  - Puppet console, a web frontend featuring:
    - A Dashboard for a quick overview of the infrastructure status
    - RBAC - Role-based access control, which allows fine-tuning on what each user can do
    - Ldap/AD authentication, with the possibility to handle users authorization schemes for each user or group
    - Detailed and general Reports view for full awareness on what's happening on the infrastructure
    - Events view to quickly identify what and where changes occur
    - Central interface to trigger puppet runs on the infrastructure nodes
  - Integration with Bolt, with the ability to run and audit:
    - Puppet agent runs
    - Puppet tasks
    - Puppet plans
    - Scheduled tasks/plans
  - HA deployment with Master and Replica server
    - Possibility to add compile masters to scale horizontally
  - Code Manager to manage Puppet code deployments on all the PuppetServers
  - Rich REST API

The Puppet version used (6.x at the time of writing) has:

  - "Future Parser" (since version 4) with features like:
    - Lamdas and iterations
    - Data type enforcement
    - Cleaner and more consistent behavior
  - Hiera 5 (since version 4.10) with three layers of hiera hierarchy
    - Global (configured in `/etc/puppetlabs/puppet/hiera.yaml`)
    - Environment (configured in `/etc/puppetlabs/code/environments/$environment/hiera.yaml`)
    - Module (configured in `$MODULE_PATH/$module/hiera.yaml`)

## Puppet Enterprise Components

Puppet Enterprise is composed by several different services which may live on the same server (All In One installation, **AIO** for short) or in different ones.

It's possible to have an HA setup with Master and Replica servers, in this scenario a failure on the Primary Master Replica has no visible effect for any Puppet related activity and a failure on the Primary Master may have some impacts according to the failing service (as described more in details below).

An AIO PE server runs the following services:

  - **pe-puppetserver**. The core Puppet Server service responsible for communication with clients and compilation of their catalogs
  - **pe-puppetdb**. The PuppetDB service, responsible for handling all the data produced by Puppet
  - **pe-console-services**. The PE web interface we can access with a browser
  - **pe-nginx**. An NGINX reverse proxy for the PE console
  - **pe-postgresql**. A PostgreSQL instance where is stored the data generated, used or handled by PuppetDB and the PE Console
  - **pe-orchestration-services**. Responsible for handling Puppet Jobs (such as Tasks, Plans and remote Puppet runs)
  - **pe-ace-server**. Agentless Catalog Executor Services which provides agentless executions services for tasks and catalogs
  - **pe-bolt-server**. Bolt server which exposes APIs for the execution of tasks

All the Puppet clients (and also the Masters, which are clients of themselves) have the following services:

  - **puppet**. The Puppet agent service, it requests the catalog from the server and applies it locally. Runs as root.
  - **pxp-agent**. It's used to allow the remote executing of Puppet runs, tasks and plans from the PE server.

## Puppet Enterprise All In One Master of Masters installation

Here follows the complete procedure to setup a Puppet Enterprise Master of Masters (**MOM**) server.

### Download of PE installation tarball

Puppet Enterprise tarball, for the Linux distro used, must be copied locally on the MOM server.

This can be copied via scp or directly downloaded from upstream Puppet site. In this case, proceed as follows.

If Internet can be reached only only via a proxy, set with with something like:

    export http_proxy=http://proxy.example.com:8080
    export https_proxy=http://proxy.example.com:8080
    export no_proxy=*.example.com,localhost

Then create a directory where to download the installation tarball:

    mkdir -p /var/tmp/pe-puppet
    cd /var/tmp/pe-puppet

Finally download the latest installation tarball in this case for RHEL 7 on x86_64 architecture:

    wget --no-check-certificate "https://pm.puppet.com/cgi-bin/download.cgi?dist=el&rel=7&arch=x86_64&ver=latest" -O puppet-enterprise-latest.tar.gz

In case a proxy was set, remember to unset it before proceeding with the installation, in order to avoid problems and errors:

    unset http_proxy
    unset https_proxy
    unset no_proxy

If we want to configure since the beginning the integration of PE with GitLab configuring Code Manager, the following steps are needed:

#### Creation / copy of ssh key pairs for GitLab access via ssh from Code Manager

Puppet Enterprise has a component, called Code Manager, which automates the deployment of Puppet code on the Master of Masters (MoM) server from a git repository. In order to do that via ssh, a key pair, with read access to the remote git repo on GitLab, has to be created (or copied) and placed in a configurable place.

To create a new ssh key pair, let's first create the directory where to place it:

    mkdir -p /etc/puppetlabs/puppetserver/ssh

Then we use ssh-keygen to create the ssh key pair (if not done before)

    ssh-keygen -b 4096 -t rsa -f /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa

In case the private and public keys were previously created, retrieve them and copy them in the same locations:

    # Private-key path:
    # /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa

    # Public key path:
    # /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub

**IMPORTANT**: The public key has then to be added (or be present) to a user (here called @puppet) on GitLab which has at least read access to the Puppet [control repo git repository](https://git.example.com/puppet/control-repo.git), as described in the following lines. 


#### GitLab configurations

The following configurations should be in place in GitLab in order to setup PE integration with code-manager:

- A dedicated user is created for access from PE (here called @puppet)
- This user has read access (Reporter) to the control-repo git repository used
- This user has read access (Reporter) to all the modules on GitLab which are listed in the control-repo's `Puppetfile`
- The previously mentioned public key (`/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub`) has been added to the @puppet user's ssh keys:  **Impersonate User -> Settings -> SSH Keys -> Add an SSH Key**

#### External modules

All the external modules added to the control-repo's `Puppetfile` have to be added to the GitLab server if PuppetMasters have no direct Internet access.

In this case, all the external modules used have to be downloaded from the Forge or GitHub and added on the internal GitLab server (for example under a group called  *external_modules*) as independent git repositories. So, for example the *puppetlabs-cd4pe_jobs module* is added to the Puppetfile as follows:

    mod 'puppetlabs/pipelines',
      :git => "git@git.example.com/puppet/external_modules/puppetlabs-cd4pe_jobs.git",
      :tag => "1.0.1"

Note that it's always recommended to specify the  **:tag** or the **:commit** id to use for each module, in order to be sure that it's always deployed a known version.

In order to retrieve a module from the Internet and push it to GitLab the following commands have to be done on a machine which is able to reach both the Internet, via a proxy, and the GitLab server:

First a new project has to be created on GitLab (here we use, as example, the puppetlabs-cd4pe_jobs under the external_modules group), then the following commands have to be run in order to clone the relevant module from GitHub and push it to the internal GitLab:

If working behind a proxy, set the proxy:

    export https_proxy=http://proxy.example.com:8080
    export no_proxy=*.example.com,localhost

Clone module from GitHub, change git origins and then push it to the internal Git server:

    git clone https://github.com/puppetlabs/puppetlabs-cd4pe_jobs
    cd puppet-cd4pe_jobs
    git remote rename origin upstream
    git remote add origin git@git.example.com/puppet/external_modules/puppetlabs-cd4pe_jobs.git
    git push origin master
    git push origin master --tags


### Puppet Enterprise MoM installation

With the ssh keys in place able to access the Puppet control repo on GitLab we can proceed with the Puppet Enterprise installation (remember, we could skip the previous steps and configure Code Manager in a later stage). This process can be manual or based on a `pe.conf` response file with some predefined configurations.

We can create this file in the directory where we placed the PE installer tarball:

    vi /var/tmp/pe-puppet/pe.conf

Content should be as follows (note: the console_admin_password has been blanked here. Place as value the desired initial password for the **admin** user on PE Console):

    {
      "console_admin_password": "XXXXXX"
      "puppet_enterprise::puppet_master_host": "%{::trusted.certname}"
      "pe_install::puppet_master_dnsaltnames": ["puppet-master","puppet-console"]
      "puppet_enterprise::profile::master::code_manager_auto_configure": true
      "puppet_enterprise::profile::master::r10k_remote": "git@git.example.com:puppet/control-repo.git"
      "puppet_enterprise::profile::master::r10k_private_key": "/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa"
    }

Note that:

  - **pe_install::puppet_master_dnsaltnames** must match any possible name used by clients or other applications (like CD4PE) to reach the MoM server
  - **puppet_enterprise::profile::master::r10k_private_key** must be the local path of the previously created or copied ssh *private* key
  - **puppet_enterprise::profile::master::r10k_remote** must be the ssh url where Puppet control-repo is hosted. Must be readable by a GitLab user having the public key matching the above private key
  - On this example the repo used is  **"git@git.example.com:puppet/control-repo-lab.git"** and the dns_alt_names are: **["puppet-master","puppet-console"]**

Code Manager can be configured also later, after PE installation, either directly from the console, in the classification interface, of via Hiera, with keys as follows:

    puppet_enterprise::profile::master::r10k_remote: git@git.example.com:puppet/control-repo.git'
    puppet_enterprise::profile::master::r10k_private_key: '/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa'
    puppet_enterprise::profile::master::code_manager_auto_configure: true

We can now finally unpack and run the installer (as root on MoM node, note that the name of the tar and the exploded dir change with PE version):

    cd /var/tmp/pe-puppet
    tar xvf puppet-enterprise-latest.tar.gz
    cd puppet-enterprise-20XX.X.0-el-7-x86_64 # Version number changes over time
    ./puppet-enterprise-installer -c /var/tmp/pe-puppet/pe.conf

Installation takes some minutes, no Internet access is required, if proxies are set as environment variables they should be unset, several lines will be shown on the screen, under the hoods Puppet agent is installed from packages in the extracted dir, then a puppet run is done based on local Puppet enterprise modules (which are then used on the same Puppet server to manage the Puppet infrastructure) and this installs and configures, using other local packages, all the needed components.

At the end of the installation a link is provided to access the console.

Login is **admin**, and password is what has been set with the setting **console_admin_password** in `pe.conf`.

#### First code deployment

After the MoM has been installed we can deploy our Puppet code on it, using the r10k configurations previously added to the *`pe.conf`.

The following commands can be run as root on the MoM server just installed.

First it's necessary to create a local token to interact with the Console. The token lifetime by default is only 5 minutes, in the following example the lifetime has been set to 420 days.

    /opt/puppetlabs/bin/puppet-access login --lifetime 420d

The output is as follows, in the prompt, can be given the credentials of an existing user on the console, the generated token will be able to do what this user can do.

In this example we use the admin user which is already present, if we want to use credentials of a user with less privileges, we have first to create it, as described later in the "Create a deploy user on PE console".

    Enter your Puppet Enterprise credentials.
    Username: admin
    Password: ****

    Access token saved to: /root/.puppetlabs/token

Then it's necessary to complete at least one code deployment, this is a prerequisite for the replica setup and to start to use our own control-repo code:

    /opt/puppetlabs/bin/puppet-code deploy --all --wait


#### Configuration of a webhook on GitLab to automate code deployments 

The above `puppet-code deploy` executed locally on the Puppet Master performs a deployment of the configured control-repo from the GitLab server. In most of the cases we want to automate this operation, usually whenever there's a change committed (or merged) on the control-repo's branches mapped to Puppet environments.

If CD4PE is used, deployments are handled by CD4PE itself and no further configurations are necessary.

Otherwise it makes sense to add a webhook on GitLab that triggers a Code Manager deployment from the PE Master, as follows.

##### Create a deploy user on PE console

Create a user on **PE console** with permissions to deploy code:

  - Click: **Access Control -> Users -> Add local user** (Specify Full Name and login)
  - Click: **User -> Edit user -> Generate Password reset**
  - Copy the link for password reset and open it with a browser to set the user password (remember it).
  - To assign a new role to the user click **User Roles -> Selected role -> Add user** (Select from menu the User name)
  - For Code Manager is enough to assign the created user to the **Code Deployers** role.

Check [here](https://docs.puppet.com/pe/latest/rbac_user_roles.html) for more details on PE user roles.

##### Request an authentication token as the deploy user

As we have seen PE allows the usage of tokens to manage access to its APIs. Check [Token Based Authentication](https://docs.puppet.com/pe/latest/rbac_token_auth.html) for more details. What we can do with the token depends on the permissions of the user used to request it.

As previously done, when requesting an admin token as root user on the MoM (if we are root on MoM we have enough powers to be considered also admin on the MoM's PE console), now we are going to create a token for the less privileged user that can just deploy code.

As a normal user (in order to not override the previously admin token created under `/root/.puppetlabs/token`) run:

    puppet-access login --lifetime 5y

We are asked to introduce a login and a password, we use the credentials of previously created PE user with Code Deployment permissions.

`Token` is stored in `~/.puppetlabs/token`, to view activities done using the Token, in the PE console, click **Access control > Users > Selected user > Details > Activity tab**.

To manage tokens default lifetime, on the PE console node (remember, the default value is just 5 minutes):

    puppet_enterprise::profile::console::rbac_token_auth_lifetime: 10y

##### Configure WebHook on GitLab

The generated token, under `~/.puppetlabs/token`, has to be added as a [webhook](https://docs.puppet.com/pe/latest/code_mgr_webhook.html) on the control-repo git repository in GitLab.

From the control-repo main page on GitLab click **Settings -> WebHooks** (On older Gitlab **Settings -> Integrations -> WebHooks**) and then add an URL as follows:

    https://<pe_console_hostname>:8170/code-manager/v1/webhook?type=gitlab&token=<puppet_access_token>

Where <pe_console_hostname> is the DNS name of the MoM and <puppet_access_token> is the token.

Keep **Trigger -> Push events** checked and eventually disable **Enable SSL verification** in case of problems and the click on **Add webhook**.

If GitLab and MoM are on the same network we have to configure GitLab to allow requests to the local network: **Admin Area -> Settings -> Network -> OutBound requests -> Check "Allow requests to the local network from web hooks and services"** or, alternatively add the MoM name under **Admin Area -> Settings -> Network -> Whitelist to allow requests to the local network from hooks and services**.

It's possible to test directly the Hook by clicking on **Test -> Push events** under the hook created (listed under **Project Hooks**). On MoM server deployment operations can be controlled in `/var/log/puppetlabs/puppetserver/puppetserver.log`.


### Puppet agents installation

To install the Puppet agent package on an existing client, it's normally enough to run the following command as root:

    curl -k https://puppet-master:8140/packages/current/install.bash | bash

The SSL certificate of the client (created the first time Puppet agent is run on the client) has to be signed manually on the Puppet Server.

This is done by running, as root, on the MoM a command like:

    /opt/puppetlabs/bin/puppetserver ca sign --certname <certname>

for example:

    /opt/puppetlabs/bin/puppetserver ca sign --certname my_client.example.com

Where *my_client.example.com* is the name of the client's certificate, which by default is the machine's FQDN.

On the Puppet server it's possible to show all the Puppet certificates (both signed and not) with:

    /opt/puppetlabs/bin/puppetserver ca list --all


#### Agents certificate recreation

In case a server is recreated with the same FQDN and has already been registered on Puppet, its previous certificate has to be cleaned up before being able to issue a new certificate request.

In order to do this, on the Mom Server, run:

    /opt/puppetlabs/bin/puppetserver ca clean --certname <certname>

Similarly it can happen to need to recreate a certificate for an existing client (not necessarily re-provisioned from scratch). In this case it's also necessary to clean the certificate on the client by running, as root on the client machine:

    /opt/puppetlabs/bin/puppet ssl clean


### Puppet Enterprise HA setup

HA setup should be done by following the [Official docs](https://puppet.com/docs/pe/latest/configure_high_availability.html)

The procedure described here is based on the following servers names:

  - Master of Masters Primary server: **pemaster01.example.com** (the PE server installed following the above procedure)
  - Master of Masters Replica server: **pemaster02.example.com** (the host candidate to be configured as Replica server)
  - Alternative DNS names that clients can use to reach the Puppet servers: **puppet-master** , **puppet-console**

#### Commands to run on Replica server candidate

First the Puppet agent has to be installed on the Replica server candidate, eventually specifying the same dns alt names used for the MoM.

    curl -k https://puppet-console:8140/packages/current/install.bash | bash -s main:dns_alt_names=puppet-master,puppet-console

After a complete and clean Puppet run, this node is configured as a normal PE infrastructure client and is ready to be configured as a replica server, as described in the next section.

#### Commands to run on the Master of Masters Primary server

All the next commands have to be run as root on the previously installed PE Master of Masters server (**pemaster01.example.com**).

    /opt/puppetlabs/bin/puppet infrastructure provision replica pemaster02.example.com

The above command takes several minutes, it uses Puppet tasks and triggers Puppet runs on the specified replica server to install and configure all the needed PE server components. At its end, this command can be used to check the infrastructure status (Master and Replica services should be visible and running):

    /opt/puppetlabs/bin/puppet infrastructure status --verbose

After the status check is positive all the infrastructure clients can be configured to use the replica server in case the primary master is not working (this is done by adding the **servers_list** parameter to the clients' `puppet.conf`, which overrides whatever is set by the **server** parameter).

This is done by running the command:

    /opt/puppetlabs/bin/puppet infrastructure enable replica pemaster02.example.com

Some questions are asked about the PE infrastructure topology, then at the end of the procedure there are notes on how to copy to the replica server a secret key file required to promote the replica to a master.

Similar commands have to be executed:

On replica server (**pemaster02.example.com**) as root:

    mdkir -p /etc/puppetlabs/orchestration-services/conf.d/secrets
    mdkir -p /etc/puppetlabs/puppetserver/ssh

From primary server (**pemaster01.example.com**) the secret key is copied via scp to the replica server (any alternative to have the same file in both place is ok):

    scp /etc/puppetlabs/orchestration-services/conf.d/secrets/keys.json pemaster02:/etc/puppetlabs/orchestration-services/conf.d/secrets/keys.json

Same for the previously created ssh key pair to access the control-repo on GitLab:

    scp /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa pemaster02:/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
    scp /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub pemaster02:/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub

Then, back on replica server (**pemaster02.example.com**) some permissions have to be set:

    chown -R pe-orchestration-services:pe-orchestration-services /etc/puppetlabs/orchestration-services/conf.d/secrets
    chown -R pe-puppet:pe-puppet /etc/puppetlabs/puppetserver/ssh


### Provision additional compile masters

Compile masters are normal Puppet clients which are promoted to act as compile masters.

It's important anyway to add, since the beginning, the right dns_alt_names to these servers' certificate. This is done by installing the Puppet-agent package with a command as follows:

    curl -k https://puppet-console:8140/packages/current/install.bash | bash -s main:dns_alt_names=puppet-master

Once puppet agent has been installed and puppet has run on the node, it's enough to pin the compiler server candidate to the PE Master node group.

In the console, click **Classification**, and in the **PE Infrastructure group**, select the **PE Master** group.

**Enter the Certname** for the compiler, click **Pin node**, and then **commit changes**.

If this is the first compile master created in the infrastructure run Puppet, as root, with the following arguments:

    /opt/puppetlabs/bin/puppet agent -t --server_list puppet-console:8140

In case other compile masters are already created (and are correctly balanced under the puppet-master dns name) just run:

    /opt/puppetlabs/bin/puppet agent -t 

and then run Puppet also on the master:

    /opt/puppetlabs/bin/puppet agent -t

The first time a compile master is added, the following activities have to be performed on the console:

 - Click **Classification** -> **PE Infrastructure** group -> **PE Master** group.
 - Select the **Configuration** tab, and in the **Class** section, in the **pe_repo** class, specify parameters:

        compile_master_pool_address = puppet-master

 - Click **Add data** and **commit** changes.

Then:

 - Click **Classification** -> **PE Infrastructure** group -> **PE Agent** group.
 - Select the **Configuration** tab, and in the **Class** section, in the **puppet_enterprise::profile::agent** class, specify parameters:

        master_uris = ["https://puppet-master"]

 - Click **Add data** and **commit** changes.

Finally run again on the Compile Master:

    /opt/puppetlabs/bin/puppet agent -t 

and then run Puppet also on the Master of Masters:

    /opt/puppetlabs/bin/puppet agent -t

