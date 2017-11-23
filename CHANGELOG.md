# Ansible Logstash role Changelog

## 2017-11-23: 1.2.0

  * Fixed issue that would make the role install the latest logstash version
    (given a mayor), instead of the specified one
  * Logstash version: Bumped default installation version
  * Switched from `command` module on plugins installation to `logstash_plugin`

## 2017-05-31: 1.1.0

  * Fixed configuration indentation
  * Added configuration for `jvm.options` file (`logstash_config_jvm`)
  * Added configuration for `logstash.yml` file (`logstash_config_daemon`)
  * Removed no longer used `logstash_opts` var, used in `/etc/[sysconfig,default]/logstash`


## 2017-03-21: 1.0.1

  * Ansible lint - Fixed trailing whitespaces
  * Ansible lint - Fixed ANSIBLE0012 using when

## 2016-12-29: 1.0.0

  * Support for 5.x
  * Removed java package installation. You should use a role for that
  * Removed sudo / become from task handlers

## 2016-02-25: 0.2.0

  * Documentation
  * Upgrade Logstash version with `logstash_upgrade`
  * Support for 2.x

