Example Guide On Setting Up Partitioning
========================================

This HowTo guide will show you some examples of how to set up both simple, single level partitioning as well as multi-level sub-partitioning. It will also show you how to partition data out of a table that has existing data (see **Sub-partition ID->ID->ID**) and undo the partitioning of an existing partition set. For more details on what each function does and the additional features in this extension, please see the **pg_partman.md** documentation file.

### Simple Time Based Static - 1 Partition Per Day

    keith=# \d partman_test.time_static_table
               Table "partman_test.time_static_table"
     Column |           Type           |       Modifiers        
    --------+--------------------------+------------------------
     col1   | integer                  | not null
     col2   | text                     | 
     col3   | timestamp with time zone | not null default now()
    Indexes:
        "time_static_table_pkey" PRIMARY KEY, btree (col1)


    keith=# SELECT create_parent('partman_test.time_static_table', 'col3', 'time-static', 'daily');
     create_parent 
    ---------------
     t
    (1 row)


    keith=# \d+ partman_test.time_static_table
                                   Table "partman_test.time_static_table"
     Column |           Type           |       Modifiers        | Storage  | Stats target | Description 
    --------+--------------------------+------------------------+----------+--------------+-------------
     col1   | integer                  | not null               | plain    |              | 
     col2   | text                     |                        | extended |              | 
     col3   | timestamp with time zone | not null default now() | plain    |              | 
    Indexes:
        "time_static_table_pkey" PRIMARY KEY, btree (col1)
    Triggers:
        time_static_table_part_trig BEFORE INSERT ON partman_test.time_static_table FOR EACH ROW EXECUTE PROCEDURE partman_test.time_static_table_part_trig_func()
    Child tables: partman_test.time_static_table_p2015_01_08,
                  partman_test.time_static_table_p2015_01_09,
                  partman_test.time_static_table_p2015_01_10,
                  partman_test.time_static_table_p2015_01_11,
                  partman_test.time_static_table_p2015_01_12,
                  partman_test.time_static_table_p2015_01_13,
                  partman_test.time_static_table_p2015_01_14,
                  partman_test.time_static_table_p2015_01_15,
                  partman_test.time_static_table_p2015_01_16

Trigger function only covers a specific period of time for 4 days before and 4 days after today:

    keith=# \sf partman_test.time_static_table_part_trig_func 
    CREATE OR REPLACE FUNCTION partman_test.time_static_table_part_trig_func()
     RETURNS trigger
     LANGUAGE plpgsql
    AS $function$ 
    BEGIN 
    IF TG_OP = 'INSERT' THEN 
        IF NEW.col3 >= '2015-01-12 00:00:00-05' AND NEW.col3 < '2015-01-13 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_12 VALUES (NEW.*); 
        ELSIF NEW.col3 >= '2015-01-11 00:00:00-05' AND NEW.col3 < '2015-01-12 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_11 VALUES (NEW.*); 
        ELSIF NEW.col3 >= '2015-01-13 00:00:00-05' AND NEW.col3 < '2015-01-14 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_13 VALUES (NEW.*);
        ELSIF NEW.col3 >= '2015-01-10 00:00:00-05' AND NEW.col3 < '2015-01-11 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_10 VALUES (NEW.*); 
        ELSIF NEW.col3 >= '2015-01-14 00:00:00-05' AND NEW.col3 < '2015-01-15 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_14 VALUES (NEW.*);
        ELSIF NEW.col3 >= '2015-01-09 00:00:00-05' AND NEW.col3 < '2015-01-10 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_09 VALUES (NEW.*); 
        ELSIF NEW.col3 >= '2015-01-15 00:00:00-05' AND NEW.col3 < '2015-01-16 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_15 VALUES (NEW.*);
        ELSIF NEW.col3 >= '2015-01-08 00:00:00-05' AND NEW.col3 < '2015-01-09 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_08 VALUES (NEW.*); 
        ELSIF NEW.col3 >= '2015-01-16 00:00:00-05' AND NEW.col3 < '2015-01-17 00:00:00-05' THEN 
            INSERT INTO partman_test.time_static_table_p2015_01_16 VALUES (NEW.*); 
        ELSE 
            RETURN NEW; 
        END IF; 
    END IF; 
    RETURN NULL; 
    END $function$


