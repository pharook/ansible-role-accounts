.. _section-role-accounts:

Role **accounts**
================================================================================

* `Ansible Galaxy page <https://galaxy.ansible.com/honzamach/accounts>`__
* `GitHub repository <https://github.com/honzamach/ansible-role-accounts>`__
* `Travis CI page <https://travis-ci.org/honzamach/ansible-role-accounts>`__

Main purpose of this role is to perform user and group account management.
The most important tasks are following:

* Installation and optional configuration of ``OpenSSH`` server.
* Installation and optional configuration of ``sudo``.
* Management of ``/root/.ssh/authorized_keys`` file for *root* user.
* Management of unprivileged user groups.
* Management of unprivileged user accounts.
  * Manage group memberships.
  * Manage content of home directory.
  * Manage ``authorized_keys`` files.

**Table of Contents:**

* :ref:`section-role-accounts-installation`
* :ref:`section-role-accounts-dependencies`
* :ref:`section-role-accounts-usage`
* :ref:`section-role-accounts-variables`
* :ref:`section-role-accounts-files`
* :ref:`section-role-accounts-author`

This role is part of the `MSMS <https://github.com/honzamach/msms>`__ package.
Some common features are documented in its :ref:`manual <section-manual>`.


.. _section-role-accounts-installation:

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


.. _section-role-accounts-dependencies:

Dependencies
--------------------------------------------------------------------------------

This role is not dependent on any other role.

Following roles have dependency on this role:

* :ref:`firewalled <section-role-firewalled>`

  This role uses the information about system administrators and user accounts to
  correctly open the firewall and make system accessible to appropriate users.

* :ref:`alchemist <section-role-alchemist>`

  This role uses the user account structure to create system accounts for  projects
  hosted on the server.

* :ref:`logserver <section-role-logserver>`

  This role uses the account features to enable access to server log for
  administrators.

* :ref:`puppeteer <section-role-puppeteer>`

  This role uses the user account structure to create system accounts for  projects
  hosted on the server.


.. _section-role-accounts-usage:

Usage
--------------------------------------------------------------------------------

Example content of inventory file ``inventory``::

    [servers]
    your-server

    [servers_accounts]
    your-server

Example content of role playbook file ``role_playbook.yml``::

    - hosts: servers_accounts
      remote_user: root
      roles:
        - role: honzamach.accounts
      tags:
        - role-accounts

Example usage::

    # Run everything:
    ansible-playbook --ask-vault-pass --inventory inventory role_playbook.yml

    # Do not touch OpenSSH configuration file:
    ansible-playbook --ask-vault-pass --inventory inventory role_playbook.yml --extra-vars '{"hm_accounts__configure_ssh":false}'

    # Do not touch sudo configuration file:
    ansible-playbook --ask-vault-pass --inventory inventory role_playbook.yml --extra-vars '{"hm_accounts__configure_sudo":false}'

It is recommended to follow these configuration principles:

* Create file ``inventory/group_vars/all/users.yml`` and within define database
  of your users::

        site_users:
            userid:
                name: User Name
                name_utf: Úšěř Ňámé
                firstname: User
                lastname: Name
                email: user.name@domain.org
                ssh_keys:
                    - "ssh-rsa AAAA..."
                    - "ssh-rsa AAAA..."
                workstations:
                    - "192.168.1.1"
                    - "::1"
            ...

* Create/edit file ``inventory/group_vars/all/vars.yml`` and within define some sensible
  defaults for all your managed servers::

        # There will probably be same primary administrator on all your servers:
        primary_admin: userid

        # You are probably always using only SSH keys, aren`t you:
        hm_accounts__password_authentication: no

        # There is probably same set of secondary administrators:
        hm_accounts__admins:
          - userid
          - ...

* Use files ``inventory/host_vars/[your-server]/vars.yml`` to customize settings
  for particular servers. Please see section :ref:`section-role-accounts-variables`
  for all available options.


.. _section-role-accounts-variables:

Configuration variables
--------------------------------------------------------------------------------


Internal role variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    This role utilizes the :ref:`secure registry <section-overview-secure-registry>`
    feature.

.. envvar:: site_users

    This is one of the most important configuration variables. It is in fact simple
    JSON database of all known user accounts and their personal data. It is intended
    to be used by other roles, so it is intentionally named without role prefix.

    * *Datatype:* ``dict``
    * *Default:* ``null``
    * *Example:*

    .. code-block:: yaml

        site_users:
            userid:
                name: User Name
                name_utf: Úšěř Ňámé
                firstname: User
                lastname: Name
                email: user.name@domain.org
                ssh_keys:
                    - "ssh-rsa AAAA..."
                    - "ssh-rsa AAAA..."
                workstations:
                    - "192.168.1.1"
                    - "::1"
            ...

.. envvar:: site_robots

    Similarly to the :envvar:`site_users` variable it is simple JSON database of
    all known site hosts. It is intended to be used by other roles, so it is
    intentionally named without role prefix.

    * *Datatype:* ``dict``
    * *Default:* ``null``
    * *Example:*

    .. code-block:: yaml

        site_hosts:
            hostname:
                ssh_keys:
                    - "ssh-dss AAAA..."
            ...

