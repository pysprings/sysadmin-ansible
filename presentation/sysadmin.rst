.. Copyright 2014 Tim Flink
.. Licensed under cc-by-sa 4.0 international

Sysadmin Automation with Ansible
================================

PySprings Python Users Group
Colorado Springs, CO


Topics
======

* Introduction

* Different Sysadmin Tool Types

* Intro to Ansible

* Example playbook to deploy a static blog with comments

* Questions


Introduction
============

While there are many, many tools available to do sysadmin tasks, we'll be focusing
on using Ansible with remote systems running Fedora.

There are certainly other tools available and Ansible isn't in any way limited
to working with Fedora. However, this talk does need to fit into about 2 hours
and covering all tools on all distributions isn't going to happen. I'm most
familiar with Ansible and Fedora, so that's what the talk will focus on.


Why?
=============================

* Time Savings

  - "Oh, I forgot to configure X"

  - Less time spent doing the same configuration tasks over and over again

* Repeatability

  - Same configuration across **all** deployments of the same type

* Ease of use

  - New users don't need to know all the details of a system to deploy or change
    configuration

  - Less effort to update configuration means that folks are more likely to
      make the updates



System Administration Tools
===========================


Configuration Management
========================

Most if not all of the early sysadmin automation tools were used for configuration
management.

* Examples:

  - cfengine

  - puppet

* Limited to configuring machines once they had an initial configuration

* This is great, but why can't we have more?

  - Running a command across all machines of a certain type

  - Instantiating VMs and cloud machines


Centralization vs Decentralization
==================================

To talk about centralized automation versus decentralized automation, we'll look
at two examples of popular systems:

Puppet
------

* Configuration is done with a ruby-ish DSL in files on a central machine

* Client program checks in with central server on a regular basis and performs
  any required changes


Ansible
-------------

* Configuration is done via actions in yaml files but can utilize static files,
  jinja2 templates and other mechanisms.

* Operations are idempotent

* Changes are pushed out from where the playbook is executed


What is Ansible?
================

According to `Ansible's documentation <http://docs.ansible.com/>`_::

  Ansible is an IT automation tool. It can configure systems, deploy software,
  and orchestrate more advanced IT tasks such as continuous deployments or zero
  downtime rolling updates.


Why Ansible?
============

* Push based
* Extensible
* Usable without a system agent (works over plain SSH)
  - Don't need to dedicate a host as puppetmaster
* Good documentation
* Easier to handle slight differences between systems in a certain class


Getting Set Up
==============

Installing Ansible
==================

The best way to install ansible is by using system packages.

* Fedora/CentOS/RHEL: `yum install ansible`
* Debian/Ubuntu: `apt-get install ansible`

For this talk, we'll also be using `Pelican <http://docs.getpelican.com/>`_

Virtual Machine
===============

To get a good base for working with ansible without paying for a cloud provider,
create a virtual machine.

Ansible is not limited to a specific distro but the this talk was written with
Fedora 20 in mind. `A prebuilt minimal cloud image is available for download
<http://download.fedoraproject.org/pub/fedora/linux/releases/20/Images/x86_64/Fedora-x86_64-20-20131211.1-sda.qcow2>`_.


SSH
===

Ansible relies heavily on SSH to execute remote commands. The easiest and best
way to facilitate this is through the use of ssh private keys::

  ssh-copy-id fedora@<ip address>

This will allow you to connect to the VM over ssh without using the system
password.


Ansible Concepts
================


Inventory
=========

The core of ansible is a system which provides a method for running commands on
specific remote machines. These machines are specified as inventory in an
ini style text file

::

  [production]
  blog.somedomain.org

  [staging]
  192.168.0.5

This specifies two machines, one in a production group and the other in a
staging group.


Running Remote Commands
=======================

To give a simple example, let's find out what kernel is running on our local
machines

.. class:: prettyprint lang-bash

::

  ansible -i local all -u fedora -a 'uname -a'


Variables
=========

Ansible uses jinja2-style variables. To substitute a variable with the name
`variable_name`, you would use::

  {{ variable_name }}

One unfortunate quirk of this variable style is that yaml can parse them as
dictionaries and they often need to be quoted::

  "{{ variable_name }}"


Plays and Playbooks
===================

While the ability to run remote commands is the core of ansible, we need to group
these commands in order to configure entire systems. "Plays" are groups of
commands and "Playbooks" are groups of plays, specified in yaml files.

