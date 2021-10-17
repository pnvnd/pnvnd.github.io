---
layout: post
title: "Monitor Synology with New Relic"
subtitle: "Linux (Upstart) Infrastructure Agent Install"
background: '/img/newrelic-synology/newrelic_synology_01.png'
---

# Monitoring Synology DSM Infrastructure
Synology DSM is a custom build of Linux using Upstart.  Follow the [tarball assisted install for the New Relic Infrastructure Agent on Synology](https://docs.newrelic.com/docs/infrastructure/install-infrastructure-agent/linux-installation/tarball-assisted-install-infrastructure-agent-linux)

In this example, I use a Synology DS214play device with DSM 6.2 installed.

## Summary for Intel Atom Devices
Start by enabling SSH on your Synology device:  
1. Control Panel
2. Terminal & SNMP
3. Terminal > Enable SSH Service

Next, open your terminal and enter `ssh user@synology`.  You may get an error message saying your home directory does not exist and will start at the room directory `/`.
1. Download the Intel x86 New Relic Infrastructure Agent: `sudo curl https://download.newrelic.com/infrastructure_agent/binaries/linux/386/newrelic-infra_linux_1.20.4_386.tar.gz --output newrelic-infra_linux_1.20.4_386.tar.gz`
2. Extract: `sudo tar -xf newrelic-infra_linux_1.20.4_386.tar.gz`
3. Append your New Relic license key (or edit in `vi`): `echo "license_key: a1b2c3d4e5f6g6h7i7j8k9l0m9n8o7p6q5r4NRAL" | sudo tee -a /etc/newrelic-infra/config_defaults.sh`
4. Run installer script: `sudo /etc/newrelic-infra/installer.sh`
5. Check if service is running: `sudo initctl status newrelic-infra`
6. If service is not running, try restarting: `sudo initctl restart newrelic-infra`

## Infrastructure Data Captured
1. System: CPU %, Load Average, Memory Free %
2. Network: Transmit/Receive Bytes per Second, Errors per Second
3. Processes: N/A
4. Storage: Disk Used %, Total Utilization %, Read/Write Bytes per Second

![New Relic Synology Screenshot](/img/newrelic-synology/newrelic_synology_01.png)

## Dashboards
Here are some useful dashboards and their NRQL query:

### CPU %

```
SELECT average(cpuPercent), min(cpuPercent), max(cpuPercent) FROM SystemSample WHERE entityName = 'DATACRUNCH' timeseries
```
<iframe src="https://chart-embed.service.newrelic.com/herald/71376e2e-e9de-4b83-a85a-a30ac631aa1b" title="CPU %" width="100%" height="400"></iframe>

### Average Load (Five Minutes)

```
SELECT average(loadAverageFiveMinute) FROM SystemSample WHERE entityName = 'DATACRUNCH' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/b9c21022-f83e-4c81-9f6e-55be87f1a7c9" title="CPU %" width="100%" height="400"></iframe>

### Memory % Free

```
SELECT average(memoryFreePercent OR memoryFreeBytes/memoryTotalBytes*100) FROM SystemSample WHERE entityName = 'DATACRUNCH' timeseries
```
<iframe src="https://chart-embed.service.newrelic.com/herald/9c044386-a00b-4aa3-845d-7c459cde0ded" title="CPU %" width="100%" height="400"></iframe>

### Disk % Usage

```
SELECT average(diskUsedPercent) FROM StorageSample WHERE entityAndMountPoint in ('3571265904580471743//volume1', '3571265904580471743//volume2') timeseries facet entityAndMountPoint
```
<iframe src="https://chart-embed.service.newrelic.com/herald/95d5e1cd-424b-48e1-93c6-1908901db836" title="CPU %" width="100%" height="400"></iframe>

### Disk I/O

```
SELECT average(readWriteBytesPerSecond OR readBytesPerSecond+writeBytesPerSecond) FROM StorageSample WHERE (entityAndMountPoint in ('3571265904580471743//volume1', '3571265904580471743//volume2')) FACET entityAndMountPoint TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/fe150234-1009-4428-a0ca-b170a8682b87" title="CPU %" width="100%" height="400"></iframe>

### Network I/O

```
SELECT average(transmitBytesPerSecond), average(receiveBytesPerSecond), average(receiveErrorsPerSecond) FROM NetworkSample WHERE entityName = 'DATACRUNCH' timeseries
```
<iframe src="https://chart-embed.service.newrelic.com/herald/5794cdd9-7fb3-436b-8848-fea1478f5b7c" title="CPU %" width="100%" height="400"></iframe>