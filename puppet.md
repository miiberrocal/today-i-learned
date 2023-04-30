## Puppet

> Infrastructure as a code.

Puppet documentation should be your first point of contact por any Puppet issues.

**Architecture**

> Terms, Facts, Catalog and Reports

Agent (A) sends data about it current state to Master (M) using Facts.

Puppet uses Facter command internally to gather information about puppet agents

Facter: 

* system inventory tool

* returns list of key value pairs called <u>Facts</u>

Facts provides: IP address | hostname | FQDN | kernel version

Puppets uses facts and compiles Catalog

Catalog specifies how agent should be configured

At the master: facter's compared against manifest, if it has additional functionality they're compiled into Catalogs and sent over to the client.

| Server                        | Node                                                       |
| ----------------------------- | ---------------------------------------------------------- |
| 1. Install Puppet Server (PS) |                                                            |
| 2. Start PS service           |                                                            |
|                               | 3. Install puppet agent                                    |
|                               | 4. Start agent service and create SSL certificate          |
| 5. Sign certificate           |                                                            |
| 6. Write manifest             |                                                            |
|                               | 7. Run agent --> manifest/catalog are executed on the node |



#### Profiles

wrapper classes > multiple component modules

Classes from component modules are always declared via a profile, they're never assigned directly to a node.

Own all the class parameters for their component classes

**Modules** 

Puppet Forge

Manifests | templates | files | facts

Specific directory structure

Allow you to split your code into multiple manifests

/etc/puppet/modules

1. create manifest --> 
1. compile manifest into catalog -->
1. deploy catalog into the clients --> 
1. execute catalog on the client by the agent --> 
1. Client configured to desired state

**Resources**

resource_type { 'resource_name'
  attribute => value
  ...
}

user { 'mitchell':
  ensure     => present,
  uid        => '18',
  shell      => '/bin/bash',
  home       => '/home/mitchell'
}

Puppet Resources: inbuilt functions that runs at the backend to perform underlying operations in Puppet.

Functions that perform individual tasks (e.g. create user)

All operations from puppet agent are performed with the help of puppet resources.

Three kinds:

* Core/Built-in
* Defined
* Custom

$ puppet --help --> all commands

$ puppet parser validate demouser.pp --> verify files syntax

$ puppet apply demouser.pp -noop

$ puppet describe [resource type] | more --> find attributes to be used on puppet code

**Classes**

class example_class {
  ...
  code
  ...
}

Collection of puppet resources bundle together as single unit.

Makes structure re-usable and organized

You can create multiple resources in .pp files, bundle together and logically assign it a name. For this you need to:

1. Define a class using class definition syntax
2. To use the class in code, utilizing "include" --> class declaration

**Manifest** (Puppet programs)

Directory with DSL files .pp extension

All .pp files with puppet code should reside inside manifests directory

$ puppet config print --> shows puppet.conf file

site.pp file: main entry point for entire puppet network, main starting point on Catalogs compilation, also refered as "site manifest".  Where all puppet classes will be declared (included).

* located at /etc/puppetlabs/code/environments/production/manifests/site.pp

**Modules**

Collection of files and directories (manifests, class definitions and any other dependent files that work together and follows specific directory structure)

create --> check --> test --> run puppet nodes

Declarative programming paradigm used by Puppet DSL

Master-agent deployment mode: production use case

Standalone deployment mode: development/testing/PoCs

**Deployment Models**

Push vs Pull

Push: master pushes to individual servers

Pull: individual servers contacts master and download configs/softwares (iniciated by agents)

1. Connection
2. Agent sends state data (facts) for Master
3. Master sends Catalog
4. Agent receives Catalog and executes configuration drafts
5. Agent responds back to Master indicating config was applied and completed

**Setup Puppet Master** (self-contained) 

Set hostname for puppet master

* use FQDN (make it persistent - reboots)
* Etc/host file
* host min. requirements: 2 processor cores (>= 1GB RAM) | 1000 nodes --> 2-4 processor cores (>= 4GB RAM)
* Allow income connections on port 8140
* master and agent should have same time set up
* NTP for puppet env

**Connect Master - Agent**

/etc/puppetlabs/puppet/puppet.conf

* Certname: FQDN of teh respective server
* Server: points to the FQDN of puppet master

$ puppet agent --test --> verify master/agent connectivity

You also need to stablish secure connectivity between servers: SSL certificates

Certification request's generated when starting puppet services on agent OR when performing ad hoc puppet run

**Certification Management**

Create file autosign.conf at /etc/puppetlabs/puppet

Always restart puppet master services after changing config files

| Main Certificate commands |
| ------------------------- |
| puppet cert list          |
| puppet cert list --all    |
| puppet cert sign          |
| puppet cert clean         |
| puppet cert generate      |

| On Agent                | On Master                        |
| ----------------------- | -------------------------------- |
| systemctl start puppet  |                                  |
| systemctl status puppet |                                  |
|                         | puppet cert list                 |
|                         | puppet cert list --all           |
|                         | puppet cert sign [node hostname] |
|                         | puppet agent -tv                 |
|                         |                                  |

Automate Signing

$ vi autosign.conf (at cd /etc/puppetlabs/pupet)

​	*.example.com --> sign all certificates generated by domain example.com

$ systemctl status puppet

$ systemctl restart puppet

