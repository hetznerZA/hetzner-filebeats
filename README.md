# Filebeats

#### Table of Contents

1. [Description](#description)
1. [Setup - The basics of getting started with filebeats](#setup)
    * [Setup requirements](#setup-requirements)
    * [Beginning with filebeats](#beginning-with-filebeats)
1. [Usage - Configuration options and additional functionality](#usage)
1. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
1. [Limitations - OS compatibility, etc.](#limitations)
1. [Development - Guide for contributing to the module](#development)

## Description

Very simple puppet module to install and configure elasticsearch filebeats.

## Setup

puppet module install hetzner-filebeats

### Setup Requirements

Puppet labs APT module
  version 2.3.0 >=

Puppet labs STDLIB module
  version 4.6.0 >= 5.0.0

### Beginning with filebeats

Use puppet module install function to install module and simply include it from your enc/profile/role/site.pp.

## Usage

The module can be called with the following parameters:

#`inputs` OPTIONAL

An array of hashes that specifies which groups of inputs (formally known as prospectors) log entries the filebeats application must export.
This value should be used if you wish to have more than one prospector.

#`logstash_hosts`

An array of strings that specifies remote hosts to use for logstash outputs, e.g ['localhost:5044']
If left empty then all other logstash options are ignored

#`logstash_bulk_max_size`

A Number representing the maximum number of events to bulk in a single Logstash request, e.g 2048
Setting this to zero or negative disables the splitting of batches.

#`logstash_index`

A string that specifies the index to use for the logstash output, defaults to '[filebeat-]YYYY.MM.DD' as per the package.

#`logstash_ssl_certificate_authorities`

An array of Strings that specifies paths to Certificate authority files when connecting to logstash.

#`logstash_ssl_certificate`

A String that specifies a path to your hosts certificate to use when connecting to logstash.

#`logstash_ssl_certificate_key`

A String that specifies a path to your hosts certificate key to use when connecting to logstash.

#`logstash_ttl`

A String that specifies the Time To Live for a connection to Logstash, you must use a elastic duration e.g. '5m', '1h', '45s'
 see https://www.elastic.co/guide/en/beats/libbeat/master/config-file-format-type.html#_duration
 NOTE: this option explicitly disables pipelining, it is not compatible with the async logstash client
 https://www.elastic.co/guide/en/beats/filebeat/current/logstash-output.html#_literal_ttl_literal

#`logstash_worker`

A integer that specifies the number of workers participating in the load balancing

#`logstash_loadbalance`

A boolean to turn on or off load balancing for logstash outputs, defaults to false.

#`elasticsearch_hosts`

A array containing the hostname/s of your elasticsearch host/s used for send the transactions directly
to Elasticsearch by using the Elasticsearch HTTP API.
If left empty then all other elasticsearch options are ignored

#`elasticsearch_proxy_host`

A string containing the hostname of your proxy host used for load balancing your cluster.

#`elasticsearch_username`

The username filebeats should use to authenticate should your cluster make use of shield

#`elasticsearch_password`

The password filebeats should use to authenticate should your cluster make use of shield

#`elasticsearch_protocol`

A string containing the protocol used by filebeats, defaults to http. 

#`elasticsearch_index`

A string that specifies the index to use for the elasticsearch output, defaults to '[filebeat-]YYYY.MM.DD' as per the package.

#`elasticsearch_ilm`

A boolean that specifies whether to enable Elastic's ILM option, defaults to false

#`elasticsearch_ssl_certificate_authorities`

An array of Strings that specifies paths to Certificate authority files.

#`elasticsearch_ssl_certificate`

A String that specifies a path to your hosts certificate to use when connecting to elasticsearch.

#`elasticsearch_ssl_certificate_key`

A String that specifies a path to your hosts certificate key to use when connecting to elasticsearch.

#`elasticsearch_template_enabled`

A boolean that allows you to overwrite template loading.

#`elasticsearch_template_name`

A string that specifies the index template to use for setting mappings in Elasticsearch.

#`elasticsearch_template_overwrite`

A boolean that allows you to overwrite the existing template.

#`elasticsearch_template_path`

A string that specifies the path to the template file

#`export_log_paths`

An array of Strings that specifies which logs the filebeats application must export.

#`ilm_check_exits`

A boolean when set to false, disables the check for an existing lifecycle policy. The default is true. You need to disable 
this check if the Filebeat user connecting to a secured cluster doesn’t have the read_ilm privilege

#`ilm_enabled`

A string that Enables or disables index lifecycle management on any new indices created by Filebeat. Valid values are
true, false, and auto (because auto is also an option, this can be a puppet Boolean)

#`ilm_overwrite`

A boolean when set to true, the lifecycle policy is overwritten at startup

#`ilm_pattern`

A string that specifies the rollover index pattern. Date math is supported in this setting

#`ilm_policy_file`

A string that specifies the path to a JSON file that contains a lifecycle policy configuration. Use this setting to load your
own lifecycle policy

#`ilm_policy_name`

A string that specifies the name to use for the lifecycle policy

#`ilm_rollover_alias`

A string that specifies the index lifecycle write alias name

#`log_settings`

A puppet Hash containing log level ('debug', 'warning', 'error' or 'critical'), to_syslog(true/false), path('/var/log/filebeat'), keepfiles(7), rotateeverybytes(10485760), name(filebeats.log)

#`service_bootstrapped`

A boolean to turn on or off the filebeat service at boot ('false'/'true'), defaults to 'true'

#`service_state`

A string to describe the state of the filebeats service ('stopped'/'running'), defaults to 'running'

## Example

Auth.log being exported with elasticsearch out requiring a user and password.

```
   class { 'filebeats':
     export_log_paths         => ['/var/log/auth.log'],
     elasticsearch_username          => 'host',
     elasticsearch_password          => 'secret',
     elasticsearch_host => ['elasticsearchproxy.myserver.com'],
   }
```

Multiple inputs with multiple log files being exported to multiple logstash hosts.

```
   class { 'filebeats':
     inputs               => [{ 'input_type'    => 'log',
                                'doc_type'      => 'log',
                                'paths'         => ['/var/log/auth.log'],
                                'include_lines' => "['sshd','passwd','vigr']",
                              },
                              { 'input_type'    => 'log',
                                'doc_type'      => 'apache',
                                'paths'         => ['/var/log/apache2/access.log', '/var/log/apache2/error.log'],
                                'fields'        => {'level' => 'debug', 'review' => 1},
                                'exclude_lines' => "['warning'']",
                              }
                             ]
     logstash_hosts       => ['logstash1.domain.com', 'logstash2.domain.com'],
     logstash_loadbalance => true,
   }
```

## Hiera data example

```
filebeats::inputs:
  - input_type: 'log'
    paths:
      - '/var/log/auth.log'
    doc_type: 'auth'
    include_lines: ['sshd','passwd','vigr']
  - input_type: 'log'
    paths:
      - '/var/log/my_app.log'
    doc_type: 'my_app'
    exclude_lines: ['warning']
  - input_type: 'log'
    paths:
      - '/var/log/security.log'
    doc_type: 'security'
    fields:
      level: 'debug'
      review: 1
  - input_type: 'log'
    paths:
      - '/var/log/app.json'
    tags:  ['json']
    json_keys_under_root: true
    json_overwrite_keys: true
    json_add_error_key: true
    json_message_key: 'log'
```

## Reference

* `Package`

Configures the apt resrouce for filebeats.

* `Config`

Configures the filebeats.yml file.

* `Service`

Ensures the service is running.

* `Params`

Specifies defaults for the installation and configuration

## Limitations

Does not support all options available to filebeats configuration.

If you're running a Filebeat version lower than 7.x, e.g 6.8.10. You need to install the `pre-v7.x` release in order for the module
to be compatible - https://github.com/xneelo/hetzner-filebeats/releases

If you're running Filebeat version 7 and up, release 2.0.0 and newer is supported.

## Development

All pull requests are welcome. This module was just created for our use and functionality will be added as we require it.
