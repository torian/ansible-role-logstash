# Ansible role for Logstash

[![Build Status](https://travis-ci.org/torian/ansible-role-logstash.svg)](https://travis-ci.org/torian/ansible-role-logstash)

This ansible role installs [Logstash](https://www.elastic.co/products/logstash) 
through the official repository packages.

## Supported Platforms
  * EL / Centos (6 / 7)
  * Debian (Wheezy / Jessie)
  * Ubuntu (Precise / Trusty)
  * AMZ Linux

## Role Variables

The following role variables are defined in `defaults/main.yml`. For a
detailed explanation about them you can take a look at the file.

```
logstash_version: 5.6.4

logstash_daemon_user: root

logstash_install_dir: /usr/share/logstash
logtash_conf_prefix:  /etc/logstash
logstash_conf_dir:    "{{logstash_conf_prefix}}/conf.d"
logstash_data_dir:    /var/lib/logstash

logstash_plugins:
  - logstash-input-s3
  - logstash-output-s3
```

The defaults for the JVM config (`jvm.options`) are based on the values
shipped by logstash

```
logstash_jvm_mem: 1g

logstash_config_jvm_defaults: |
  -Xms{{logstash_jvm_mem}}
  -Xmx{{logstash_jvm_mem}}
  -XX:+UseParNewGC
  -XX:+UseConcMarkSweepGC
  -XX:CMSInitiatingOccupancyFraction=75
  -XX:+UseCMSInitiatingOccupancyOnly
  -XX:+DisableExplicitGC
  -Djava.awt.headless=true
  -Dfile.encoding=UTF-8
  -XX:+HeapDumpOnOutOfMemoryError

logstash_config_jvm: "{{logstash_config_jvm_defaults}}"
```

The defaults for the daemon config (`logstash.yml`) are also based
on the defaults shipped by logstash, but in this case, they values
of `logstash_config_daemon_defaults` and `logstash_config_daemon`
are merged using the jinja2 filter `combine()`:

```
logstash_config_daemon_defaults:
  path.data: "{{logstash_data_dir}}"
  path.config: "{{logstash_conf_dir}}"
  path.logs: "{{logstash_logs_dir}}"

logstash_config_daemon: {}
```

## Usage

Setting the configuration for `input`, `filter` and `output` is specified
using the following special vars:

  * `logstash_inputs`
  * `logstash_filters`
  * `logstash_outputs`

This vars are expanded into their own sections `input {}`, `filter {}` and
`output {}`, so you are free to specify whatever config suits you, i.e.:

```
---
- hosts: all

  vars:
    - logstash_inputs: |
        file {
            path => "/var/log/nginx/access.log"
            tags => ["nginx"]
        }
        file {
            path => "/var/log/nginx/error.log"
            tags => ["nginx"]
        }

    - logstash_filters: |
        grok { match => [ "message", "%{HTTPDATE:[@metadata][timestamp]}" ] }
        date { match => [ "[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z" ] }

    - logstash_outputs: |
        stdout { codec => rubydebug }


  roles:
    - { role: torian.logstash}
```

### Installing additional plugins

By default, and just as an example, the role installs two plugins:

```
logstash_plugins:
  - logstash-input-s3
  - logstash-output-s3
```

If you don't want them, or need to specify different ones, the just override
the default setting

### Logstash version upgrade

If you need to upgrade from a previous logstash version, the role can
manage it. Specify an extra var `logstash_upgrade=True` and the package manager
is going to install the latest available release that matches the major from
`logstash_version`.