.. class:: prettyprint lang-yaml

::

  - name: example command
    somemodule: src={{ source }} dest={{ dest }}

Example Playbook
================

Looking at update_host.yml::

    # requires --extra-vars="target=somevhostname yumcommand=update"

    - name: update hosts
      hosts: "{{ target }}"
      sudo: true

    tasks:
    - name: expire-caches
      action: command yum clean expire-cache

    - name: yum -y {{ yumcommand }}
      action: command yum -y {{ yumcommand }}
      async: 7200
      poll: 50

Running the Example
===================

From the `CHECKOUT_ROOT/ansible` directory::

  ansible-playbook -i blog --private-key=/path/to/key -e 'target=<target ip>' -u fedora update_host.yml

Remember to do substitutions:

* actual key's path for `/path/to/key`

* machine's IP address for `<target ip>`


Getting More Complicated
========================

The provided example playbook does the following:

* Configure a remote host for:
  - Serving a static blog over http/https
  - Hosting comments in that blog
  - Syncing blog contents from a remote machine

* Build a static blog using `Pelican <http://docs.getpelican.com/>`_.

* Synchronize the static blog output with remote host


Playbook Configuration
======================

Playbooks can be configured in multiple ways:

* Command line options (using `-e 'variable=value`)

* Dedicated variable files imported by the playbook

* Inline variables in plays


File Management
===============

A lot of configuration is done via configuration files. With ansible, you can
copy static files, generate files from jinja2 templates, modify existing files
and other actions.

For the sake of simplicity, we'll be talking about templates and static files.


Copying Files
=============

The ansible module for copying files to a remote location is pretty straightforward::

  copy: src=somefile.conf dest=/etc/somefile.conf owner=root group=root mode=0755

This will copy the `somefile.conf` file (living in one of the default locations)
to the remote host as `/etc/somefile.conf`. The remote file will be owned by the
`root` user and the `root` group with permissions of `755`.


Generating Files from Templates
===============================

The syntax for generating files from templates is almost identical to copying files::

  template: src=somefile.conf dest=/etc/somefile.conf owner=root group=root mode=0755

The main differences in basic usage are: 

* default locations for files

* name of module


File Locations
==============

While you can specify full paths to the files used by ansible, you can also rely
on conventions which make the playbooks much less static.

In directories relative to the playbook:

* Templates in `templates/`

* Files in `files/`

* Tasks in `tasks/`


Roles
=====

One method for reusing configuration is through the use of roles. These roles
are usually specific to a certain function (mail server, web server etc.) and
if written well, can be applied to many hosts.

One way to think of roles is as self-contained sub-playbooks - each with their
own set of tasks, files, templates and defaults.


Dissecting the Blog Playbook
============================

See the file `CHECKOUT_ROOT/ansible/blog.yml` for the contents.


Running the Blog Playbook
=========================

From the `CHECKOUT_ROOT/ansible` directory::

  ansible-playbook -i blog --private-key=/path/to/key -e 'target=<target ip>' -u fedora update_host.yml

Remember to do substitutions:

* actual key's path for `/path/to/key`

* machine's IP address for `<target ip>`


Questions and Next Steps
========================


Next Steps
==========

* `Ansible Documentation <http://docs.getpelican.com/>`_

Other Examples
--------------

* `Ansible Examples <https://github.com/ansible/ansible-examples>`_

  - Maintained by the Ansible folks

* `PySprings Playbook <https://bitbucket.org/pysprings/ansible-playbooks>`_

  - Probably not the best example right now, needs a lot of updating

  - Help would certainly not be frowned on (hint, hint)



Questions
=========

If you have questions at a later date, ask!
 * pysprings mailing list
 * #pysprings on Freenode IRC



.. unused slides
.. Tool Overview - Fabric
.. ======================
.. 
.. * Easy to do both local and remote commands
.. 
.. * Good for automating specific tasks like:
.. 
..   - Deploying a single application
.. 
..   - Updating a deployment with new code from version control
.. 
.. 
.. Tool Overview - Ansible
.. =======================
.. 
.. 
.. * Good for more complicated sysadmin tasks
.. 
..   - Lends more towards running everything on remote machines
.. 
.. * Doesn't require a system agent
.. 
.. 
.. Tool Overview - Saltstack
.. =========================
.. 
.. * Fast operations due to use of zeromq
.. 
.. * Requires a system agent
.. 
.. * Good for more complicated sysadmin tasks
.. 
..  
