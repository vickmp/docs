.. _upgrade_single:

================================================================================
Upgrading Single Front-end Deployments
================================================================================

If you are upgrading from a 6.0.x installation you only need to follow a reduced set of steps. If you are running a 5.12.x version or older, please check :ref:`these set of steps <upgrading_from_previous_extended_steps>` (some additional ones may apply, please review them at the end of the section).

.. important::

    Users of the Community Edition of OpenNebula can upgrade from the previous stable version if they are running a non-commercial OpenNebula cloud. In order to access the migrator package a request needs to be made through this `online form <https://opennebula.io/get-migration>`__.

.. _upgrade_60:

Upgrading from 6.0.x
^^^^^^^^^^^^^^^^^^^^^

This section describes the installation procedure for systems that are already running a 6.0.x OpenNebula. The upgrade to OpenNebula |version| can be done directly following this section, you don't need to perform intermediate version upgrades. The upgrade will preserve all current users, hosts, resources and configurations.

When performing a minor upgrade OpenNebula adheres to the following convention to ease the process:

  * No changes are made to the configuration files, so no configuration file will be changed during the upgrade.
  * Database versions are preserved, so no upgrade of the database schema is needed.

When a critical bug requires an exception to the previous rules it will be explicitly noted in this guide.

Upgrading a Federation and High Availability
================================================================================

You need to perform the following steps in all the HA nodes and all zones. You can upgrade the servers one by one to not incur any downtime.

Step 1 Stop OpenNebula Services
===============================

Before proceeding, make sure you don't have any VMs in a transient state (prolog, migr, epil, save). Wait until these VMs get to a final state (run, suspended, stopped, done). Check the :ref:`Managing Virtual Machines guide <vm_guide_2>` for more information on the VM life-cycle.

Now you are ready to stop OpenNebula and any other related services you may have running, e.g. Sunstone or OneFlow. It's preferable to use the system tools, like `systemctl` or `service` as `root` in order to stop the services.

Step 2 Upgrade Front-end to the New Version
===========================================

Upgrade the OpenNebula software using the package manager of your OS. Refer to the :ref:`Single Front-end Installation guide <frontend_installation>` for a complete list of the OpenNebula packages installed on your system. Package repos need to be pointing to the latest version (|version|).

For example, in CentOS/RHEL simply execute:

.. prompt:: text # auto

    # yum upgrade opennebula

For Debian/Ubuntu use:

.. prompt:: text # auto

   # apt-get update
   # apt-get install --only-upgrade opennebula

.. note:: If Sunstone is running as an Apache Phusion Passenger app, please check the :ref:`filesystem permissions <sunstone_fs_permissions>`.

Step 3 Upgrade Hypervisors to the New Version
=============================================

You can skip this section for vCenter Hosts.

Upgrade the OpenNebula node KVM or LXD packages, using the package manager of your OS.

For example, in a rpm-based Linux distribution simply execute:

.. prompt:: text # auto

   # yum upgrade opennebula-node-kvm

For deb-based distros use:

.. prompt:: text # auto

   # apt-get update
   # apt-get install --only-upgrade opennebula-node-kvm

.. note:: If you are using LXD the package is ``opennebula-node-lxd``.

Update the Drivers
==================

You should now be able to start OpenNebula as usual, running ``service opennebula start`` as ``root``. At this point, as ``oneadmin`` user, execute ``onehost sync`` to update the new drivers in the Hosts.

.. note:: You can skip this step if you are not using KVM Hosts, or any Hosts that use remote monitoring probes.

Testing
=======

OpenNebula will continue the monitoring and management of your previous Hosts and VMs.

As a measure of caution, look for any error messages in oned.log, and check that all drivers are loaded successfully. After that, keep an eye on oned.log while you issue the onevm, onevnet, oneimage, oneuser, onehost **list** commands. Try also using the **show** subcommand for some resources.

Restoring the Previous Version
==============================

If for any reason you need to restore your previous OpenNebula, simply uninstall OpenNebula |version|, and install again your previous version. After that, update the drivers if needed, as outlined in Step 12.

.. _upgrading_from_previous_extended_steps:

Upgrading from 5.6.x+
^^^^^^^^^^^^^^^^^^^^^

.. important::

    If you haven't done so, please enable the :ref:`OpenNebula and needed 3rd party repositories <setup_opennebula_repos>` before attempting the upgrade process. If you want to use Docker related functionality of OpenNebula and/or OpenNebula Edge Clusters provisioning you'll need to follow :ref:`this for RedHat <install_docker_deps_rh>` or :ref:`this for Debian <install_docker_deps_deb>` distributions.

Step 1. Check Virtual Machine Status
================================================================================