.. envvar:: primary_admin

    Identifier of the primary administrator of paticular server. It must point to valid entry
    in :envvar:`site_users` configuration structure. It is intended to be used by other roles,
    so it is intentionally named without role prefix.

    * *Datatype:* ``string``
    * *Default:* ``null``

.. envvar:: hm_accounts__install_packages

    List of packages defined separately for each linux distribution and package manager,
    that MUST be present on target system. Any package on this list will be installed on
    target host. This role currently recognizes only ``apt`` for ``debian``.

    * *Datatype:* ``dict``
    * *Default:* (please see YAML file ``defaults/main.yml``)
    * *Example:*

    .. code-block:: yaml

        hm_accounts__install_packages:
          debian:
            apt:
              - sudo
              - ...

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

    List containing identifiers of all system administrator accounts, that should
    have **root** access to target system. Identifiers must point to valid entry
    in :envvar:`site_users` configuration structure.

    * *Datatype:* ``list of strings``
    * *Default:* ``[]`` (empty list)

.. envvar:: hm_accounts__robots

    List containing identifiers of all robotic accounts, that should have **root** access
    to target system. Identifiers must point to valid entry in :envvar:`site_robots`
    configuration structure.

    * *Datatype:* ``list of strings``
    * *Default:* ``[]`` (empty list)

.. envvar:: hm_accounts__groups

    Dictionary containing all unprivileged user groups, that should be present
    on target system. Keys of the dictionary should be the desired system names
    of the groups, values should be dictionaries that may contain additional
    parameters to be possibly used by other roles.

    * *Datatype:* ``dictionary of dictionaries``
    * *Default:* ``{}`` (empty dictionary)

.. envvar:: hm_accounts__users

    Dictionary containing all unprivileged user accounts, that should be present
    on target system. Keys of the dictionary should be the desired user account
    names, values should be dictionaries that may contain additional parameters
    to be possibly used by other roles. Currently the only supported additional
    parameter is ``groups``, which

    * *Datatype:* ``dictionary of dictionaries``
    * *Default:* ``{}`` (empty dictionary)


Built-in Ansible variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:envvar:`ansible_lsb['codename']`

    Linux distribution codename. It is used for :ref:`template customizations <section-overview-role-customize-templates>`.


.. _section-role-accounts-files:

Managed files
--------------------------------------------------------------------------------

.. note::

    This role supports the :ref:`template customization <section-overview-role-customize-templates>` feature.

This role manages content of following files on target system:

* ``/etc/sudoers`` *[TEMPLATE]*

  Customizable with following templates::

    ``inventory/host_files/{{ inventory_hostname }}/honzamach.accounts/sudoers.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/sudoers.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/sudoers.j2``
    ``inventory/group_files/servers/honzamach.accounts/sudoers.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers/honzamach.accounts/sudoers.j2``

* ``/etc/ssh/sshd_config`` *[TEMPLATE]*

  Customizable with following templates::

    ``inventory/host_files/{{ inventory_hostname }}/honzamach.accounts/sshd_config.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/sshd_config.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/sshd_config.j2``
    ``inventory/group_files/servers/honzamach.accounts/sshd_config.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers/honzamach.accounts/sshd_config.j2``

* ``/root/.ssh/authorized_keys`` *[TEMPLATE]*

  Customizable with following templates::

    ``inventory/host_files/{{ inventory_hostname }}/honzamach.accounts/authorized_keys.root.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/authorized_keys.root.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/authorized_keys.root.j2``
    ``inventory/group_files/servers/honzamach.accounts/authorized_keys.root.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers/honzamach.accounts/authorized_keys.root.j2``

* ``/home/ ... /.ssh/authorized_keys`` *[TEMPLATE]*

  Customizable with following templates::

    ``inventory/user_files/{{ ivar_user }}/honzamach.accounts/authorized_keys.j2``
    ``inventory/host_files/{{ inventory_hostname }}/honzamach.accounts/authorized_keys.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/authorized_keys.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers_{{ msms_server_type }}/honzamach.accounts/authorized_keys.j2``
    ``inventory/group_files/servers/honzamach.accounts/authorized_keys.{{ ansible_lsb['codename'] }}.j2``
    ``inventory/group_files/servers/honzamach.accounts/authorized_keys.j2``

* Additional files in user home directories for particular users.

  If you provide any content in ``user_files/all/honzamach.accounts/home/`` or
  ``user_files/[user_name]/honzamach.accounts/home/`` directories in your inventory
  directory, all files and directories will be transfered to the user`s home directory
  on target host.


.. _section-role-accounts-author:

Author and license
--------------------------------------------------------------------------------

| *Copyright:* (C) since 2019 Honza Mach <honza.mach.ml@gmail.com>
| *Author:* Honza Mach <honza.mach.ml@gmail.com>
| Use of this role is governed by the MIT license, see LICENSE file.
|
