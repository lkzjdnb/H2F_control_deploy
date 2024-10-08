# Ansible Playbook for the deployement of the H2F control software
This repository contains a playbook used to install the [H2F control interface](https://github.com/lkzjdnb/H2F_control) and all needed services.

## Remote preparation 
> This playbook as been tested on debian 12 but should work on any derivatives without too much modifications.

You should have access to the server via ssh, for security and convenience it is recommanded to setup key-based authentification.
> Your user should have superuser access via sudo (for doas add `--become-method=doas` when invoking ansible)

Network setup with outside remotes and local devices (WiFi authentification, IP assignement, USB authorisation, ...) has to be done according to your installation prior to running this playbook (See [network-setup.md](/docs/network-setup.md) for more information).
> :warning: Make sure to reflect your installation in the configuration ([bridge_config](https://github.com/lkzjdnb/bridge_config)).

> If you are running this playbook on a test environment without devices (VM for example) see [Testing](#testing)

## Setup
Download the playbook:
```
git clone https://github.com/lkzjdnb/H2F_control_deploy.git
cd H2F_control_deploy
```

Add the remote to the `inventory/hosts` file : 

`remote_name ansible_host=<remote_url> ansible_ssh_user=<username> ansible_ssh_pipelining=False`

Create configuration for the remote in the file `inventory/host_vars/<remote_name>/vars.yml` setting variables (see the example for help).

## Migrating from previous installation
To import data from an other influxDB installation : 
- Export the data as line-protocol from the previous server ([see wiki](https://docs.influxdata.com/influxdb/v2/write-data/migrate-data/migrate-oss/)) : 
```
influxd inspect export-lp \
  --bucket-id 12ab34cd56ef \
  --engine-path ~/.influxdbv2/engine \
  --output-path path/to/export.lp
  --compress
```
- Add the generated file to `./files`
- Add the file to the `influx_migrate_file` variable in `host_vars/<hostname>/vars.yml`

> Make sure to remove it from the configuration after this initial install otherwise each subsequent update will reupload the data

## Running
> Because Ansible support on Windows is not easy, a thin docker wrapper around ansible is provided. See [running using docker](docs/Running-on-docker.md)

Run the playbook, this will install and setup all service. And can take a few minutes (30 mins) to finish : 

`ansible-playbook -v -i inventory/hosts playbook.yaml --tags "install-all"`

If everything went correctly the interface should have started and you should now be able to access the services : 
- Grafana at http://grafana.<hostname>
- InfluxDB at http://influx.<hostname>
- The webUI at http://webui.<hostname>

# Maintenance
For compatibility this playbook use docker-compose as a service manager.

Services configurations can be found in the user home in the folder defined by the `folder` variable (default `/home/<user>/station_control`). With a folder for each service.

Log can be viewed with `docker-compose logs` while in the service folder.

Started services can be viewed with `docker ps`.

Services can be started/stopped with `docker-compose up`/`docker-compose down`.

# Testing
To create a test environment for this project see more informations at [Testing.md](/docs/Testing.md).
