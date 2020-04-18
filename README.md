﻿# Cacti v1+ Docker Container
[![](https://images.microbadger.com/badges/image/smcline06/cacti.svg)](https://microbadger.com/images/smcline06/cacti "Get your own image badge on microbadger.com")

##### Github Repo: https://github.com/scline/docker-cacti
[![GitHub Open Issues](https://img.shields.io/github/issues/scline/docker-cacti.svg)](https://github.com/scline/docker-cacti/issues)[![GitHub Stars](https://img.shields.io/github/stars/scline/docker-cacti.svg)](https://github.com/scline/docker-cacti)[![GitHub Forks](https://img.shields.io/github/forks/scline/docker-cacti.svg)](https://github.com/scline/docker-cacti) 

##### Dockerhub Repo: https://hub.docker.com/r/smcline06/cacti/
[![Stars on Docker Hub](https://img.shields.io/docker/stars/smcline06/cacti.svg)](https://hub.docker.com/r/smcline06/cacti)[![Pulls on Docker Hub](https://img.shields.io/docker/pulls/smcline06/cacti.svg)](https://hub.docker.com/r/smcline06/cacti)  

## Cacti System
Cacti is a complete network graphing solution designed to harness the power of RRDTool's data storage and graphing functionality. Cacti provides following features:

* remote and local data collectors
* network discovery
* device management automation
* graph templating
* custom data acquisition methods
* user, group and domain management
* C3 level security settings for local accounts
  * strong password hashing
  * forced regular password changes, complexity, and history 
  * account lockout support

All of this is wrapped in an intuitive, easy to use interface that makes sense for both LAN-sized installations and complex networks with thousands of devices.
Developed in the early 2000's by Ian Berry as a high school project, it has been used by thousands of companies and enthusiasts to monitor and manage their Networks and Data Centers.
More information around this opensource product can be located at the following [website][cws].

## Using this image
### Running the container
This container contains Cacti v1+ and is not compatible with older version of cacti. It does rely on an external MySQL database that can be already configured before initial startup or having the container itself perform the setup and initialization. If you want this container to perform these steps for you, you will need to pass the root password for mysql login or startup will fail. This container automatically incorporates Cacti Spine's multithreaded poller.

### Exposed Ports
The following ports are important and used by Cacti

| Port |     Notes     |  
|------|:-------------:|
|  80  | HTTP GUI Port |
|  443 | HTTPS GUI Port|

It is recommended to allow at least one of the above ports for access to the monitoring system. This is translated by the -p hook. For example
`docker run -p 80:80 -p 443:443`

#### HTTPS 
By default https will work if exposed, though using a self signed certificate. To provide your own certificate replace via mounting the following files:

```
/etc/ssl/certs/cacti.key
/etc/ssl/certs/cacti.crt
```

## Installation

### Cacti Master
The main cacti poller settings, these are required for single cacti and multi cacti host installations.

| Environment Variable | Function |
|---|---|
| DB_NAME | The MySQL database name, this is used for both cacti settings and spine poller configurations. |
| DB_USER | MySQL database user cacti should use. Both cacti and spine poller will share these settings. |
| DB_PASS | MySQL database password assigned to `DB_USER` Both cacti and spine poller will share these settings. |
| DB_HOST | The IP address, FQDN/hostname, or linked container name that cacti would use as a database. |
| DB_PORT | What TCP port is the MySQL database listening on, by default its 3306. |
| DB_ROOT_PASS | This is only needed  if the `INITIALIZE_DB` is set to 1. This is required if you want the cacti container to setup remote MySQL user accounts and Databases for use. |
| INITIALIZE_DB | Can be `0` for false or `1` for true. If true the container will require `DB_ROOT_PASS` to the target database. The container will attempt to create usernames/passwords and Databases required on the remote system for Cacti to funtion. |
| TZ | TimeZone, please select a format Centos understands, a list can be generated by running `ls /usr/share/zoneinfo`. |
| BACKUP_RETENTION | Number of backup files to keep. |
| REMOTE_POLLER | Can be `0` for false (default) or `1` for true. |
| PHP_MEMORY_LIMIT | PHP memory limit adjust, by defaults its 128M |
| PHP_MAX_EXECUTION_TIME |  PHP max execution time adjust, by defaults its 30 second |
| PHP_SNMP | If set to `0`, will remove PHP-SNMP, this is sometimes required for some scripts or snmpv3 to work properly. by defualt this php-snmp is enabled |

### Remote Cacti Pollers
Remote cacti poller containers require the following, the major differance here is the inclusion of RDB (remote database) variables which should be pointed at the master cacti installation settings. 

| Environment Variable | Function |
|---|---|
| DB_NAME | The MySQL database, this is used for both cacti settings and spine poller configurations. | 
| DB_USER | MySQL database user cacti should use. Both cacti and spine poller will share these settings. | 
| DB_PASS | MySQL database password assigned to `DB_USER` Both cacti and spine poller will share these settings. | 
| DB_HOST | The IP address, FQDN/hostname, or linked container name that cacti would use as a database. | 
| DB_PORT | What TCP port is the MySQL database listening on, by default its 3306. | 
| INITIALIZE_DB | Can be `0` for false or `1` for true. If true the container will require `DB_ROOT_PASS` to the target database. The container will attempt to create usernames/passwords and Databases required on the remote system for Cacti to funtion. |
| TZ | TimeZone, please select a format Centos understands, a list can be generated by running `ls /usr/share/zoneinfo`.|
| BACKUP_RETENTION | Number of backup files to keep|
| REMOTE_POLLER | Can be `0` for false (default) or `1` for true. If true the container is setup as a remote poller. |
| RDB_NAME | The master Cacti instance MySQL database name, this is used for both cacti settings and spine poller configurations. | 
| RDB_USER | MySQL database user used by the master Cacti container should use. | 
| RDB_PASS | MySQL database password assigned to `RDB_USER` that is used by the master Cacti container. | 
| RDB_HOST | The IP address, FQDN/hostname, or linked container name that the master Cacti instance uses | 
| RDB_PORT | What TCP port is the MySQL database listening on, by default its 3306. | 

### Database Settings
The folks at Cacti.net recommend the following settings for its MySQL based database. Please understand depending on your systems resources and amount of devices your installation is monitoring these settings may need to change for optimal performance. I would recommend shooting any questions around these settings to the [Cacti community forums][cacti_forums].

|MySQL Variable| Recommended Value | Notes |
|---|---|---|
| Version | >= 5.6 | MySQL 5.6+ and MariaDB 10.0+ are great releases, and are very good versions to choose. Make sure you run the very latest release though which fixes a long standing low level networking issue that was casuing spine many issues with reliability. |
| collation_server | utf8mb4_unicode_ci | When using Cacti with languages other than English, it is important to use the utf8mb4_unicode_ci collation type as some characters take more than a single byte. |
| character_set_client | utf8mb4 | When using Cacti with languages other than English, it is important ot use the utf8mb4 character set as some characters take more than a single byte. |
| max_connections | >= 100 | Depending on the number of logins and use of spine data collector, MySQL will need many connections. The calculation for spine is: total_connections = total_processes * (total_threads + script_servers + 1), then you must leave headroom for user connections, which will change depending on the number of concurrent login accounts. |
| max_heap_table_size | >= 10% RAM |  If using the Cacti Performance Booster and choosing a memory storage engine, you have to be careful to flush your Performance Booster buffer before the system runs out of memory table space. This is done two ways, first reducing the size of your output column to just the right size. This column is in the tables poller_output, and poller_output_boost. The second thing you can do is allocate more memory to memory tables. We have arbitrarily chosen a recommended value of 10% of system memory, but if you are using SSD disk drives, or have a smaller system, you may ignore this recommendation or choose a different storage engine. You may see the expected consumption of the Performance Booster tables under Console -> System Utilities -> View Boost Status.|
| max_allowed_packet | >= 16777216 | With Remote polling capabilities, large amounts of data will be synced from the main server to the remote pollers. Therefore, keep this value at or above 16M. |
| tmp_table_size | >= 64M | When executing subqueries, having a larger temporary table size, keep those temporary tables in memory. |
| join_buffer_size | >= 64M | When performing joins, if they are below this size, they will be kept in memory and never written to a temporary file. |
| innodb_file_per_table | ON | When using InnoDB storage it is important to keep your table spaces separate. This makes managing the tables simpler for long time users of MySQL. If you are running with this currently off, you can migrate to the per file storage by enabling the feature, and then running an alter statement on all InnoDB tables. |
| innodb_file_format | Barracuda | 	When using innodb_file_per_table, it is important to set the innodb_file_format to Barracuda. This setting will allow longer indexes important for certain Cacti tables. |
| innodb_large_prefix	| 1 | If your tables have very large indexes, you must operate with the Barracuda innodb_file_format and the innodb_large_prefix equal to 1. Failure to do this may result in plugins that can not properly create tables. |
| innodb_buffer_pool_size | >=25% RAM | InnoDB will hold as much tables and indexes in system memory as is possible. Therefore, you should make the innodb_buffer_pool large enough to hold as much of the tables and index in memory. Checking the size of the /var/lib/mysql/cacti directory will help in determining this value. We are recommending 25% of your systems total memory, but your requirements will vary depending on your systems size. |
| innodb_doublewrite | ON  | This settings should remain ON unless your Cacti instances is running on either ZFS or FusionI/O which both have internal journaling to accomodate abrupt system crashes. However, if you have very good power, and your systems rarely go down and you have backups, turning this setting to OFF can net you almost a 50% increase in database performance. |
| innodb_lock_wait_timeout | >= 50 | Rogue queries should not for the database to go offline to others. Kill these queries before they kill your system. |
| innodb_flush_log_at_timeout | >= 3 |  	As of MariaDB 10.3.22-1:10.3.22+maria~bionic, the you can control how often MariaDB flushes transactions to disk. The default is 1 second, but in high I/O systems setting to a value greater than 1 can allow disk I/O to be more sequential |
| innodb_read_io_threads | >= 32 | With modern SSD type storage, having multiple read io threads is advantageous for applications with high io characteristics. |
| innodb_write_io_threads | >= 16 | With modern SSD type storage, having multiple write io threads is advantageous for applications with high io characteristics. |
|innodb_buffer_pool_instances| >=	9 | MariaDB will divide the innodb_buffer_pool into memory regions to improve performance. The max value is 64. When your innodb_buffer_pool is less than 1GB, you should use the pool size divided by 128MB. Continue to use this equation upto the max of 64.|
| innodb_io_capacity	| 5000 | If you have SSD disks, use this suggestion. If you have physical hard drives, use 200 * the number of active drives in the array. If using NVMe or PCIe Flash, much larger numbers as high as 100000 can be used. |
| innodb_io_capacity_max | 10000 | If you have SSD disks, use this suggestion. If you have physical hard drives, use 2000 * the number of active drives in the array. If using NVMe or PCIe Flash, much larger numbers as high as 200000 can be used. |
| memory_limit | >= 800M | A minimum of 800 MB memory limit |
| max_execution_time | >= 60 | A minimum of 1 m execution time |

### Data Backups
Included is a backup script that will backup cacti (including settings/plugins), rrd files, and spine. This is accomplished by taking a complete copy of the root spine and cacti directory and performing a MySQL dump of the cacti database which stores all the settings and device information. To manually perform a backup, run the following exec commands: 

```
docker exec <docker image ID or name> ./backup.sh
```

This will store compressed backups in a tar.gz format within the cacti docker container under /backups directory. Its recommended to map this directory using volumes so data is persistent. By default it only stores 7 most recent backups and will automatically delete older ones, to change this value update `BACKUP_RETENTION` environmental variable with the number of backups you wish to store.

### Restore Backup
To restore from an existing backup, run the following docker exec command with the backup file location as an argument.
```
docker exec <docker image ID or name> ./restore.sh /backups/<filename>
```

To get a list of backups, the following command should display them:
```
docker exec <docker image ID or name> ls /backups
```

### Updating Cacti
You can now update the Cacti/Spine version of this container using the included script. By default this will update to the *latest* version. 
```
docker exec <docker image ID or name> ./upgrade.sh
```

If you want to specify a specific version please update the `/upgrade.sh` values.
 * [Cacti Version Links][cacti_download]
 * [Spine Version Links][spine_download]

```
#!/bin/bash
# script to upgrade a cacti instance to latest, if you want a specific version please update the following download links
cacti_download_url=http://www.cacti.net/downloads/cacti-latest.tar.gz
spine_download_url=http://www.cacti.net/downloads/spine/cacti-spine-latest.tar.gz
``` 

## Docker Cacti Architecture
-----------------------------------------------------------------------------
With the recent update to version 1+, Cacti has introduced the ability to have remote polling servers. This allows us to have one centrally located UI and information system while scaling out multiple datacenters or locations. Each instance, master or remote poller, requires its own MySQL based database. The pollers also have an addition requirement to access the Cacti master's database with read/write access.

Some docker-compose examples can be found in the following [readme][arch]

## Container Customization
There are a few customizations you can do if you are building the container locally. During the build process Plugins and Device Templates can be added to folders where at startup, scripts will import and install.

### Device Templates
Dropping device templates in the `/templates/` folder using the following structure:
```
├── templates
│   ├── template_name.xml
│   ├── resource
│   │   └── script_queries
│   │       └── ...
│   │   └── script_server
│   │       └── ...
│   │   └── snmp_queries
│   │       └── ...
│   ├── scripts
│   │   └── ...
```
At buld/first boot you will see some log messages that they were imported to the underlying Cacti system
```
2017-03-24_19:22 [New Install] Installing supporting template files.
2017-03-24_19:22 [New Install] Installing template file /templates/cacti_host_template_juniper_networks.xml
```

### Plugins
To have plugins automatically loaded on boot, simply have the uncompressed plugin in the `plugins` folder within the main directory. Upon build/run, the startup script will automatically install them to the appropriate directory. Please understand that you will need to enable any plugins via Cacti GUI for them to become active. 

To add plugins after the container is built, for example if pulling directly form dockerhub, mount the `/cacti/plugins` directory using [docker volumes][docker_volume_help]. 

### Settings
Settings can be passed through to cacti at initial install by placing the SQL changes in the form of filename.sql under the settings folder. start.sh will automatically merge all *.sql files during install. For example the folling is there to enable spine by default:
##### /settings/spine.sql
```
--
-- Enable spine poller from docker installation
--

REPLACE INTO `%DB_NAME%`.`settings` (`name`, `value`) VALUES('path_spine', '/spine/bin/spine');
REPLACE INTO `%DB_NAME%`.`settings` (`name`, `value`) VALUES('path_spine_config', '/spine/etc/spine.conf');
REPLACE INTO `%DB_NAME%`.`settings` (`name`, `value`) VALUES('poller_type', '2');
```

# Change Log
#### 1.2.11 - 04/17/2020
 * Update Docker container to use Centos8 over Centos7
 * Close issue [#59](https://github.com/scline/docker-cacti/issues/59) - errors with percona on compose single instance; Thank you [miguelwill](https://github.com/miguelwill)
   * Update docker-comose examples with Mariadb:10.3 from older Percona version
 * Close issue [#61](https://github.com/scline/docker-cacti/issues/61) - spine directory and crontab empty after docker-compose down and restoring; Thank you [kevburkett](https://github.com/kevburkett)
   * Update docker-compose with `spine` volume
 * Enable HTTPS functionality and self-sign certs when needed
 * Allow functionality to disable PHP-SNMP usagage via `PHP_SNMP` environment variable
 se 
 * Manual patch on provisioning new remote pollers - https://github.com/Cacti/cacti/issues/3459
 * Update Cacti and Spine from 1.2.8 to 1.2.11
   * [changelog 1.2.10 -> 1.2.11][CL1.2.11]
   * [changelog 1.2.9 -> 1.2.10][CL1.2.10]
   * [changelog 1.2.8 -> 1.2.9][CL1.2.9]

#### 1.2.8 - 12/11/2019
 * Update Cacti and Spine from 1.2.6 to 1.2.8
   * [changelog 1.2.7 -> 1.2.8][CL1.2.8]
   * [changelog 1.2.6 -> 1.2.7][CL1.2.7]

#### 1.2.6a - 10/30/2019
 * Update start.sh to persist Apache Cacti configurations on restart. [#52] (https://github.com/scline/docker-cacti/issues/52)
 * Update docker-compose examples to use different type of volumes so `docker-compose down` will not affect data without the `-v` flag.

#### 1.2.6 - 09/06/2019
 * Update Cacti and Spine from 1.2.0 to 1.2.6
   * [changelog][cacti_changelog]
 * Removed 1.1.X changelog notes from README.md, this can be located in [CHANGELOG.md](https://github.com/scline/docker-cacti/blob/master/changelog.md)
 * Close Issue [#49](https://github.com/scline/docker-cacti/issues/49) - New version of Spine don't have configure file
 * Close Issue [#45](https://github.com/scline/docker-cacti/issues/45) - Directories backup and backups mixed up; thank you [shortbloke](https://github.com/shortbloke) for [PR #46](https://github.com/scline/docker-cacti/pull/46)
 * Merge [PR #47](https://github.com/scline/docker-cacti/pull/47) and [PR #48](https://github.com/scline/docker-cacti/pull/48) - Add modify PHP env; thank you [joey741019](https://github.com/joey741019)

#### 1.2.0 - 01/06/2019
 * Update Cacti and Spine from 1.1.38 to 1.2.0
    * [changelog][cacti_changelog]

 * Add sendmail to dockerfile via yum due to cacti 1.2.0 requirements
 * Created separate changlog file for future documentation cleanup
 * Update PHP variable readme to include `max_execution_time` and `memory_limit` changes for 1.2.0
 * Add and Hotfix the PHP variable `max_execution_time` for PHP_MAX_EXECUTION_TIME and `memory_limit` for PHP_MEMORY_LIMIT

# ToDo
* Auto import remote pollers, currently you need to navigate to there GUI for a few clicks.
* Documentation cleanup.

[CL1.2.11]: http://www.cacti.net/release_notes.php?version=1.2.11
[CL1.2.10]: http://www.cacti.net/release_notes.php?version=1.2.10
[CL1.2.9]: http://www.cacti.net/release_notes.php?version=1.2.9
[CL1.2.8]: http://www.cacti.net/release_notes.php?version=1.2.8
[CL1.2.7]: http://www.cacti.net/release_notes.php?version=1.2.7
[CL1.2.6]: http://www.cacti.net/release_notes.php?version=1.2.6
[CL1.2.5]: http://www.cacti.net/release_notes.php?version=1.2.5
[CL1.2.4]: http://www.cacti.net/release_notes.php?version=1.2.4
[CL1.2.3]: http://www.cacti.net/release_notes.php?version=1.2.3
[CL1.2.2]: http://www.cacti.net/release_notes.php?version=1.2.2
[CL1.2.1]: http://www.cacti.net/release_notes.php?version=1.2.1
[CL1.2.0]: http://www.cacti.net/release_notes.php?version=1.2.0
[cacti_changelog]: https://www.cacti.net/changelog.php
[cacti_download]: http://www.cacti.net/downloads
[spine_download]: http://www.cacti.net/downloads/spine
[arch]: https://github.com/scline/docker-cacti/tree/master/docker-compose
[docker_volume_help]: https://docs.docker.com/engine/tutorials/dockervolumes
[cacti_forums]: http://forums.cacti.net
[cws]: http://cacti.net
