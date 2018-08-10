Icinga Client Setup
==================

These are general instructions for setting up an Icinga Satellite/Client system. Please refer to the Icinga documentation on distributed setups for full details - https://www.icinga.com/docs/icinga2/latest/doc/06-distributed-monitoring/

Setup the EPEL repository

https://packages.icinga.com/epel/

sudo yum install epel-release


If Postgres related packages are needed and the client is subscribed to our portal, see the instructions a http://access.crunchydata.com
 
Otherwise set up the relevant pgdg repository - https://www.postgresql.org/download/

* Install postgres client if necessary
    yum install postgresql95
* If client is using check_postgres, install via package. Then also create symlink to current, known plugins directory for icinga plugins
```
[root@pacs-bastion plugins]# pwd
/usr/lib64/nagios/plugins
[root@pacs-bastion plugins]# ln -s /usr/bin/check_postgres.pl 
```      

Follow icinga repository setup instructions for relevant OS found here -  https://www.icinga.com/docs/icinga2/latest/doc/02-getting-started/#installing-icinga-2

When installing on Amazon Linux, the rpm repository install will not work. Manually create the following file:
```
[crunchy@pacs-bastion yum.repos.d]$ pwd
/etc/yum.repos.d

[crunchy@pacs-bastion yum.repos.d]$ cat ICINGA-release.repo 
[icinga-stable-release]
name=ICINGA (stable release for epel)
baseurl=http://packages.icinga.com/epel/6/release/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ICINGA
```

No matter what OS icinga is installed on, ensure you go into the repo file and set "enabled=0" so the client doesn't accidentally upgrade the icinga server without explicitly enabling the repo to do so

```
sudo yum --enablerepo=icinga-stable-release install icinga2

sudo yum install nagios-plugins-all
```

Generate ticket on master. Get the final common name (CN) from the ZoneName variable in the constants.conf file located in the git repository for this system
```
icinga2 pki ticket --cn client.common.name.com
```
Run icinga node wizard on client system: 
```
icinga2 node wizard
```
* Ensure the common name for the client entered matches the one used when generating the ticket above
* Use the following for the master CN : monitoring.crunchydata.com
* Choose yes to establish connection back to the master and use the following hostname: data.crunchydata.com
* After entering request ticket value, ensure the local zone name and parent zone names again match the values given above 
* Say yes to accepting config/commands from parent
* Say yes to disable the inclusion of conf.d


Master key info as of August 8, 2018
```
Parent certificate information:

 Subject:     CN = monitoring.crunchydata.com
 Issuer:      CN = Icinga CA
 Valid From:  Dec  8 23:21:07 2015 GMT
 Valid Until: Dec  4 23:21:07 2030 GMT
 Fingerprint: CA 6E 45 9B 4B 14 FC 9F 91 6A 0D 82 14 0B 6D 52 23 D6 8B A9 
```
Ensure the following features are enabled/disabled on the client system. Wizard should have taken care of this:
```
$ sudo icinga2 feature list
Disabled features: command compatlog debuglog elasticsearch gelf graphite influxdb livestatus notification opentsdb perfdata statusdata syslog
Enabled features: api checker mainlog
```
Grab icinga2.conf, constants.conf, zones.conf & features-available/api.conf from git repository and put in place. Restart icinga2

Reload icinga2 on the master icinga system. This should push the configuration files out to the client system and things should start showing up in icingaweb

