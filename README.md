# Description

This role provisions a [Nimbus](https://nimbus.team/) [Fluffy](https://github.com/status-im/nimbus-eth1/tree/master/fluffy) Eth1 node.

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
- name: infra-role-nimbus-fluffy
  src: git+git@github.com:status-im/infra-role-nimbus-fluffy.git
  scm: git
```

# Configuration

The crucial settings are:
```yaml
# branch which should be built
nimbus_fluffy_repo_branch: 'master'
# Portal network to connect to
nimbus_fluffy_network: 'mainnet'
# optional setting for debug mode
nimbus_fluffy_log_level: 'DEBUG'
```

# Management

## Service

Assuming the `stable` branch was built you can manage the service with:
```sh
sudo systemctl start nimbus-fluffy-master
sudo systemctl status nimbus-fluffy-master
sudo systemctl stop nimbus-fluffy-master
```
You can view logs under:
```sh
tail -f /data/nimbus-fluffy-master/logs/service.log
```
All node data is located in `/data/nimbus-fluffy-master/data`.

## Builds

A timer will be installed to build the image:
```sh
 > sudo systemctl list-units --type=service '*nimbus-fluffy-*'
  UNIT                         LOAD   ACTIVE SUB     DESCRIPTION
  nimbus-fluffy-master.service loaded active running Nimbus Eth1 Fluffy node (master)
  nimbus-fluffy-debug.service  loaded active running Nimbus Eth1 Fluffy node (debug)
```
To rebuild the image:
```sh
 > sudo systemctl start build-nimbus-fluffy-mainnet-master.service
 > sudo systemctl status build-nimbus-fluffy-mainnet-master.service
 ● nimbus-fluffy-master-build.service - Build nimbus-fluffy-master
     Loaded: loaded (/etc/systemd/system/nimbus-fluffy-master-build.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Wed 2021-09-29 12:00:12 UTC; 2h 15min ago
TriggeredBy: ● nimbus-fluffy-master-build.timer
       Docs: https://github.com/status-im/infra-role-systemd-timer
    Process: 1212987 ExecStart=/data/nimbus-fluffy-master/build.sh (code=exited, status=0/SUCCESS)
   Main PID: 1212987 (code=exited, status=0/SUCCESS)

Sep 29 12:00:12 build.sh[1213054]: HEAD is now at f782327f reimplement engine API rpc kiln spec v2
Sep 29 12:00:12 build.sh[1212987]:  >>> Binary already built
Sep 29 12:00:12 systemd[1]: nimbus-fluffy-master-build.service: Succeeded.
Sep 29 12:00:12 systemd[1]: Finished Build nimbus-fluffy-master.
```
To check full build logs use:
```sh
journalctl -u build-nimbus-fluffy-mainnet-master.service
```

# Requirements

Due to being part of Status infra this role assumes availability of certain things:

* The `iptables-persistent` module
