[![PGXN version](https://badge.fury.io/pg/pg_partman.svg)](https://badge.fury.io/pg/pg_partman)

PG Partition Manager
====================

pg_partman is an extension to create and manage both time-based and number-based table partition sets. As of version 5.0.0, only native, declarative partition is supported and the older trigger-based methods have been dropped.

Child table creation is all managed by the extension itself. For an existing table that needs partitioning, the creation of a new partitioned parent must be done first and the data migrated over after setup is complete.

Optional retention policy can automatically drop partitions no longer needed for both native and non-native partitioning.

A background worker (BGW) process is included to automatically run partition maintenance without the need of an external scheduler (cron, etc) in most cases.

Bug reports & feature requests can be directed to the Issues section on Github - https://github.com/pgpartman/pg_partman/issues

For questions, comments, or if you're just not sure where to post, please use the Discussions section on Github. Feel free to post here no matter how minor you may feel your issue or question may be - https://github.com/pgpartman/pg_partman/discussions

If you're still trying to evaluate whether partitioning is a good choice for your environment, keep an eye on the HypoPG project. Version 2 will have a hypothetical partitioning feature that will let you evaluate different partitioning schemes without requiring you to actually partition your data. I may see about integrating this feature into pg_partman once it is available. - https://hypopg.readthedocs.io

INSTALLATION
------------
Requirement: 

 * PostgreSQL >= 14

Recommended: 

 * pg_jobmon (>=v1.4.1). PG Job Monitor will automatically be used if it is installed and setup properly.
https://github.com/omniti-labs/pg_jobmon

### From Source
In the directory where you downloaded pg_partman, run

    make install

If you do not want the background worker compiled and just want the plain PL/PGSQL functions, you can run this instead:

    make NO_BGW=1 install

### Package
I do not personally maintain any OS packages for pg_partman, but several repository maintainers from the PostgreSQL Development Group (PGDG) have kindly been maintaining packages for the community. Please check the [PostgreSQL Downloads](https://www.postgresql.org/download/) page to see if your OS has a package available

### Setup
The background worker must be loaded on database start by adding the library to shared_preload_libraries in postgresql.conf

    shared_preload_libraries = 'pg_partman_bgw'     # (change requires restart)

You can also set other control variables for the BGW in postgresql.conf. "dbname" is required at a minimum for maintenance to run on the given database(s). These can be added/changed at anytime with a simple reload. See the documentation for more details. An example with some of them:

    pg_partman_bgw.interval = 3600
    pg_partman_bgw.role = 'keith'
    pg_partman_bgw.dbname = 'keith'

Log into PostgreSQL and run the following commands. Schema is optional (but recommended) and can be whatever you wish, but it cannot be changed after installation. If you're using the BGW, the database cluster can be safely started without having the extension first created in the configured database(s). You can create the extension at any time and the BGW will automatically pick up that it exists without restarting the cluster (as long as shared_preload_libraries was set) and begin running maintenance as configured.

    CREATE SCHEMA partman;
    CREATE EXTENSION pg_partman SCHEMA partman;

pg_partman does not require a superuser to run, but currently still requires it to be installed. If not using a superuser, it is recommended that a dedicated role is created for running pg_partman functions and to be the owner of all partition sets that pg_partman maintains. At a minimum this role will need the following privileges (assuming pg_partman is installed to the "partman" schema and that dedicated role is called "partman"):

    CREATE ROLE partman WITH LOGIN;
    GRANT ALL ON SCHEMA partman TO partman;
    GRANT ALL ON ALL TABLES IN SCHEMA partman TO partman;
    GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA partman TO partman;
    GRANT EXECUTE ON ALL PROCEDURES IN SCHEMA partman TO partman;  -- PG11+ only
    GRANT ALL ON SCHEMA my_partition_schema TO partman;
    GRANT TEMPORARY ON DATABASE mydb to partman; -- allow creation of temp tables to move data out of default 

If you need the role to also be able to create schemas, you will need to grant create on the database as well. In general this shouldn't be required as long as you give the above role CREATE privileges on any pre-existing schemas that will contain partition sets.

    GRANT CREATE ON DATABASE mydb TO partman;

I've received many requests for being able to install this extension on Amazon RDS. As of PostgreSQL 12.5, RDS has made the pg_partman extension available. Many thanks to the RDS team for including this extension in their environment!

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/PostgreSQL_Partitions.html


UPGRADE
-------

Run "make install" same as above or update your respective packages to put the new script files and libraries in place. Then run the following in PostgreSQL itself:

    ALTER EXTENSION pg_partman UPDATE TO '<latest version>';

If you are doing a pg_dump/restore and you've upgraded pg_partman in place from previous versions, it is recommended you use the --column-inserts option when dumping and/or restoring pg_partman's configuration tables. This is due to ordering of the configuration columns possibly being different (upgrades just add the columns onto the end, whereas the default of a new install may be different).

If upgrading between any major versions of pg_partman (4.x -> 5.x, etc), please carefully read all intervening version notes in the CHANGELOG, especially those notes for the major version. There are often additional instructions and other important considerations for the updates.

IMPORTANT NOTE: Some updates to pg_partman must drop and recreate its own database objects. If you are revoking PUBLIC privileges from functions/procedures, that can be added back to objects that are recreated as part of an update. If restrictions from PUBLIC use are desired for pg_partman, it is recommended to install it into its own schema as shown above and the revoke undesired access to that schema. Otherwise you may have to add an additional step to your extension upgrade procedures to revoke PUBLIC access again.

Special considerations are needed if you are upgrading to 5+ from any version less than 5.0.0. Please see [pg_partman_5.0.0_upgrade.md](doc/pg_partman_5.0.0_upgrade.md).

EXAMPLES
--------
For setting up native partitioning with pg_partman on a brand new table, or to migrate an existing normal table to native partitioning, see [pg_partman_howto.md](doc/pg_partman_howto.md).

For migrating a trigger-based partitioned table to native partitioning using pg_partman, see [migrate_to_native.md](doc/migrate_to_native.md). Note that if you plan to migrate to pg_partman, you will first have to migrate to a natively partitioned table before it can be managed by pg_partman.

Other documents are also available in the documents folder.

See the [pg_partman.md reference file](doc/pg_partman.md) in the doc folder for full details on all commands and options for pg_partman.


TESTING
-------
This extension can use the pgTAP unit testing suite to evalutate if it is working properly - [http://www.pgtap.org](http://www.pgtap.org).
WARNING: You MUST increase max_locks_per_transaction above the default value of 64. For me, 128 has worked well so far. This is due to the sub-partitioning tests that create/destroy several hundred tables in a single transaction. If you don't do this, you risk a cluster crash when running subpartitioning tests.

