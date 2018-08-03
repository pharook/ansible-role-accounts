.. _section-role-accounts:

Role **accounts**
================================================================================

Ansible role for convenient management of user accounts on target hosts.

* `Ansible Galaxy page <https://galaxy.ansible.com/honzamach/accounts>`__
* `GitHub repository <https://github.com/honzamach/ansible-role-accounts>`__
* `Travis CI page <https://travis-ci.org/honzamach/ansible-role-accounts>`__


Description
--------------------------------------------------------------------------------

Main purpose of this role is to perform user and group account and access management.
The most important tasks are following:

* Installation and optional configuration of ``OpenSSH`` server
* Installation and optional configuration of ``sudo``
* Management of ``authorized_keys`` file for *root* user
* Management of unprivileged user groups
* Management of unprivileged user accounts
  * manage group memberships
  * manage content of home directory
  * manage ``authorized_keys`` files

.. note::

    This role requires the :ref:`secure registry <section-overview-secure-registry>` feature.

    This role supports the :ref:`template customization <section-overview-customize-templates>` feature.


Requirements
--------------------------------------------------------------------------------

This role does not have any special requirements.


Dependencies
--------------------------------------------------------------------------------

This role is not dependent on any other role.

Following roles have direct dependency on this role:

* :ref:`firewalled <section-role-firewalled>`

  This role uses the information about system administrators and user accounts to
  correctly open the firewall and make system accessible to appropriate users.

* :ref:`alchemist <section-role-alchemist>`
* :ref:`logserver <section-role-logserver>`
* :ref:`puppeteer <section-role-puppeteer>`


Managed files
--------------------------------------------------------------------------------

This role directly manages content of following files on target system:

* ``/etc/sudoers``
* ``/etc/ssh/sshd_config``
* ``/root/.ssh/authorized_keys``
* ``/home/ ... /.ssh/authorized_keys``
* additional files in user home directories as defined by users

  If you provide any content in ``user_files/all`` or ``user_files/[user_name]``
  directories in your playbook root directory, all files and directories will be
  transfered to the user`s home folder on target host.


Role variables
--------------------------------------------------------------------------------

There are following internal role variables defined in ``defaults/main.yml`` file,
that can be overriden and adjusted as needed:

.. envvar:: hm_accounts__configure_ssh

    Enable/disable configuration of OpenSSH server. When set to ``false`` do not
    touch the configuration file.

    * *Datatype:* ``bool``
    * *Default:* ``true``

.. envvar:: hm_accounts__configure_sudo

    Enable/disable configuration of sudo. When set to ``false`` do not touch the
    configuration file.

    * *Datatype:* ``bool``
    * *Default:* ``true``

.. envvar:: hm_accounts__password_authentication

    Enable/disable SSH password authentication.

    * *Datatype:* ``string``, available options: ``"yes"``, ``"no"``
    * *Default:* ``"yes"``

.. envvar:: hm_accounts__admins

    List containing identifiers of all system administrators, that should
    have *root* access to target system. Identifiers must point to valid entry
    in :envvar:`site_users` secret configuration structure.

    * *Datatype:* ``list of strings``
    * *Default:* ``[]`` (empty list)

.. envvar:: hm_accounts__robots

    List containing identifiers of all robotic accounts, that should have *root* access
    to target system. Identifiers must point to valid entry in :envvar:`site_robots`
    secret configuration structure.

    * *Datatype:* ``list of strings``
    * *Default:* ``[]`` (empty list)

.. envvar:: hm_accounts__groups

    Dictionary containing all unprivileged user groups, that should be present
    on target system. The data under each key should currently be empty dictionary,
    in the future perhaps some group options may be possible.

    * *Datatype:* ``dictionary of dictionaries``
    * *Default:* ``{}`` (empty dictionary)

.. envvar:: hm_accounts__users

    Dictionary containing all unprivileged user accounts, that should be present
    on target system. Subdictionaries may contain *groups* attribute, which may
    contain list of all user groups that the user should be member of.

    * *Datatype:* ``dictionary of dictionaries``
    * *Default:* ``{}`` (empty dictionary)


Usage and customization
--------------------------------------------------------------------------------

This role is (attempted to be) written according to the `Ansible best practices <https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html>`__. The default implementation should fit most users,
however you may customize it by tweaking default variables and providing custom
templates.


Variable customizations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most of the usefull variables are defined in ``defaults/main.yml`` file, so they
can be easily overridden almost from `anywhere <https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable>`__.


Template customizations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This roles uses *with_first_found* mechanism for all of its templates. If you do
not like anything about built-in template files you may provide your own custom
templates. For now please see the role tasks for list of all checked paths for
each of the template files.


Installation
--------------------------------------------------------------------------------

To install the role `honzamach.accounts <https://galaxy.ansible.com/honzamach/accounts>`__
from `Ansible Galaxy <https://galaxy.ansible.com/>`__ please use variation of
following command::

    ansible-galaxy install honzamach.accounts

To install the role directly from `GitHub <https://github.com>`__ by cloning the
`ansible-role-accounts <https://github.com/honzamach/ansible-role-accounts>`__
repository please use variation of following command::

    git clone https://github.com/honzamach/ansible-role-accounts.git honzamach.accounts

Currently the advantage of using direct Git cloning is the ability to easily update
the role when new version comes out.


Example Playbook
--------------------------------------------------------------------------------

Example content of inventory file ``inventory``::

    [servers-accounts]
    localhost

Example content of role playbook file ``playbook.yml``::

    - hosts: servers-accounts
      remote_user: root
      roles:
        - role: honzamach.accounts
      tags:
        - role-accounts

Example usage::

    # Run everything:
    ansible-playbook -i inventory playbook.yml

    # Do not touch OpenSSH configuration file:
    ansible-playbook -i inventory playbook.yml --extra-vars '{"hm_accounts__configure_ssh":false}'

    # Do not touch sudo configuration file:
    ansible-playbook -i inventory playbook.yml --extra-vars '{"hm_accounts__configure_sudo":false}'


License
--------------------------------------------------------------------------------

MIT


Author Information
--------------------------------------------------------------------------------

Jan Mach <honza.mach.ml@gmail.com>
