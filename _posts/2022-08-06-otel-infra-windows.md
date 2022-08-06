---
layout: post
title: "Infrastructure Monitoring with OpenTelemetry"
subtitle: "Example using New Relic"
background: '/img/otel-infra/otel-infra_windows_10.png'
---

# Introduction
We'll be using the OpenTelemetry Collector `otelcol-contrib` to gather host metrics on Windows, Linux, and Mac.  At the time of this writing, we'll be using the latest `otelcol-contrib` package version 0.57.2.

## Windows

The following steps assumes you are running Windows PowerShell as an Administrator.

1. Create a directory somewhere to keep the installation in one place.  
    ```powershell
    mkdir "C:\Program Files\OpenTelemetry"
    ```  
    ![Screenshot](/img/otel-infra/otel-infra_windows_01.png)

1. Change to the newly created directory.  
    ```powershell
    cd "C:\Program Files\OpenTelemetry"
    ```
    ![Screenshot](/img/otel-infra/otel-infra_windows_02.png)

1. Download the `otelcol-contrib` package from GitHub.  
    ```powershell
    (New-Object Net.WebClient).DownloadFile("https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.57.2/otelcol-contrib_0.57.2_windows_amd64.tar.gz", "C:\Program Files\OpenTelemetry\otelcol-contrib_0.57.2_windows_amd64.tar.gz")
    ```
    ![Screenshot](/img/otel-infra/otel-infra_windows_03.png)

1. Extract the files to the current directory, either using the command line or 7zip should work.  
    ```powershell
    tar -xf otelcol-contrib_0.57.2_windows_amd64.tar.gz
    ```
    ![Screenshot](/img/otel-infra/otel-infra_windows_04.png)

1. Download a sample configuration file for the OpenTelemetry Contrib collector from GitHub  
    ```powershell
    (New-Object Net.WebClient).DownloadFile("https://gist.githubusercontent.com/pnvnd/1533b04609fbbe583056afdc31683667/raw/242f9fa682989639a06fc22eb209b8ffd1a4d4c3/otel-config_windows.yaml", "C:\Program Files\OpenTelemetry\otel-config_windows.yaml")
    ```  
   Otherwise, you can create `otel-config_windows.yaml` manually with the following:  
    ```yaml
    extensions:
      health_check:

    receivers:
      hostmetrics:
        collection_interval: 20s
        scrapers:
          cpu:
            metrics:
              system.cpu.utilization:
                enabled: true
          load:
          memory:
            metrics:
              system.memory.utilization:
                enabled: true
          disk:
          filesystem:
            metrics:
              system.filesystem.utilization:
                enabled: true
          network:
          paging:
            metrics:
              system.paging.utilization:
                enabled: true
    #     processes:
          process:
            mute_process_name_error: true

    processors:
      memory_limiter:
        check_interval: 1s
        limit_mib: 1000
        spike_limit_mib: 200
      batch:
      cumulativetodelta:
        include:
          metrics:
            - system.network.io
            - system.disk.operations
            - system.network.dropped
            - system.network.packets
            - process.cpu.time
          match_type: strict
      resource:
        attributes:
          - key: host.id
            from_attribute: host.name
            action: upsert
      resourcedetection:
        detectors: [env, system]

    exporters:
      otlp:
        endpoint: https://otlp.nr-data.net:4317
        headers:
          api-key: YOUR_KEY_HERE

    service:
      pipelines:
        metrics:
          receivers: [hostmetrics]
          processors: [batch, resourcedetection, resource, cumulativetodelta]
          exporters: [otlp]
    ```
    Note: The `processes` is commented out for the Windows configuration, since this is only supported in Linux.

1. Edit the `otel-config_windows.yaml` file and replace `YOUR_KEY_HERE` with your New Relic Ingest - License key ending with NRAL 
   ```powershell
   ((Get-Content -path "C:\Program Files\OpenTelemetry\otel-config_windows.yaml" -Raw) -replace 'YOUR_KEY_HERE','XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXNRAL') | Set-Content -Path "C:\Program Files\OpenTelemetry\otel-config_windows.yaml"
   ```
    ![Screenshot](/img/otel-infra/otel-infra_windows_05.png)

1. Create a Windows service to run the OpenTelemetry Collector in the background  
   ```powershell
   New-Service -Name "otel-infra" -BinaryPathName 'C:\Program Files\OpenTelemetry\otelcol-contrib.exe --config=file:"C:\Program Files\OpenTelemetry\otel-config_windows.yaml"' -DisplayName "OpenTelemetry Infrastructure Agent" -StartupType Manual -Description "OpenTelemetry Infrastructure Agent v0.57.2" -Credential "DESKTOP-IED569R\Peter"
   ```
   ![Screenshot](/img/otel-infra/otel-infra_windows_07.png)

   Note: Running this service as a `Local System` account does not have access to gather storage metrics, so you'll need to create a service account or use your current account  credentials to get these metrics.  
   ![Screenshot](/img/otel-infra/otel-infra_windows_06.png)