Before proceeding, make sure you don't have any VMs in a transient state (prolog, migr, epil, save). Wait until these VMs get to a final state (running, suspended, stopped, done). Check the :ref:`Managing Virtual Machines guide <vm_guide_2>` for more information on the VM life-cycle.

Step 2. Set All Hosts to Offline Mode
================================================================================

Set all Hosts to offline mode to stop all monitoring processes.

If you are upgrading from version 6.2+. Use ``onezone disable <zone_id>`` to make sure that no operation changing OpenNebula state are executed.

Step 3. Stop OpenNebula
================================================================================

Stop OpenNebula and any other related services you may have running: OneFlow, EC2, and Sunstone. It's preferable to use the system tools, like `systemctl` or `service` as `root` in order to stop the services.

.. warning:: Make sure that every OpenNebula process is stopped. The output of `systemctl list-units | grep opennebula` should be empty.

Step 4. Back-up OpenNebula Configuration
================================================================================

Back-up the configuration files located in **/etc/one** and **/var/lib/one/remotes/etc**. You don't need to do a manual backup of your database; the ``onedb`` command will perform one automatically.

.. prompt:: text # auto

    # cp -ra /etc/one /etc/one.$(date +'%Y-%m-%d')
    # cp -ra /var/lib/one/remotes/etc /var/lib/one/remotes/etc.$(date +'%Y-%m-%d')

Step 5. Upgrade OpenNebula Packages Repository
================================================================================

In order to be able to retrieve the packages for the latest version, you need to update the OpenNebula packages repository. The instructions for doing this are detailed :ref:`here <repositories>`.

Step 6. Upgrade to the New Version
================================================================================

Ubuntu/Debian

.. prompt:: text # auto

    # apt-get update
    # apt-get install --only-upgrade opennebula opennebula-sunstone opennebula-gate opennebula-flow opennebula-provision python3-pyone

RHEL

.. prompt:: text # auto

    # yum upgrade opennebula opennebula-sunstone opennebula-gate opennebula-flow opennebula-provision python-pyone python3-pyone

