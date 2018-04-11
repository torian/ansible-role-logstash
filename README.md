# Ansible role for Logstash

[![Build Status](https://travis-ci.org/torian/ansible-role-logstash.svg)](https://travis-ci.org/torian/ansible-role-logstash)

This Ansible role installs [Logstash](https://www.elastic.co/products/logstash)
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
logstash_conf_prefix:  /etc/logstash
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

Setting configuration for `input`, `filter`, and `output` happens in the `logstash_configs` list. Each list item in `logstash_configs` takes a `name`, `config`, and optional `pipeline`. When `pipeline` is omitted the configuration is placed in `logstash_conf_dir` directory and picked up by the `main` pipeline (which is the default when no `pipelines.yml` is configured).

An example single configuration file for the default `main` pipeline could be:

```
logstash_configs:
  - name: mylogs
    config: |
      input{
        beats {
          host => "0.0.0.0"
          port => 5044
          codec => "json"
        }
      }
      filter {
        uuid {
          target => "uuid"
        }
      }
      output {
        file {
          path => "/var/log/mylogs/%{logger}-%{+YYYY-MM-dd}.log.gz"
          codec => json_lines
        }
      }
```

Alternatively, this can be split into multiple files. The following would generate two configuration files, `/etc/logstash/conf.d/input.conf` and `/etc/logstash/conf.d/output.conf`:

```
logstash_configs:
  - name: input
    config: |
      input {
        beats {
          host => "0.0.0.0"
          port => 5044
          codec => "json"
        }
      }
  - name: output
    config: |
      output {
        stdout {
          codec => rubydebug
        }
      }
```

The `pipeline` parameter for each `logstash_configs` item allows you to specifcy different configurations per pipeline.

A two pipeline example:

```
logstash_pipelines:
  - name: application_events
    config:
      pipeline.workers: 3
  - name: system_logs
logstash_configs:
  - name: ec2_instances
    pipeline: system_logs
    config: |
      input {
        file {
          path => ["/var/log/*.log"]
        }
      }
      output {
        elasticsearch {
          hosts => ["myesloghost:9200"]
        }
      }
  - name: user_events
    pipeline: application_events
    config: |
      input {
        http {
          host => "0.0.0.0"
          port => 8080
        }
      }
      output {
        elasticsearch {
          hosts => ["myesmetricshost:9200"]
        }
      }
```

This allows for different pipelines to help split groups of different inputs and outputs.

There are also some legacy deprecated variables that can be used instead of `logstash_pipelines` and `logstash_configs`:

  * `logstash_inputs`
  * `logstash_filters`
  * `logstash_outputs`

These vars are expanded into their own sections `input {}`, `filter {}` and
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

These vars get wrapped in a default `logstash_configs` which is set as:

```
logstash_configs:
  - name: filters
    config: |
      filter {
        {{ logstash_filters }}
      }
  - name: input
    config: |
      input {
        {{ logstash_inputs }}
      }
  - name: output
    config: |
      output {
        {{ logstash_outputs }}
      }
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
