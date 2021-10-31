---
layout: post
title: "Monitor Raspberry Pi OS with New Relic"
subtitle: "Linux (systemd) Infrastructure Agent Install"
background: '/img/newrelic_raspberrypi4/nr_rpi4_04.png'
---

# Monitoring Raspberry Pi OS Infrastructure
Raspberry Pi OS custom build of Debian (Linux).  Follow the [tarball assisted install for the New Relic Infrastructure Agent on Synology](https://docs.newrelic.com/docs/infrastructure/install-infrastructure-agent/linux-installation/tarball-assisted-install-infrastructure-agent-linux) to start monitoring Raspberry Pi devices without Docker.

In this example, I use a Raspberry Pi 4B (8GB) device with Rasbberry Pi OS Lite (32-bit) installed.
![Raspberry Pi 4 Screenshot 1](/img/newrelic_raspberrypi4/nr_rpi4_00.png)


## Summary for Raspberry Pi Devices
Start by enabling SSH on your Raspberry Pi device:  
1. `sudo raspi-config`
2. Select `Interface Options` and press `ENTER`
![Raspberry Pi 4 Screenshot 2](/img/newrelic_raspberrypi4/nr_rpi4_00a.png)
3. Select `P2 SSH` amd press `ENTER`
![Raspberry Pi 4 Screenshot 3](/img/newrelic_raspberrypi4/nr_rpi4_00b.png)
4. Selecy `Yes` and press `ENTER`
![Raspberry Pi 4 Screenshot 4](/img/newrelic_raspberrypi4/nr_rpi4_00c.png)

Next, open your terminal and enter `ssh pi@raspberrypi` and your password.
1. Download New Relic Infrastructure Agent. In this case, my Raspberry Pi 4B uses an ARM Cortex-A72 processor, so we'll need to download the ARM agent (not AMR64 agent, since our operating system is 32-bit).  If you downloaded the ARM64 infrastructure agent and installed that instead, the service fails to start.  (ARM and ARM64 variants of the New Relic Infrastructure agent are also available [here](https://download.newrelic.com/infrastructure_agent/binaries/linux/)):  
`sudo curl https://download.newrelic.com/infrastructure_agent/binaries/linux/arm/newrelic-infra_linux_1.20.6_arm.tar.gz --output newrelic-infra_linux_1.20.6_arm.tar.gz `

2. Extract:  
 `sudo tar -xf newrelic-infra_linux_1.20.6_arm.tar.gz`

3. Append your New Relic license key (or edit in `vi`, `nano`, etc.):  
 `echo "license_key=\"a1b2c3d4e5f6g6h7i7j8k9l0m9n8o7p6q5r4NRAL\"" | sudo tee -a ~/newrelic-infra/config_defaults.sh`

4. Run installer script:  
`sudo ~/newrelic-infra/installer.sh`

5. Check if service is running:  
`sudo systemctl status newrelic-infra`

6. If service is not running, try restarting:  
`sudo systemctl restart newrelic-infra`


![Raspberry Pi 4 Screenshot 5](/img/newrelic_raspberrypi4/nr_rpi4_02.png)

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

![Raspberry Pi 4 Screenshot 6](/img/newrelic_raspberrypi4/nr_rpi4_03.png)

### Limit Infrastructure Processes
You may want to configure how frequently samples are taken for infrastructure processes, or limit processors with 0 memory to lower data ingests.  Check the New Relic documentation for the default sample rates.  The snippit below sets process samples to every 60 seconds.
```
echo "disable_zero_mem_process_filter: true" | sudo tee -a /etc/newrelic-infra.yml
echo "metrics_system_sample_rate: 60" | sudo tee -a /etc/newrelic-infra.yml
echo "metrics_network_sample_rate: 60" | sudo tee -a /etc/newrelic-infra.yml
echo "metrics_process_sample_rate: 60" | sudo tee -a /etc/newrelic-infra.yml
echo "metrics_storage_sample_rate: 60" | sudo tee -a /etc/newrelic-infra.yml
```

## Dashboards

### CPU %
Ideally, CPU% min, max, and average should be more or less the same to indicate balanced load.

```
SELECT average(cpuPercent), min(cpuPercent), max(cpuPercent) FROM SystemSample WHERE entityName = 'raspberrypi' timeseries
```
<iframe src="https://chart-embed.service.newrelic.com/herald/8a143467-ffe3-4bfe-8bbd-5cd109aabd98" title="CPU %" width="100%" height="400"></iframe><br><br>

### Average Load (Five Minutes)
The average load refers to the average number of system processes, threads, or tasks that are waiting and ready for CPU time, in the last 1, 5, 15 minutes.

```
SELECT average(loadAverageFifteenMinute), average(loadAverageFiveMinute), average(loadAverageOneMinute) FROM SystemSample WHERE entityName = 'raspberrypi' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/93d84f5e-403f-4f17-af14-2f7a1b4d449f" title="Average Load" width="100%" height="400"></iframe><br><br>

### Memory % Free

```
SELECT average(memoryUsedBytes), average(memoryCachedBytes), average(memorySharedBytes), average(memorySlabBytes) FROM SystemSample WHERE entityName = 'raspberrypi' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/b67e5fe2-42d8-4cf1-8bd3-a081d21cb52f" title="Memory %" width="100%" height="400"></iframe><br><br>

### Disk % Usage

```
SELECT average(diskUsedPercent) FROM StorageSample WHERE entityName = 'raspberrypi' FACET entityAndMountPoint TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/ff9e11ea-9df7-4eda-a590-ad1060e5bed4" title="Disk %" width="100%" height="400"></iframe><br><br>

### Disk I/O

```
SELECT average(readWriteBytesPerSecond OR readBytesPerSecond+writeBytesPerSecond) FROM StorageSample WHERE hostname = 'raspberrypi' FACET entityAndMountPoint TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/38abbfed-6ceb-44d8-a86f-43b7f5ba54af" title="Disk I/O" width="100%" height="400"></iframe><br><br>

### Network I/O

```
SELECT average(transmitBytesPerSecond), average(receiveBytesPerSecond), average(receiveErrorsPerSecond) FROM NetworkSample WHERE entityName = 'raspberrypi' TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/5794cdd9-7fb3-436b-8848-fea1478f5b7c" title="Network I/O" width="100%" height="400"></iframe><br><br>

### Events

```
SELECT changedPath, summary, changeType, source FROM InfrastructureEvent WHERE entityName = 'raspberrypi' SINCE 72 HOURS AGO
```
<iframe src="https://chart-embed.service.newrelic.com/herald/71bbde4d-5c18-4f4a-8493-ab95d2fc92b4" title="Event List" width="100%" height="400"></iframe><br><br>

### Processes
```
SELECT max(cpuPercent) FROM ProcessSample WHERE entityName = 'raspberrypi' AND processDisplayName != 'newrelic-infra' FACET processDisplayName TIMESERIES
```
<iframe src="https://chart-embed.service.newrelic.com/herald/585a6bbe-998d-4004-9319-16c952f04d42" title="Process CPU %" width="100%" height="400"></iframe><br><br>

![Raspberry Pi 4 Screenshot 6](/img/newrelic_raspberrypi4/nr_rpi4_05.png)

Otherwise, import this dashboard to your New Relic One account without creating these manually:  
<script src="https://gist.github.com/pnvnd/388a2cbb0a0474cf56aeb2a70a20b12b.js"></script>