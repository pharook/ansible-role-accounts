.. _section-cookbook-roles-cleanup:

Account management
================================================================================


.. _section-cookbook-roles-accounts-newuser:

Add new user
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to add a brand new user to your server management environment please follow
these simple steps:

1. Add appropriate record to file ``inventory/group_vars/all/users.yml``, subkey
   :envvar:`site_users`::

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

2. Now use role :ref:`accounts <section-role-accounts>` to enable the user acces to appropriate
   systems. You may use ``inventory/group_vars/all/vars.yml`` file to define global
   defaults for all servers and/or ``inventory/host_vars/[server-name]/vars.yml``
   to fine tune the configuration for particular hosts. In any case, you will
   probably use either configuration variable :envvar:`hm_accounts__admins` to grant administrator access,
   or :envvar:`hm_accounts__users` to grant unprivileged access.

The :envvar:`site_users` dictionary is acting as a registry, which allows you to
define users along with all their metadata at a single place. All roles then have
access to all that data and you can just point out the users they should take into
account.

Then execute the role::

    apbf role_accounts.yml

Please refer to role :ref:`accounts <section-role-accounts>` for more details.
