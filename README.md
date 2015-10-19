[![PGXN version](https://badge.fury.io/pg/pg_partman.svg)](https://badge.fury.io/pg/pg_partman)

PG Partition Manager
====================

pg_partman is an extension to create and manage both time-based and serial-based table partition sets. Sub-partitoning is also supported. Child table & trigger function creation is all managed by the extension itself. Tables with existing data can also have their data partitioned in easily managed smaller batches. Optional retention policy can automatically drop partitions no longer needed.
A background worker (BGW) process is included to automatically run partition maintenance without the need of an external scheduler (cron, etc) in most cases.

All bug reports, feature requests and general questions can be directed to the Issues section on Github. Please feel free to post here no matter how minor you may feel your issue or question may be. - https://github.com/keithf4/pg_partman/issues

INSTALLATION
------------
Requirement: PostgreSQL 9.4 or greater

Recommended: pg_jobmon (>=v1.3.0). PG Job Monitor will automatically be used if it is installed and setup properly.
https://github.com/omniti-labs/pg_jobmon

In the directory where you downloaded pg_partman, run

    make install

If you do not want the background worker compiled and just want the plain PL/PGSQL functions, you can run this instead:

    make NO_BGW=1 install

The background worker must be loaded on database start by adding the library to shared_preload_libraries in postgresql.conf

    shared_preload_libraries = 'pg_partman_bgw'     # (change requires restart)

You can also set other control variables for the BGW in postgresql.conf. "dbname" is required at a minimum for maintenance to run on the given database(s). These can be added/changed at anytime with a simple reload. See the documentation for more details. An example with some of them:

    pg_partman_bgw.interval = 3600
    pg_partman_bgw.role = 'keith'
    pg_partman_bgw.dbname = 'keith'

Log into PostgreSQL and run the following commands. Schema is optional (but recommended) and can be whatever you wish, but it cannot be changed after installation. If you're using the BGW, the database cluster can be safely started without having the extension first created in the configured database(s). You can create the extension at any time and the BGW will automatically pick up that it exists without restarting the cluster (as long as shared_preload_libraries was set) and begin running maintenance as configured.

    CREATE SCHEMA partman;
    CREATE EXTENSION pg_partman SCHEMA partman;

Functions must either be run as a superuser or you can set the ownership of the extension functions to a superuser role and they will also work (SECURITY DEFINER is set).

The 1.8.x branch is still available on github if you have PostgreSQL versions 9.1 - 9.3. You will have to install the 1.8.7 tagged release located here: https://github.com/keithf4/pg_partman/releases/tag/v1.8.7 then check the "updates" folder in the latest 2.x.x release to see if there have been any updates to the 1.8.x series since then and apply them. Only bug fixes are being applied to the 1.8.x series and only while versions of PostgreSQL prior to 9.4 are still officially supported themselves. All new development is being done on 2.x.x, so it's advised that you update your PostgreSQL cluster to make managing this extension easier.

UPGRADE
-------

Run "make install" same as above to put the script files and libraries in place. Then run the following in PostgreSQL itself:

    ALTER EXTENSION pg_partman UPDATE TO '<latest version>';

If upgrading from 1.x to 2.x, please see the CHANGELOG or the notes in the update script itself for additional instructions for updating your trigger functions to the newer version and other important considerations for the update.

EXAMPLE
-------

First create a parent table with an appropriate column type for the partitioning type you will do. Apply all defaults, indexes, constraints, privileges & ownership to the parent table and they will be inherited to newly created child tables automatically (not already existing partitions, see docs for how to fix that). Here's one with columns that can be used for either

    CREATE schema test;
    CREATE TABLE test.part_test (col1 serial, col2 text, col3 timestamptz NOT NULL DEFAULT now());

Then just run the create_parent() function with the appropriate parameters

    SELECT partman.create_parent('test.part_test', 'col3', 'time', 'daily');
    or
    SELECT partman.create_parent('test.part_test', 'col1', 'id', '100000');

This will turn your table into a parent table and premake 4 future partitions and also make 4 past partitions. To make new partitions for time-based partitioning, use the run_maintenance() function. Ideally, you'd run this as a cronjob to keep new partitions premade in preparation of new data. Serial based partitioning does not always require run_maintenance() (see doc file below).

This should be enough to get you started. Please see the [pg_partman.md file](doc/pg_partman.md) in the doc folder for more information on the types of partitioning supported and what the parameters in the create_parent() function mean. 


TESTING
-------
This extension can use the pgTAP unit testing suite to evalutate if it is working properly (http://www.pgtap.org).
WARNING: You MUST increase max_locks_per_transaction above the default value of 64. For me, 128 has worked well so far. This is due to the sub-partitioning tests that create/destroy several hundred tables in a single transaction. If you don't do this, you risk a cluster crash when running subpartitioning tests.


LICENSE AND COPYRIGHT
---------------------

PG Partition Manager (pg_partman) is released under the PostgreSQL License, a liberal Open Source license, similar to the BSD or MIT licenses.

Copyright (c) 2015 OmniTI, Inc.

Permission to use, copy, modify, and distribute this software and its documentation for any purpose, without fee, and without a written agreement is hereby granted, provided that the above copyright notice and this paragraph and the following two paragraphs appear in all copies.

IN NO EVENT SHALL THE AUTHOR BE LIABLE TO ANY PARTY FOR DIRECT, INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES, INCLUDING LOST PROFITS, ARISING OUT OF THE USE OF THIS SOFTWARE AND ITS DOCUMENTATION, EVEN IF THE AUTHOR HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

THE AUTHOR SPECIFICALLY DISCLAIMS ANY WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE SOFTWARE PROVIDED HEREUNDER IS ON AN "AS IS" BASIS, AND THE AUTHOR HAS NO OBLIGATIONS TO PROVIDE MAINTENANCE, SUPPORT, UPDATES, ENHANCEMENTS, OR MODIFICATIONS.