### Simple Serial ID Static - 1 Partition Per 10 ID Values Starting With Empty Table

    keith=# \d partman_test.id_static_table
           Table "partman_test.id_static_table"
     Column |           Type           |   Modifiers   
    --------+--------------------------+---------------
     col1   | integer                  | not null
     col2   | text                     | 
     col3   | timestamp with time zone | default now()
    Indexes:
        "id_static_table_pkey" PRIMARY KEY, btree (col1)


    keith=# SELECT create_parent('partman_test.id_static_table', 'col1', 'id-static', '10');
     create_parent 
    ---------------
     t
    (1 row)


    keith=# \d+ partman_test.id_static_table
                               Table "partman_test.id_static_table"
     Column |           Type           |   Modifiers   | Storage  | Stats target | Description 
    --------+--------------------------+---------------+----------+--------------+-------------
     col1   | integer                  | not null      | plain    |              | 
     col2   | text                     |               | extended |              | 
     col3   | timestamp with time zone | default now() | plain    |              | 
    Indexes:
        "id_static_table_pkey" PRIMARY KEY, btree (col1)
    Triggers:
        id_static_table_part_trig BEFORE INSERT ON partman_test.id_static_table FOR EACH ROW EXECUTE PROCEDURE partman_test.id_static_table_part_trig_func()
    Child tables: partman_test.id_static_table_p0,
                  partman_test.id_static_table_p10,
                  partman_test.id_static_table_p20,
                  partman_test.id_static_table_p30,
                  partman_test.id_static_table_p40

Trigger function only covers for 4*10 intervals above the current max (0). Once max id reaches higher values, it will aslo cover up to 4*10 intervals behind the current max.
Trigger also takes care of making new partitions automatically when current max reaches 50% of the current child table's maximum.

    CREATE OR REPLACE FUNCTION partman_test.id_static_table_part_trig_func()
     RETURNS trigger
     LANGUAGE plpgsql
    AS $function$ 
    DECLARE
        v_current_partition_id  bigint;
        v_last_partition        text := 'partman_test.id_static_table_p40';
        v_id_position           int;
        v_next_partition_id     bigint;
        v_next_partition_name   text;         
        v_partition_created     boolean;
    BEGIN
    IF TG_OP = 'INSERT' THEN 
        IF NEW.col1 >= 0 AND NEW.col1 < 10 THEN  
            INSERT INTO partman_test.id_static_table_p0 VALUES (NEW.*); 
        ELSIF NEW.col1 >= 10 AND NEW.col1 < 20 THEN 
            INSERT INTO partman_test.id_static_table_p10 VALUES (NEW.*);
        ELSIF NEW.col1 >= 20 AND NEW.col1 < 30 THEN 
            INSERT INTO partman_test.id_static_table_p20 VALUES (NEW.*);
        ELSIF NEW.col1 >= 30 AND NEW.col1 < 40 THEN 
            INSERT INTO partman_test.id_static_table_p30 VALUES (NEW.*);
        ELSIF NEW.col1 >= 40 AND NEW.col1 < 50 THEN 
            INSERT INTO partman_test.id_static_table_p40 VALUES (NEW.*);
        ELSE
            RETURN NEW;
        END IF;
        v_current_partition_id := NEW.col1 - (NEW.col1 % 10);
        IF (NEW.col1 % 10) > (10 / 2) THEN
            v_id_position := (length(v_last_partition) - position('p_' in reverse(v_last_partition))) + 2;
            v_next_partition_id := (substring(v_last_partition from v_id_position)::bigint) + 10;
            WHILE ((v_next_partition_id - v_current_partition_id) / 10) <= 4 LOOP 
                v_partition_created := partman.create_partition_id('partman_test.id_static_table', ARRAY[v_next_partition_id]);
                IF v_partition_created THEN
                    PERFORM partman.create_function_id('partman_test.id_static_table');
                    PERFORM partman.apply_constraints('partman_test.id_static_table');
                END IF;
                v_next_partition_id := v_next_partition_id + 10;
            END LOOP;
        END IF;
    END IF; 
    RETURN NULL; 
    END $function$


### Sub-partition Time->Time->Time - Yearly -> Monthly -> Daily

    keith=# \d partman_test.time_static_table
               Table "partman_test.time_static_table"
     Column |           Type           |       Modifiers        
    --------+--------------------------+------------------------
     col1   | integer                  | not null
     col2   | text                     | 
     col3   | timestamp with time zone | not null default now()
    Indexes:
        "time_static_table_pkey" PRIMARY KEY, btree (col1)

Create top yearly partition set that only covers 2 years forward/back

    keith=# SELECT create_parent('partman_test.time_static_table', 'col3', 'time-static', 'yearly', p_premake := 2);
     create_parent 
    ---------------
     t
    (1 row)

Now tell pg_partman to partition all yearly child tables by month. Do this by giving it the parent table of the yearly partition set (happens to be the same as above)

    keith=# SELECT create_sub_parent('partman_test.time_static_table', 'col3', 'time-static', 'monthly', p_premake := 2);
     create_sub_parent 
    -------------------
     t
    (1 row)

    keith=# select tablename from pg_tables where schemaname = 'partman_test' order by tablename;
                tablename             
    ----------------------------------
     time_static_table
     time_static_table_p2013
     time_static_table_p2013_p2013_01
     time_static_table_p2014
     time_static_table_p2014_p2014_11
     time_static_table_p2014_p2014_12
     time_static_table_p2015
     time_static_table_p2015_p2015_01
     time_static_table_p2015_p2015_02
     time_static_table_p2015_p2015_03
     time_static_table_p2016
     time_static_table_p2016_p2016_01
     time_static_table_p2017
     time_static_table_p2017_p2017_01

