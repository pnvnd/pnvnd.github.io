---
layout: post
title: "Monitor Synology with New Relic"
subtitle: "Linux (Upstart) Infrastructure Agent Install"
background: '/img/newrelic-synology/newrelic_synology_01.png'
---

# Monitoring Synology DSM Infrastructure
Synology DSM is a custom build of Linux using Upstart.  Follow the [tarball assisted install for the New Relic Infrastructure Agent on Synology](https://docs.newrelic.com/docs/infrastructure/install-infrastructure-agent/linux-installation/tarball-assisted-install-infrastructure-agent-linux) to start monitoring Synology devices without Docker.

In this example, I use a Synology DS214play device (32-bit Intel Processor) with DSM 6.2 installed.

## Summary for Intel Atom Devices
Start by enabling SSH on your Synology device:  
1. Control Panel
2. Terminal & SNMP
3. Terminal > Enable SSH Service > Apply

![Synology Screenshot 1](/img/newrelic-synology/newrelic_synology_00.png)

Next, open your terminal and enter `ssh user@synology`.  You may get an error message saying your home directory does not exist and will start at the root directory `/`.
1. Download New Relic Infrastructure Agent.  In this case, my Synology DS214play uses an Intel Atom CE5335 processor which is 32-bit only (ARM and ARM64 variants of the New Relic Infrastructure agent are also available [here](https://download.newrelic.com/infrastructure_agent/binaries/linux/)):  
`sudo curl https://download.newrelic.com/infrastructure_agent/binaries/linux/386/newrelic-infra_linux_1.20.4_386.tar.gz --output newrelic-infra_linux_1.20.4_386.tar.gz`

2. Extract:  
 `sudo tar -xf newrelic-infra_linux_1.20.4_386.tar.gz`

3. Append your New Relic license key (or edit in `vi`):  
 `echo "license_key=\"a1b2c3d4e5f6g6h7i7j8k9l0m9n8o7p6q5r4NRAL\"" | sudo tee -a /etc/newrelic-infra/config_defaults.sh`

4. Run installer script:  
`sudo /etc/newrelic-infra/installer.sh`

5. Check if service is running:  
`sudo initctl status newrelic-infra`

6. If service is not running, try restarting:  
`sudo initctl restart newrelic-infra`


![Synology Screenshot 2](/img/newrelic-synology/newrelic_synology_02.png)

## Infrastructure Data Captured
1. System
    - CPU %
    - Load Average
    - Memory Free %
2. Network
    - Transmit Bytes per Second
    - Receive Bytes per Second
    - Errors per Second
3. Processes: Not reported by default.
    - To enable, `echo "enable_process_metrics: true" | sudo tee -a /etc/newrelic-infra.yml` and restart service
4. Storage:
    - Disk Used %
    - Total Utilization %
    - Read/Write Bytes per Second
5. Events  

![New Relic Screenshot 1](/img/newrelic-synology/newrelic_synology_03.png)

## Dashboards
Here are some useful dashboards and their NRQL query.
![New Relic Screenshot 2](/img/newrelic-synology/newrelic_synology_01.png)

### CPU %

```
SELECT average(cpuPercent), min(cpuPercent), max(cpuPercent) FROM SystemSample WHERE entityName = 'DATACRUNCH' timeseries
```
<iframe src="https://chart-embed.service.newrelic.com/herald/6402b31c-bbd4-453b-ae65-4b9c72a587c9?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Average Load

```
SELECT average(loadAverageFifteenMinute), average(loadAverageFiveMinute), average(loadAverageOneMinute) FROM SystemSample WHERE entityName = 'DATACRUNCH' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/deff37e0-516e-483d-a753-8c454755f61c?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Memory % Free

```
SELECT average(memoryUsedBytes), average(memoryCachedBytes), average(memorySharedBytes), average(memorySlabBytes) FROM SystemSample WHERE entityName = 'DATACRUNCH' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/eb846c6a-36df-4295-9372-8a34206a17bd?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Disk % Usage

```
SELECT average(diskUsedPercent) FROM StorageSample WHERE entityName = 'DATACRUNCH' FACET entityAndMountPoint TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/95d5e1cd-424b-48e1-93c6-1908901db836" title="CPU %" width="100%" height="400"></iframe><br><br>

### Disk I/O

```
SELECT average(readWriteBytesPerSecond OR readBytesPerSecond+writeBytesPerSecond) FROM StorageSample WHERE hostname = 'DATACRUNCH' FACET entityAndMountPoint TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/f051cb0b-a5a9-45d6-b247-4843e667e37c?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Network I/O

```
SELECT average(transmitBytesPerSecond), average(receiveBytesPerSecond), average(receiveErrorsPerSecond) FROM NetworkSample WHERE entityName = 'DATACRUNCH' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/0724084f-2ed4-4eea-899c-c231c8b53ddc?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Events

```
SELECT changedPath, summary, changeType, source FROM InfrastructureEvent WHERE entityName = 'DATACRUNCH' SINCE 72 HOURS AGO
```
<iframe src="https://chart-embed.service.newrelic.com/herald/a9e99d70-c6f6-46a0-85ca-a0bda3264b00?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

### Processes
```
SELECT max(cpuPercent) FROM ProcessSample WHERE entityName = 'DATACRUNCH' AND processDisplayName != 'newrelic-infra' FACET processDisplayName TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/956167a9-5952-401a-92a5-5a851207b516?height=400px&timepicker=true" title="CPU %" width="100%" height="400"></iframe><br><br>

![New Relic Screenshot 32](/img/newrelic-synology/newrelic_synology_04.png)

## Cost Summary
After enabling the infrastructure agent for 2 days, only 100 MB of data was ingested per day.  This works out to about 3GB per month ingested, far below the 100GB included in the free-tier account.