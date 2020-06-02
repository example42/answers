
# Continuous Delivery for Puppet Enterprise (CD4PE) setup

Installation of CD4PE involves:

- Some preparation works in order to be able to operate in an closed environment without Internet access
- Hiera and classification configurations for the node where CD4PE has to be installed
- Puppet run (using the new PE servers) to installe CD4PE
- Some manual configurations on the CD4PE web interface

Let's see these steps in detail, in this example hostnames have fqdn as follows:

- PE server (MoM): *puppet-console.example.com* ()
- CD4PE: *cd4pe.example.com*
- GitLab server: *git.example.com*

### Prerequisite operations

Before being able to install CD4PE some activities must be done in advance.

#### Docker images in internal repo

CD4PE needs 2 different docker images:

- the **cd4pe** image, which actually contains the cd4pe application
- the **puppet-dev-tools** image which is used in the Hardware Job agents to perform the various CI jobs

In a default installation these images are downloaded directly from the Internet. In an infrastructure without Internet access they have been copied to a local registry.

The local registry, in this document where GitLab is used for reference, can be configured on GitLab itself.

Here a new group with public access has to be created: we call it **public_repo** in this example, and 2 different projects, used just to host docker images, have to be added there:

- [puppet_cd4pe](https://git.example.com/public_repo/puppet_cd4pe/container_registry) for the cd4pe images
- [puppet_devtools](https://git.example.com/public_repo/puppet_devtools/container_registry) for the puppet-dev-tools images

Both of them have to be configured as public, into a public group, to avoid complications related to automation of images download and because they contain images which are already available publically on the internet.

In order to push to the GitLab registry the needed docker images the following commands have to be executed on an internal host able to use the Internet, eventually via proxy servers, and the internal GitLab instance on port 8443.

For the cd4pe image (here is used version 3.6.1) steps are as follows (Similar steps have to be done for each new version).

first, check if docker is present:

    docker --version

If needed, configure http proxy for docker daemon to download public images 

    mkdir -p /etc/systemd/system/docker.service.d
    cat > /etc/systemd/system/docker.service.d/proxy.conf << EOF
    [Service]
    Environment="HTTPS_PROXY=http://proxy.example.com:8080" "HTTP_PROXY=http://proxy.example.com:8080" "NO_PROXY=.example.com"
    EOF
    systemctl daemon-reload
    systemctl restart docker

Login to the Internal Registry using the credentials of a valid user on GitLab

    docker login git.example.com:8443

Download the cd4pe image of the needed version

    docker pull index.docker.io/puppet/continuous-delivery-for-puppet-enterprise:3.6.1

Tag the image 

    docker tag puppet/continuous-delivery-for-puppet-enterprise:3.6.1 git.example.com:8443/public_repo/puppet_cd4pe:3.6.1

Push the image to the internal registry

    docker push git.example.com:8443/public_repo/puppet_cd4pe:3.6.1

Similarly for the puppet-dev-tools image used in Job Hardware:

Download the image with tag gosu (which has the gosu package already installed. This probably needed only on legacy Distelli agents)

    docker pull index.docker.io/puppet/puppet-dev-tools:gosu 

Tag the image

    docker tag puppet/puppet-dev-tools:gosu  git.example.com:8443/public_repo/puppet_devtools:gosu

Push the image to the internal registry

    docker push git.example.com:8443/public_repo/puppet_devtools:gosu


#### Gitlab configurations

A new user, here called **puppet_cdpe** has to be created in order to handle all the activitis done by CD4PE:

- User is created and is given **Maintaner** access to the control-repo repository.
- A new impersonation token has to be created wih **api** and **read_user** scopes.

#### Puppet Enterprise configuration

On the Puppet Enterprise console a new user has to be created with some specific permissions in order to be used in CD4PE

- Create the account **cdpe_user**
- Create a **Cdpe_User_Role** which has the following permissions:

| Type | Permission	| Object |
| ---- | ----------	| ------ |
| Job orchestrator	Start, stop, and view jobs | - |
| Node groups | Create, edit, and delete child groups | All |
| Node groups | View | All |
| Node groups | Edit configuration data | All |
| Node groups | Set environment | All |
| Nodes	| View node data from PuppetDB | - |
| Puppet agent	| Run Puppet on agent nodes | - |
| Puppet environment | Deploy code | All |
| Puppet Server | Compile catalogs for remote nodes	| - |
| Tasks	| Run tasks | cd4pe_jobs::run_cd4pe_job on all nodes |

- Assign the user  **cdpe_user** to the Cdpe_User Role
- Click on Generate Password reset to configure a password for the user. This will be needed when configuring Puppet Enterprise integration on CD4PE.


In case custom cd4pe Job involve the execution of tasks via PE console, another user has to be created to with the permissions to run the relevant job on CD4PE Job Hardware nodes:

- Create the account **task_user**
- Create a **task_user_Role** which has the following permissions:

| Type | Permission	| Object |
| ---- | ----------	| ------ |
| Job orchestrator	Start, stop, and view jobs | - |
| Nodes	| View node data from PuppetDB | - |
| Tasks	| Run tasks | Selected tasks on all nodes |

- Assign the user  **task_user** to the task_user Role
- Click on Generate Password reset to configure a password for the user.
- Generate a token with the credentials of the task_user. On a node where Puppet client tools have been installed (can be the MoM, but be sure to run the command as a user who has not a token already present in his `~/.puppetlabs/token`:

        /opt/puppetlabs/bin/puppet-access login --lifetime 420d

- Copy the token generated under `~/.puppetlabs/token` in the Hiera files for all the nodes which are used as Hardware Job for the relevant infrastructure.

### CD4PE installation

CD4PE is an application running inside a container and installed using the official [puppetlabs/cd4pe](https://github.com/puppetlabs/puppetlabs-cd4pe) which has been added to the control-repo's Puppetfile.

Installation is hence done via Puppet itself on clients which have the following hiera settings (current settings apply for a standalone setup of a single CD4PE instance):

    ---
    # Disable sending of usage analytics 
    cd4pe::analytics: false

    # Url of the cd4pe image to use (must be pushed to the local registry before)
    cd4pe::cd4pe_image: 'git.example.com:8443/public_repo/puppet_cd4pe'

    # Version of the cd4pe image to use
    cd4pe::cd4pe_version: '3.6.1'

    # Name of the PE server from where to download PE packages like (pe-postgresl)
    cd4pe::repo::config::master: 'puppet-console.example.com'

    # DNS resolvable name by which the CD4PE server is expected to be reached
    cd4pe::resolvable_hostname: 'cd4pe.example.com'

    # Avoid automatic addtion of entries to the cd4pe container's /etc/hosts from the host
    cd4pe::manage_pe_host_mapping: false

    # Extra parameters to pass to docker container
    cd4pe::cd4pe_docker_extra_params:
      - --env JVM_ARGS="-Duser.timezone=UTC -Xmx2048M -Xms2048M -Dlog4j.configurationFile=/etc/service/pfi/log4j.properties" # More RAM for Java
      - "--add-host cd4pe.example.com %{::ipaddress}"        # Custom entries to add to the container's /etc/hosts file

We might need to configure Docker in /etc/docker/daemon.json to allow insecure registries, with something like:

    { insecure-registries: ["git.example.com:8443"] }


### CD4PE configuration

After completing the docker install and puppet run, connect to cd4pe on http://cd4pe.example.com:8080/ and proceed with the configuration

- Create a root administrator account

    Login: admin@example.com
    Password: xxxx

- Endpoints configurations

    WebUI: http://cd4pe.example.com:8080/
    Backend Service: http://cd4pe.example.com:8000/
    Agent Service: dump://cd4pe.example.com:7000/

- Configure Storage, in this case local disk on the host is used

    Disk
    Storage Bucket: cd4pe

After these configurations, when a user tries to connect to the UI a Workspace Account creation is requested. Multiple similar accounts can be created. 

    Login: puppet@example.com
    Password: xxxx

Then with this user (or also as root user) a new Workspace can be created and configured:

    Select source control: Gitlab
    Host: https://git.example.com
    Token: The one previously created on GitLab for puppet_cd4pe account 
    Select Organization: Infrastructure Automation / Puppet
    Select Repository: Puppet Control Repo (Private Repository)
    Select Branch: master
    Control Repo Name: control-repo

Puppet Enterprise credentials configuration:

    Name: Puppet-Enterprise-Master-of-Masters
    Puppet Enterprise Console Address: puppet-console.example.com
    Authorization: Basic Authorization
    Puppet Enterprise Username: cdpe_user
    Puppet Enterprise Password: [ The one previously inserted on Puppet Enterprise ]

After this configuration the following settings should appear:

    PuppetDB Service: puppet-console.example.com:8081
    Code Manager Service: puppet-console.example.com:8170
    Orchestrator Service: puppet-console.example.com:8143
    Classifier Service: puppet-console.example.com:4433
    CA Certificate: .... 
    Puppet Server Service: puppet-console.example.com:8140

#### Pipelines configurations

We configure 2 pipelines for the control repo:

##### Regex pipeline

To create a new Pipeline to test feature branches:

  - Select the control-repo previously added in the workspace
  - Click on **Add Pipeline** (+ icon)
    - Select **Branch Regex**
    - In Configure Regex write: **\d+-feature_.***
    - Click **Add Pipeline** and then **Done**
    - Check that Pipeline trigger is set to **Commit**
  - Under the new Regex Pipeline created, click **+ Add default pipeline**
  - If wanted, add a Lint check job (which has to be configured, as later shown). Keeping **Auto Promote** checked with **All succeeded** selection, click **+ Add stage**
    - Stage Name: **Lint checks**
    - Select Item: **Jobs**
    - Select jobs:
      - **control-repo-lint-validation**
    - Click **Add Stage** and then **Done**
  - Keeping **Auto Promote** checked with **All succeeded** selection, click **+ Add a deployment**
    - Select Item: **Deployment**
    - SELECT PUPPET ENTERPRISE INSTANCE: **<USE existing selection>**
    - SELECT A DEPLOYMENT POLICY: **Built-in deployment policies**
    - Select **Feature branch policy**
    - Click **Add Deployment to Stage**, click **Done**

##### Master pipeline

Then we can configure the Pipeline for the master branch:

  - Select the control-repo previously added in the workspace
  - Click on **Add Pipeline** (+ icon)
    - Select **master**
    - Set Pipeline trigger to **Commit**
    - Click **Add Pipeline** and then **Done**

  - Keeping **Auto Promote** checked with **All succeeded** selection, click **+ Add stage**
    - STAGE NAME: **Deployment to production** 
    - SELECT ITEM: **Deployment**
    - SELECT PUPPET ENTERPRISE INSTANCE: **<USE existing selection>**
    - SELECT A NODE GROUP: **Production environment - Puppet environment: production**
    - SELECT A DEPLOYMENT POLICY: **Built-in deployment policies**
    - Select **Direct deployment policy**
    - Optionally set max_node_failure to the number of nodes allowed to fail
    - Keep noop to false
    - Click **Add Stage**, click **Done**


#### Jobs configuration

The following custom Jobs can be added to the Jobs list.

If we use [Tiny Puppet](https://github.com/example42/puppet-tp) we can leverage on tp tests and create a job able to run them:

- custom-tp-test-shared-hardware
  - Job Name: **custom-tp-test-shared-hardware**
  - Description: **Tp test on shared hardware**
  - Commands Job: `puppet task run tp::test -q 'inventory { facts.osfamily ~ "^RedHat" }' --token-file /root/.puppetlabs/token`
  - Job Hardware: **Run on puppet hardware**
  - Docker Configuration: Keep **off** Run this job inside of a Docker container
  - Capabilities: Select **Docker**


If we want to add Lint checks, as in the previous example forwith the Regex pipeline, we have to add the relevant job:

- control-repo-lint-validation
  - Job Name: **control-repo-lint-validation**
  - Description: **Validate that the control repo linting is correct**
  - Commands Job: `rake -f /Rakefile lint`
  - Job Hardware: **Selected - run on puppet hardware**
  - Docker Configuration: **Selected - Run this inside of a docker container**
  - Docker Image Name: **git.example.com:8443/public_repo/puppet_devtools:gosu**
  - Capabilites: Selected - **Docker**

IMPORTANT: Besides the addition of new custom Jobs, in each of the existing jobs the docker image has to be changed is an internal registry is used.
Click on edit, and set as Docker Image Name: **git.example.com:8443/public_repo/puppet_devtools:gosu**