The day this tutorial was written is 2015-01-12. You now see that the years are covered 2 ahead and 2 behind. And for the monthly partitions, they have been created to cover 2 months ahead and 2 months behind. A parent table ALWAYS has at least one child, so for the time period that is outside of what the trigger covers, just a single table has been made for the lowest possible month in that yearly time period (January). Now tell pg_partman to partition every monthly table that currently exists by day. Do this by giving it the parent table of each monthly partition set (the parent with the just the year suffix since it's children are the monthly partitions).

    SELECT create_sub_parent('partman_test.time_static_table_p2013', 'col3', 'time-static', 'daily', p_premake := 2);
    SELECT create_sub_parent('partman_test.time_static_table_p2014', 'col3', 'time-static', 'daily', p_premake := 2);
    SELECT create_sub_parent('partman_test.time_static_table_p2015', 'col3', 'time-static', 'daily', p_premake := 2);
    SELECT create_sub_parent('partman_test.time_static_table_p2016', 'col3', 'time-static', 'daily', p_premake := 2);
    SELECT create_sub_parent('partman_test.time_static_table_p2017', 'col3', 'time-static', 'daily', p_premake := 2);

    keith=# select tablename from pg_tables where schemaname = 'partman_test' order by tablename;
                      tablename                   
    ----------------------------------------------
     time_static_table
     time_static_table_p2013
     time_static_table_p2013_p2013_01
     time_static_table_p2013_p2013_01_p2013_01_01
     time_static_table_p2014
     time_static_table_p2014_p2014_11
     time_static_table_p2014_p2014_11_p2014_11_01
     time_static_table_p2014_p2014_12
     time_static_table_p2014_p2014_12_p2014_12_01
     time_static_table_p2015
     time_static_table_p2015_p2015_01
     time_static_table_p2015_p2015_01_p2015_01_10
     time_static_table_p2015_p2015_01_p2015_01_11
     time_static_table_p2015_p2015_01_p2015_01_12
     time_static_table_p2015_p2015_01_p2015_01_13
     time_static_table_p2015_p2015_01_p2015_01_14
     time_static_table_p2015_p2015_02
     time_static_table_p2015_p2015_02_p2015_02_01
     time_static_table_p2015_p2015_03
     time_static_table_p2015_p2015_03_p2015_03_01
     time_static_table_p2016
     time_static_table_p2016_p2016_01
     time_static_table_p2016_p2016_01_p2016_01_01
     time_static_table_p2017
     time_static_table_p2017_p2017_01
     time_static_table_p2017_p2017_01_p2017_01_01

Again, assuming today's date is 2015-01-12, it has created the subpartitions to cover 2 days in the past and 2 days in the future. All other parent tables outside of the current time period have the lowest possible day created for them.


### Sub-partition ID->ID->ID - 10,000 -> 1,000 -> 100


### Set run_maintenance() to run often enough

Using the above time-based partitions, run_maintenance() should be called at least twice a day to ensure it keeps up with the requirements of the smallest time partition interval (daily). Example cron entry to run at 1:03am and 1:03pm:

    03 01,13 * * * psql -c "SELECT run_maintenance()"

For serial based partitioning that uses run_maintenance() (the sub-partitioning above does so), you must know your data ingestion rate and call it often enough to keep new partitions created ahead of that rate.

### Use Retention Policy
To drop partitions on the first example above that are older than 30 days, set the following:

    UPDATE part_config SET retention = '30 days', retention_keep_table = false WHERE parent_table = 'partman_test.time_static_table';

To drop partitions on the second example above that contain a value 100 less than the current max (max(col1) - 100), set the following:

    UPDATE part_config SET retention = '100', retention_keep_table = false WHERE parent_table = 'partman_test.id_static_table';

For example, once the current id value of col1 reaches 1000, all partitions with values less than 900 will be dropped.

If you'd like to keep the old data stored offline in dump files, set the retention_schema column as well:

    UPDATE part_config SET retention = '30 days', retention_schema = 'archive' WHERE parent_table = 'partman_test.time_static_table';

Then use the included python script **dump_partition.py** to dump out all tables contained in the archive schema:

    $ python dump_partition.py -c "host=localhost username=postgres" -d mydatabase -n archive -o /path/to/dump/location 


### Undo Partitioning - Simple Time Based Static


### Undo Partitioning - Simple Serial ID Static


### Undo Partitioning - Sub-partition Time->Time->Time


### Undo Partitioning - Sub-partition ID->ID->ID


