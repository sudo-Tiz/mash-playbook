# Prometheus Ansible role

[Promtail](https://grafana.com/oss/promtail/) is a log aggregation system designed to store and query logs from all your applications and infrastructure. This role helps you to set up promtail:

- with everything run in [Docker](https://www.docker.com/) containers
- powered by [the official promtail container image](https://hub.docker.com/r/grafana/promtail/)


## Installing

To configure and install promtail on your own server(s), you should use a playbook like [Mother of all self-hosting](https://github.com/mother-of-all-self-hosting/mash-playbook) or write your own.

## Configuration

To enable this service, add the following configuration to your `vars.yml` file and re-run the [installation](../installing.md) process:

```yaml
########################################################################
#                                                                      #
# promtail                                                             #
#                                                                      #
########################################################################

promtail_enabled: true

########################################################################
#                                                                      #
# /promtail                                                            #
#                                                                      #
########################################################################
```

### Exposing the web interface

By setting a hostname you will expose promtail on this domain.
Usually you should also set up basic_auth in this case, otherwise everyone will be able to access your metrics

```yaml
promtail_hostname: promtail.example.org

# Uncommenting the following lines allows you to configure basic auth
# promtail_basic_auth_enabled: true
# promtail_basic_auth_username: ''
# promtail_basic_auth_password: ''
```
### Scraping logs
#### Default scraping behavior
By default, promtail will scrape systemd-journald and ssh logs you can enable/disable this behavior by setting :

```yaml
promtail_journald_scraper_enabled: false
promtail_varlog_scraper_enabled: false
```
#### Scraping other logs
##### Exemple with ssh: 

Use ``promtail_container_additional_mounts_custom`` to add volumes to monitor:
```yaml
promtail_container_additional_mounts_custom:
 - "type=bind,source=</path/to/ssh/logs>,target=/data/ssh,readonly"
```


Use ``promtail_config_scrape_additional`` to define your scraping configuration:
```yaml
promtail_config_scrape_additional: >-
  - job_name: ssh
    static_configs:
    - localhost
      __path__: /data/ssh
      labels:
        job: ssh
```



##### Exemple with syslog:
Following exemple show how to use rsyslog and promtail to scrape syslog using port 1514   

Prerequesites :  
*Edit your rsyslog configuration in order to send logs to promtail.*  
*This could be done by creating a /etc/rsyslog.d/00-promtail-relay.conf with following content:*
```
*.* action(type="omfwd" protocol="tcp" target="<promtail_host>" port="<promtail_port>" Template="RSYSLOG_SyslogProtocol23Format" TCP_Framing="octet-counted" KeepAlive="on")
```

You could need to expose container to acces it from host:
```yaml
promtail_container_additionnal_bind_ports:
  - ["<optionnal_host_ip>:<host_port>", "<promtail_port>"]
```

Use ``promtail_config_scrape_additional`` to define your scraping configuration:
```yaml
promtail_config_scrape_additional: >-
  - job_name: syslog
    syslog:
      listen_address: 0.0.0.0:1514
      labels:
        job: syslog
    relabel_configs:
      - source_labels: [__syslog_message_hostname]
        target_label: host
      - source_labels: [__syslog_message_hostname]
        target_label: hostname
      - source_labels: [__syslog_message_severity]
        target_label: level
      - source_labels: [__syslog_message_app_name]
        target_label: application
      - source_labels: [__syslog_message_facility]
        target_label: facility
      - source_labels: [__syslog_connection_hostname]
        target_label: connection_hostname
```

Check [defaults/main.yml](defaults/main.yml l) for the full list of supported options.

## Recommended other services

- [Grafana](grafana.md) - a web-based tool for visualizing your Promtail logs
