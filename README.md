# pluggable Ansible

## purpose

An Ansible repo with pluggable roles, such as:
- databases: MySQL/MariaDB, PostgreSQL
- scripting: PHP, Ruby
- webservers: Apache2 (httpd), Nginx

## prerequisites
- RHEL 8
- Ansible 2.12 or higher
- an Ansible service account with authorized ssh login
- Vaultwarden 2023 or equivalent

## content
- pluggable roles that can perform a **default** installation of aforementioned components
- a customization role that performs a **specific** installation and/or configuration
- installing PostgreSQL, or Nginx, does only that: the installation of a software package, without any project related details
- the custom role is designed for the latter: the creation of database, the configuration of a webserver

## variables
The idea is to centralize variables, and to define default values, as much as possible.

### defaults

- try to define **default** values as often as possible, in your YAML or Jinja2 syntax
- grouping the variables into **one file** improves readability
- refering to your variables is an easy syntax, for example: {{ mariadb.port }}

```yml
# {{ ansible_managed }}
[mariadb]
datadir = {{ mariadb.data_dir | default('/var/lib/mysql/', true) }}
general_log_file = {{ mariadb.log_dir | default('/var/log/mariadb', true) }}/mariadb.log
general_log = 1
log-error = {{ mariadb.log_dir | default('/var/log/mariadb', true) }}/mariadb_error.log
log_bin = {{ mariadb.log_dir | default('/var/log/mariadb', true) }}/mariadb_bin
port = {{ mariadb.port | default('3306', true) }}
```

### defaults/main.yml

```yml
mariadb:
  config_file: /etc/my.cnf
  data_dir: /var/lib/mysql/
  group: mysql
  log_dir: /var/log/mariadb/
  logs:
    - mariadb.log
    - mariadb_error.log
  owner: mysql
  packages:
    - { package: 'mariadb-server', state: 'present' }
    - { package: 'mariadb-connector-c', state: 'present' }
    - { package: 'python3-PyMySQL', state: 'present' }
    - { package: 'perl-DBD-MySQL', state: 'present' }
  port: 3306
```
## customize

After a *default installation*, that is: no customization added, your setup will look like this:
- databases: PostgreSQL or MySQL does not contain anything project related, the server is initialized, the passwords retrievable from Vaultwarden
- webservers: no vhosts (Apache2) or server blocks (Nginx) are created, your configuration is *stripped from default values* and should be secure
- In case you need something specific, adapt the required variables and *change only the custom role*.

## usage

values between square brackets are optional **[ ]** or an array of options **[ x | y ]** :

```yml
ansible-playbook [ -l <hosts> | -i <inventory> ] [ --become ] [ --check ] -e "env=<environment> [ database_type= mariadb | postgresql ] 
     [ webserver_type= httpd | nginx ] [ scripting_language= php | ruby ]" site.yml
```

## license
GPLv3