1. Start the `otel-infra` service by running `services.msc` or use the command line  
   ```powershell
   net start "otel-infra"
   ```
   ![Screenshot](/img/otel-infra/otel-infra_windows_08.png)
   ![Screenshot](/img/otel-infra/otel-infra_windows_09.png)

1. Log into New Relic and check your Hosts to see metrics.
   ![Screenshot](/img/otel-infra/otel-infra_windows_10.png)
1. Stop the `otel-infra` service with
   ```powershell
   net stop "otel-infra"
   ```
1. If you need to uninstall the service and remove the files, use this command  
   ```powershell
   "sc delete otel-infra" | cmd
   cd ~
   rm -r "C:\Program Files\OpenTelemetry"
   ```
   ![Screenshot](/img/otel-infra/otel-infra_windows_11.png)

## Linux
Similar to the Windows setup, commands will be listed here to do the same thing, but with a Raspberry Pi 4B device.
![Screenshot](/img/otel-infra/otel-infra_linux_00.png)


1. Make a directory somewhere to save the files and configurations
    ```sh
    sudo mkdir /opt/opentelemetry
    ```
1. Change to the newly created directory
    ```sh
    cd /opt/opentelemetry
    ```

1. Download the `otelcol-contrib` package for Linux.  In this example, we're running this on a Raspberry Pi 4B (8GB) with Raspbian (64-Bit), so we're going to get the `arm64` version.
    ```sh
    sudo wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.57.2/otelcol-contrib_0.57.2_linux_arm64.tar.gz
    ```
1. Download a sample configuration file for Linux
    ```sh
    sudo wget https://gist.githubusercontent.com/pnvnd/1533b04609fbbe583056afdc31683667/raw/93c55d4065bc783d3f340931cca91fceacccaae8/otel-config_linux.yaml
    ```
    Otherwise, create `otel-config_linux.yaml` manually with the following
    ```yaml
    extensions:
      health_check:

    receivers:
      hostmetrics:
        collection_interval: 20s
        scrapers:
          cpu:
            metrics:
              system.cpu.utilization:
                enabled: true
          load:
          memory:
            metrics:
              system.memory.utilization:
                enabled: true
          disk:
          filesystem:
            metrics:
              system.filesystem.utilization:
                enabled: true
          network:
          paging:
            metrics:
              system.paging.utilization:
                enabled: true
          processes:
          process:
            mute_process_name_error: true

    processors:
      memory_limiter:
        check_interval: 1s
        limit_mib: 1000
        spike_limit_mib: 200
      batch:
      cumulativetodelta:
        include:
          metrics:
            - system.network.io
            - system.disk.operations
            - system.network.dropped
            - system.network.packets
            - process.cpu.time
          match_type: strict
      resource:
        attributes:
          - key: host.id
            from_attribute: host.name
            action: upsert
      resourcedetection:
        detectors: [env, system]

    exporters:
      otlp:
        endpoint: https://otlp.nr-data.net:4317
        headers:
          api-key: YOUR_KEY_HERE

    service:
      pipelines:
        metrics:
          receivers: [hostmetrics]
          processors: [batch, resourcedetection, resource, cumulativetodelta]
          exporters: [otlp]
    ```

1. Extract the files  
    ```sh
    sudo tar -xf otelcol-contrib_0.57.2_linux_arm64.tar.gz
    ```

1. Check the version of your `otelcol-contrib` package  
    ```sh
    sudo ./otelcol-contrib --version
    ```
1. Edit `otel-config_linux.yaml` and update with your New Relic Ingest - License key  
    ```sh
    sudo nano otel-config_linux.yaml
    ```
    Otherwise run this command  
    ```sh
    sudo sed -i 's/YOUR_KEY_HERE/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXNRAL/' otel-config_linux.yaml
    ```

1. Start your OpenTelemetry Collector in background
    ```sh
    sudo nohup ./otelcol-contrib --config=file:"/home/pi/otel/otel-config_linux.yaml" &
    ```

1. Log into New Relic and check your hosts for metrics
![Screenshot](/img/otel-infra/otel-infra_linux_01.png)

1. To check if you are running the `otelcol-contrib`  run  
    ```sh
    suo ps aux | grep otelcol-contrib
    ```
1. Stop the `otelcol-contrib` service from running using this command  
    ```sh
    sudo pkill otelcol-contrib
    ```

## Mac
TBD

