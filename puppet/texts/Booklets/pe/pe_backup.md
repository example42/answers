## Puppet Enterprise backup

Backups allow you to more easily migrate to a new master, troubleshoot, and quickly recover in the case of system failures.

Besides the backup commands described here it's recommended to make a snapshot of the Puppet servers Virtual Machines (in case they don't run directly on physical hardware) before making an upgrade and, generally, in a continuous way, in order to have a fast recovery option in case of need.

Strictly speaking anyway, in order to restore from scratch a Puppet Enterprise installation you need the installer and the recovery of the files described below.

You can use the **puppet-backup** command to back up and restore your primary master or master of masters in monolithic installations only.

You can't use this command to back up split installations or compile masters.

By default, the backup command creates a backup of:

  - Your PE configuration, including license, classification, and RBAC settings (However, configuration backup does not include Puppet gems or Puppet Server gems).
  - PE CA certificates and the full SSL directory
  - The Puppet code deployed to your code directory at backup time.
  - PuppetDB data, including facts, catalogs and historical reports.

This is basically the content of the following directories:

  - **/etc/puppetlabs** (configurations, ssl ca, deployed puppet code)
  - **/opt/puppetlabs/server/data/postgresql/XXX/data** (PostgreSQL data)

Each time you create a new backup, PE creates a single, timestamped backup file, named in the format pe_backup-$TIMESTAMP.tgz.

This file includes everything you're backing up.

By default, PE writes backup files to **/var/puppetlabs/backups**, but you can change this location when you run the backup command.

When you restore, specify the backup file you want to restore from.

