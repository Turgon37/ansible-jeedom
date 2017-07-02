Ansible Role Jeedom
========

:warning: This role is under development


This roles configures an instance of Jeedom with all dependencies except Web Server.
I consider that is not to the responsibility of this role to setup the webserver.
Please use another role that configure an apache webserver like [Apache2 role](https://github.com/Turgon37/ansible-apache2)

## OS Family

This role is available only for Debian and in fact for Raspbian too.


## Features

At this day the role can be used to configure :

  * Jeedom user/group, filesystem rights
  * Required packages installation
  * Jeedom cron task management
  * PHP directive configurations

## Jeedom supported version

  * 2.4.6

## Configuration

### Server

The following variables are user writable  :

| Name                       | Description                                            |
| ---------------------------|--------------------------------------------------------|
| jeedom__version            | The name of the branch to use for first installation   |
| jeedom__hook_version       | The real version of Jeedom,use to apply specifics hooks|
| jeedom__php_version        | The php version you want to use (5/7)                  |
| jeedom__root               | The root path for all jeedom files (ex /var/www/html)  |
| jeedom__user               | The system user you want jeedom run as                 |
| jeedom__config_db_host     | The hostname of the database server                    |
| jeedom__config_db_port     | The port of the database server                        |
| jeedom__config_db_dbname   | The name of the database                               |
| jeedom__config_db_username | The username use to login to the database server       |
| jeedom__config_db_password | The password use to login to the database server       |

The following variables are automatically fill by hooks :

| Name                  | Description                          |
| ----------------------|--------------------------------------|
| jeedom__packages      | The list of package that Jeedom needs|
| jeedom__cron          | The list of crontask descriptions    |
| jeedom__php_directive | A set of php.ini directives to set   |

### Example

* Configure a serveur with a Jeedom in /var/www/html

```
- hosts: all
  roles:
    - jeedom
  vars:
    jeedom__version: stable
    jeedom__hook_version: 2.4.6
    jeedom__root: /var/www/html
    jeedom__config_db_host: 'mysql.example.com'
    jeedom__config_db_port: '3306'
    jeedom__config_db_dbname: 'www_jeedom'
    jeedom__config_db_username: 'www_jeedom'
    jeedom__config_db_password: 'CXXXXXXXXXXXXXXXX'
```
