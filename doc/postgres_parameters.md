## Setting PostgreSQL server parameters

The stolon keeper manages the starting `postgresql.conf` populating it with all its controlled postgres parameters needed for managing the replication.

The keepers's controlled postgres parameters are:
```
unix_socket_directories
wal_level
wal_keep_segments
hot_standby
```

All the other parameters can be provided by the users in different ways: using an externally managed configuration and/or using a centralized configuration define in the [cluster_config](cluster_config.md).

The externally managed configuration is useful when the user wants to manually control or use external tools to handle the postgres server parameters and is more suitable for "static" infrastructures.

The centralized configuration is suited when the user doesn't want to set for every postgres instance its  parameters but it prefers to keep them the same for all the instances and handle them in an centralized configuration. This fits well with "dynamic infrastructures" like cloud native environments (for example on `kubernetes` when you can easily scale the number of keepers, since manually setting the configuration, connecting to every keeper's pod and triggering instance reload/restart becomes complex and probably it's an anti-pattern).

### Externally managed configuration

By default the `postgresql.conf` generated by the stolon keeper also includes a configuration directory under it's `$dataDir/postgres/conf.d`. The user can specify another configuration directory providing the `--pg-conf-dir` option to the stolon keeper.

The user can place its custom config files inside this directory.

If the user wants to change the configuration files inside this directory with an active postgresql instance, it should also handle the instance reload or restart. Triggering a reload/restart can clash with concurrent keeper operations so in future a stolonctl function to trigger instances reload/restart will also be added.

It's up to the user to keep the configuration synced (if wanted) between all the postgres instances.

**Note**: when adding new keepers, if the configuration directory is inside the postgresql data dir it'll be automatically copied from the source sender (since the stolon keeper uses `pg_basebackup`). If it's outside the postgres data dir then it should be manually populated.

### Centralized configuration

The user can provide the required server parameters inside the [cluster config](cluster_config.md). The keeper will read the new config, generate the new configuration file and reload the instance. If some parameters needs an instance restart to be applied this should be manually done by the user.

A stolonctl function to trigger instances reload/restart will also be added.

## Order of Precedence

keepers managed parameters will always override user defined parameters and centralized configuration parameters will override the ones provided by external configuration.

1. keepers managed parameters
2. centralized configuration parameters
3. externally managed configuration

## Configuration checks

Actually stolon doesn't do any check on the provided configurations, so, if the provided parameters are wrong this won't create problems at instance reload (just some warning in the postgresql logs) but at the next instance restart, it'll probably fail making the instance not available (thus triggering failover if it's the master or other changes in the clusterview).