.. important::

    When **upgrading** an existing deployment which could be running OpenNebula older than 5.10.0 anytime in the past, you might need to upgrade also required Ruby dependencies with script ``install_gems`` if you are not yet using the shipped Ruby gems (i.e., when symbolic link ``/usr/share/one/gems`` doesn't exist on your Front-end)!

    If unsure, run ``/usr/share/one/install_gems`` and the script warns if action is not relevant for you. For example:

    .. prompt:: text # auto

        # /usr/share/one/install_gems
        WARNING: Running install_gems is not necessary anymore, as all the
        required dependencies are already installed by your packaging
        system into symlinked location /usr/share/one/gems. Ruby gems
        installed by this script won't be used until this symlink exists.
        Remove the symlink before starting the OpenNebula services
        to use Ruby gems installed by this script. E.g. execute

            # unlink /usr/share/one/gems

        Execution continues in 15 seconds ...

    Read :ref:`this <ruby_runtime>` for more information.

Community Edition
------------------

There is an additoinal step if you are upgrading OpenNebula CE. After you get the `opennebula-migration-community package <https://opennebula.io/get-migration>`__, you need to install it in the OpenNebula Front-end.

RHEL
~~~~~~~~~~~

.. prompt:: bash # auto

		# rpm -i opennebula-migration-community*.rpm

Debian/Ubuntu
~~~~~~~~~~~~~

.. prompt:: bash # auto

		# dpkg -i opennebula-migration-community*.deb

Step 7. Update Configuration Files
================================================================================

If you haven't modified any configuration files, you can skip this step and proceed to the next one.

In HA setups it is necessary to replace in the file ``/etc/one/monitord.conf`` the default value ``auto`` of ``MONITOR_ADDRESS`` attributed to the virtual IP address used in RAFT_LEADER_HOOK and RAFT_FOLLOWER_HOOK in ``/etc/one/oned.conf``.

Community Edition
-----------------

In order to update the configuration files with your existing customizations you'll need to:

#. Compare the old and new configuration files: ``diff -ur /etc/one.YYYY-MM-DD /etc/one`` and ``diff -ur /var/lib/one/remotes/etc.YYYY-MM-DD /var/lib/one/remotes/etc``. You can use graphical diff-tools like ``meld`` to compare both directories; they are very useful in this step.
#. Edit the **new** files and port all the customizations from the previous version.

Enterprise Edition
------------------

If you have modified configuration files, let's use ``onecfg`` to automate the configuration file upgrades.

After updating the packages, ``onecfg status`` command should show that there's a new configuration available:

.. prompt:: text # auto

    # onecfg status
    --- Versions -----------------
    OpenNebula: 6.2.0
    Config:     6.0.3

    --- Backup to Process ---------------------
    Snapshot:    /var/lib/one/backups/config/backup
    (will be used as one-shot source for next update)

    --- Available updates --------
    New config: 6.2.0
    - from 6.0.0 to 6.2.0 (YAML,Ruby)

After checking the state of configuration, in most cases running the following command without any extra parameters will suffice, as it will upgrade the probes based on the internal configuration version tracking of the currently installed OpenNebula.

.. prompt:: text # auto

     # onecfg upgrade
     ANY   : Backup stored in '/tmp/onescape/backups/2020-6...
     ANY   : Configuration updated to 6.2.0

If you get conflicts when running onecfg upgrade refer to the :ref:`onecfg upgrade basic usage documentation <cfg_usage>` on how to upgrade and troubleshoot the configurations, in particular the :ref:`onecfg upgrade doc <cfg_upgrade>` and the :ref:`troubleshooting section <cfg_conflicts>`.

FireEdge public endpoint is not working
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After updating the configuration file `sunstone-server.conf`, if you didn't install FireEdge :ref:`FireEdge <fireedge_setup>` you might get an error like this `FireEdge public endpoint is not working, please contact your cloud administrator` in the Web GUI. By default this configuration file will have the following configuration enabled.

.. code::

    $ tail -n 5 /etc/one/sunstone-server.conf
    # FireEdge
    ################################################################################

    :private_fireedge_endpoint: http://localhost:2616
    :public_fireedge_endpoint: http://localhost:2616

If you don't want to use the new feature, comment these out in order to get rid of the error.

Step 8. Upgrade the Database Version
================================================================================

.. important::

    Users of the Community Edition of OpenNebula can upgrade from the previous stable version if they are running a non-commercial OpenNebula cloud. In order to access the migrator package a request needs to be made through this `online form <https://opennebula.io/get-migration>`__.

Make sure at this point that OpenNebula is not running. If you installed from packages, the service may have been started automatically. Simply run the ``onedb upgrade -v`` command. The connection parameters are automatically retrieved from ``oned.conf``.

Step 9. Check DB Consistency
================================================================================

First, move the |version| backup file created by the upgrade command to a safe place. If you face any issues, the ``onedb`` command can restore this backup, but it won't downgrade databases to previous versions. Then, execute the ``onedb fsck`` command:

.. code::

    $ onedb fsck
    MySQL dump stored in /var/lib/one/mysql_localhost_opennebula.sql
    Use 'onedb restore' or restore the DB using the mysql command:
    mysql -u user -h server -P port db_name < backup_file

    Total errors found: 0


Step 10. Start OpenNebula
================================================================================

Now you should be able to start OpenNebula as usual by running as ``root``:

.. prompt:: text # auto

    # systemctl start opennebula

At this point OpenNebula will continue the monitoring and management of your previous Hosts and VMs.  As a measure of caution, look for any error messages in ``oned.log``, and check that all drivers are loaded successfully. You may also try some  **show** subcommand for some resources to check everything is working (e.g. ``onehost show``, or ``onevm show``).

Step 11. Restore Custom Probes
================================================================================

If you have any custom monitoring probes, follow :ref:`these instructions <devel-im>`, to update them to the new monitoring system


Step 12. Update the Hypervisors
================================================================================

.. warning:: If you're using vCenter please skip to the next step.

First we need to let OpenNebula know that the hypervisors can be monitored again, but we still don't want to deploy any VMs on them until they are fully updated. Instead, we will switch them to ``DISABLE`` state:

.. prompt:: text $ auto

   $ onehost disable <host_id>

After that, update the virtualization, storage and networking drivers.  As the ``oneadmin`` user, execute:

.. prompt:: text $ auto

   $ onehost sync

Then log in to your hypervisor Hosts and update the ``opennebula-node`` packages:

Ubuntu/Debian

.. prompt:: text # auto

    # apt-get install --only-upgrade opennebula-node-<hypervisor>

RHEL

.. prompt:: text # auto

    # yum upgrade opennebula-node-<hypervisor>

.. note:: Note that the ``<hypervisor>`` tag should be replaced by the name of the corresponding hypervisor (i.e ``kvm``, ``lxc`` or ``firecracker``).

.. important::  For KVM hypervisor it's necessary to restart also the libvirt service


Step 13. Enable Hosts
================================================================================

If you are upgrading from version 6.2+. Use ``onezone enable <zone_id>`` to make OpenNebula fully functional.

Enable all Hosts, disabled in step 2.

.. important:: Due to the new monitoring system, the firewall of the Front-end (if enabled) must allow TCP and UDP packages incoming from the Hosts on port 4124.

After following all the steps, please review the corresponding guide:

.. toctree::
   :maxdepth: 1

   Additional Steps for 5.8.x <upgrade_58>
   Additional Steps for 5.6.x <upgrade_56>
