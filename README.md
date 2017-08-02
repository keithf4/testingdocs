# Tempered

PostgreSQL Metric Gathering and Visualization.

## Bloat check setup

 - Install pgstattuple contrib module into all databases that need bloat monitoring

    CREATE EXTENSION pgstattuple;

 - Download pg_bloat_check.py - https://github.com/keithf4/pg_bloat_check 
 - Run script to create tables to store bloat statistics
 
    $ ./pg_bloat_check.py -c "host=localhost dbname=mydb user=myrole" --create_stats_table

 - Grant privileges to postgres_exporter role

    GRANT SELECT ON bloat_stats TO postgres_exporter;
    GRANT SELECT ON bloat_indexes TO postgres_exporter;
    GRANT SELECT ON bloat_tables TO postgres_exporter;

 - Run script to populate data so that prometheus can query it.

    $ ./pg_bloat_check.py -c "host=localhost dbname=mydb user=myrole"
