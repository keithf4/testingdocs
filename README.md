PGExtractor is both a command line script and python class that can be used to provide more finely 
detailed filtering options for PostgreSQL's pg_dump program. Each database object is extracted into 
its own file organized into folders by type. This makes having the database schema as a reference 
easier and allows for better checking into version control. By default data is not dumped, but 
is easily done with a single option and can be done as plaintext or pg_dump's custom format. Object 
filtering can be done directly via command line options or fed in with external text files. Regex 
pattern matching is also possible.

See --help & --examples for a full list of available options and how to use the script.

The script only uses pg_dump/all to touch the database. 
pg_restore is only used for generating ddl and does not ever touch the database.

Several of the class methods are public and can be used to inspect a custom format binary dump 
file or apply some of the editing options.

````
Python 3.3.1 (default, Sep 25 2013, 19:29:01) 
[GCC 4.7.3] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pg_extractor import PGExtractor
>>> p = PGExtractor()
>>> object_list = p.build_main_object_list("dumpfile.pgr")
>>> table_object_list = p.build_type_object_list(object_list, ['TABLE'])
>>> for t in table_object_list:
...     print(t)
... 
{'objname': 'job_detail_p0', 'objid': '238; 1259 596233', 'objowner': 'keith', 'objtype': 'TABLE', 'objschema': 'jobmon'}
{'objname': 'job_detail_p10', 'objid': '239; 1259 596244', 'objowner': 'keith', 'objtype': 'TABLE', 'objschema': 'jobmon'}
...
````

Remove the password hashes from an existing "pg_dumpall -r" roles file:
````
>>> p.remove_passwords("pg_dumpall_roles.sql")
````

### New Version 2.x

Version 2.x is a complete rewrite of PG Extractor in python. Most of the configuration options are the same,
but many have been changed for clarity, so please check the --help. 

Non-compatibilities with 1.x to be aware of when dropping in 2.x to replace it
 * The "hostname" is no longer a default part of the directory structure created. If this is still desired, set the --hostnamedir option with whatever the existing directory is.
 * Built in version control options are gone. If these are really wanted please create an Issue on github and I'll look into it. They were rather fragile options and could easily lead to a whole lot of things getting checked into version control that should not have been and it would take some significant work to really get them working properly. I've found it's easier (and safer) to manage version control check-ins separately.
 * Removed --rolesdir option

New features:
 * Full python Class object with public functions that can be used on a prexisting dump file
 * --jobs option to allow parallel object extraction
 * --remove_passwords option can remove the password hashes from an extracted roles file
 * --getdefaultprivs extracts the default privileges set for any roles that used ALTER DEFAULT PRIVILEGES
 * --delete cleans up empty folders properly
 * --wait option to allow a pause in object extraction. Helps reduce load when data is included in extraction.
 * --temp option to allow setting custom temporary working space
 * Sequences files can now include the statement to set the current value if data is output
 *  Better support for when objects have mixed case names or special characters. Special characters in an object name turn into ***,hexcode,*** for the filename so they can be valid and the hex code is better identifiable where it starts and ends
 *Rules & Triggers on views are now always included in the view file itself properly.

*The version 1.x series written in perl will no longer be developed. Only bug fixes to the existing code will be accepted.*

