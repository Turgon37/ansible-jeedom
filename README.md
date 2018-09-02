Ansible Role Jeedom
========

[![Build Status](https://travis-ci.org/Turgon37/ansible-jeedom.svg?branch=master)](https://travis-ci.org/Turgon37/ansible-jeedom)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-Turgon37.jeedom-blue.svg)](https://galaxy.ansible.com/Turgon37/jeedom/)

## Description

:grey_exclamation: Before using this role, please know that all my Ansible roles are fully written and accustomed to my IT infrastructure. So, even if they are as generic as possible they will not necessarily fill your needs, I advice you to carrefully analyse what they do and evaluate their capability to be installed securely on your servers.

This roles install and configures Jeedom Web Application.

## Requirements

Require Ansible >= 2.4

### Dependencies

Jeedom require at least the following things :

  * a PHP engine (Apache + mod_php is supported)
  * a Mysql/MariaDB database
  * a jobs scheduler (cron)
  * an optional sudoers role to configure sudo rights

I personally provide the required roles but this role does not strictly require them. So you are free to use any equivalent role to fit the needs above.

If you want to use an advanced sudoers rule management role you have to use this rule [ansible-sudoers](https://github.com/Turgon37/ansible-sudoers). And set jeedom__sudo_use_sudoers_role to true.

If you use the zabbix monitoring profile you will need the role [ansible-zabbix-agent](https://github.com/Turgon37/ansible-zabbix-agent)

## OS Family

This role is available for Debian

## Features

At this day the role can be used to :

  * install jeedom
  * configure jeedom (database credentials)
  * perform first installation of jeedom
  * monitoring items for
    * Zabbix
  * [local facts](#facts)

## Configuration

### Role

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below. To see default values please refer to this file.

| Name                            | Types/Values| Description                                                                      |
| ------------------------------- | ------------|--------------------------------------------------------------------------------- |
| `jeedom__install_version`       | String      | Jeedom version release to install (default to stable)                            |
| `jeedom__install_directory`     | String      | Directory where jeedom will be installed                                         |
| `jeedom__sudo_use_sudoers_role` | Boolean     |Enable the usage of advanced sudoers rule [see dependencies above](#Dependencies) |


## Facts

I deliver the following facts with this role. This is explicitly NOT intended to be used within your ansible run as it will only work on your second run. Its intention is to make querying versions per server.

* ```ansible_local.jeedom.version_full```
* ```ansible_local.jeedom.version_major```
* ```ansible_local.jeedom.installed_plugins```


## Example

### Playbook

Use it in a playbook as follows:

```yaml
- hosts: all
  roles:
    - turgon37.jeedom
```

### Inventory

Some examples:

* Basic configuration with Zabbix monitoring of processus

```
jeedom__install_directory: /var/www/html

jeedom__config_db_dbname: application_jeedom
jeedom__config_db_username: application-jeedom
jeedom__config_db_password: '{{ vault_mysql_jeedom }}'

jeedom__processus_monitoring:
  - process_name: python
    user: root
    cmdline: broadlinkd.py
  - process_name: python
    user: '{{ jeedom__service_user }}'
    cmdline: openzwaved.py
  - process_name: python
    user: root
    cmdline: rfplayer2d.py
  - process_name: python
    user: '{{ jeedom__service_user }}'
    cmdline: xiaomihomed.py
```

* With apache configuration

Currently, my Jeedom configuration require this set of Apache mods

```
apache2__modules_enabled:
  - access_compat # provide supports for old directives Allow,Order that are deprecated
  - alias  # provide  Alias
  - authn_core
  - authz_core
  - deflate # provide Gzip compression
  - dir # provide  DirectoryIndex
  - env  # provide SetEnv
  - headers  # provide RequestHeader
  - mime
  - mpm_prefork
  - negotiation # handle Content Type
  - php7.0
  - ssl  # Handle SSL
  - socache_shmcb  # required by mod_ssl
```

Then use a virtual host like the following, (configuration based on this [apache](https://github.com/Turgon37/ansible-apache2) role)

```
jeedom-http:
  hosts:
    - ip: 10.0.0.1
    - ip: 127.0.0.1
  server_name: 'jeedom.example.com'
  document_root: '{{ jeedom__install_directory }}'
  document_root_user: '{{ apache2__service_user }}'
  document_root_group: '{{ apache2__service_user }}'
  error_log: '{{ jeedom__install_directory }}/log/http.error'
  error_log_user: '{{ apache2__service_user }}'
  error_log_group: '{{ apache2__service_user }}'
  sections:
    - type: directory
      path: '{{ jeedom__install_directory }}'
      directives:
        - allow_override: All
        - options: -Indexes -ExecCGI -FollowSymLinks
        - require: all granted
    - type: files_match
      regex: '\.(appcache|atom|bbaw|bmp|crx|css|cur|eot|f4[abpv]|flv|geojson|gif|htc|ico|jpe?g|js|json(ld)?|m4[av]|manifest|map|mp4|oex|og[agv]|opus|otf|pdf|png|rdf|rss|safariextz|svgz?|swf|topojson|tt[cf]|txt|vcard|vcf|vtt|webapp|web[mp]|webmanifest|woff2?|xloc|xml|xpi)$'
      directives:
        - header:
            - unset Content-Security-Policy
            - unset X-Frame-Options
            - unset X-XSS-Protection
  directives:
    - header:
        # Reducing MIME type security risks
        - set X-Content-Type-Options "nosniff"
        # Clickjacking
        - set X-Frame-Options "DENY"
        # Reflected Cross-Site Scripting (XSS) attacks
        - set X-XSS-Protection "1; mode=block"
        - unset X-Powered-By
```