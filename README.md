# Description

This role provisions a [Portal Nework Client](https://github.com/status-im/nimbus-eth1/tree/master/portal) Eth1 node devleoped by [Nimbus Team](https://nimbus.team/) .

# Introduction

The role will:

* Checkout a branch from the [nimbus-eth1](https://github.com/status-im/nimbus-eth1) repo
* Build it using the [`build.sh`](./templates/build.sh.j2) Bash script
* Schedule regular builds using [Systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
* Start a node by defining a [Systemd service](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

# Ports

The service exposes three ports by default:

* `9009` - DevP2P UDP peering port. Must __ALWAYS__ be public.
* `9200` - Prometheus metrics port. Should not be public.
* `9900` - JSON RPC port. Must __NEVER__ be public.

# Installation

Add to your `requirements.yml` file:
```yaml
- name: infra-role-nimbus-portal-client
  src: git+git@github.com:status-im/infra-role-nimbus-portal-client.git
  scm: git
```

# Configuration

The crucial settings are:
```yaml
# branch which should be built
nimbus_portal_client_repo_branch: 'master'
# Portal network to connect to
nimbus_portal_client_network: 'mainnet'
# optional setting for debug mode
nimbus_portal_client_log_level: 'DEBUG'
```

# Management

## Service

Assuming the `stable` branch was built you can manage the service with:
```sh
sudo systemctl start nimbus-portal-client-master
sudo systemctl status nimbus-portal-client-master
sudo systemctl stop nimbus-portal-client-master
```
You can view logs under:
```sh
tail -f /data/nimbus-portal-client-master/logs/service.log
```
All node data is located in `/data/nimbus-portal-client-master/data`.

## Builds

A timer will be installed to build the image:
```sh
 > sudo systemctl list-units --type=service '*nimbus-portal-client-*'
  UNIT                         LOAD   ACTIVE SUB     DESCRIPTION
  nimbus-portal-client-master.service loaded active running Nimbus Eth1 Portal Network node (master)
  nimbus-portal-client-debug.service  loaded active running Nimbus Eth1 Portal Network node (debug)
```
To rebuild the image:
```sh
 > sudo systemctl start build-nimbus-portal-client-master
 > sudo systemctl status build-nimbus-portal-client-master
 ● nimbus-portal-client-master-build.service - Build nimbus-portal-client-master
     Loaded: loaded (/etc/systemd/system/nimbus-portal-client-master-build.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Wed 2021-09-29 12:00:12 UTC; 2h 15min ago
TriggeredBy: ● nimbus-portal-client-master-build.timer
       Docs: https://github.com/status-im/infra-role-systemd-timer
    Process: 1212987 ExecStart=/data/nimbus-portal-client-master/build.sh (code=exited, status=0/SUCCESS)
   Main PID: 1212987 (code=exited, status=0/SUCCESS)

Sep 29 12:00:12 build.sh[1213054]: HEAD is now at f782327f reimplement engine API rpc kiln spec v2
Sep 29 12:00:12 build.sh[1212987]:  >>> Binary already built
Sep 29 12:00:12 systemd[1]: nimbus-portal-client-master-build.service: Succeeded.
Sep 29 12:00:12 systemd[1]: Finished Build nimbus-portal-client-master.
```
To check full build logs use:
```sh
journalctl -u build-nimbus-portal-client-master.service
```

# Requirements

Due to being part of Status infra this role assumes availability of certain things:

* The `iptables-persistent` module
