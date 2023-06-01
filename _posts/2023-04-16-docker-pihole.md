---
layout: post
title: "Running Pihole with Docker"
subtitle: "...and automating updates with cron"
background: '/img/docker-pihole/pihole.png'
---

# Introduction
Pihole runs great on a Raspberry Pi, but what about other devices?  If you have a server lying around that can run Docker, Pihole will work.

## Why?
Pihole was created to run on Rasbperry Pi devices, but this shouldn't be limited to device.  There is an official Docker image that can be used on any device that can run Docker.  The only caveat is that you'll need to manually update the container when a new version comes out.  Luckily, the following guide will show you how to do it with a script.

## Instructions

Make sure you have a device capabile of running Docker first.

1. Start by pulling the latest `pihole` image.
   ```
   docker pull pihole/pihole
   ```
2. If you already have `pihole` running, stop it, and remove the container
   ```
   docker stop pihole
   docker rm -f pihole
   ```
3. We're going to create two files on the device running docker so we can create mount points later: `docker-compose.yml` and `resolv.conf`.
   ```
   sudo mkdir /opt/pihole
   sudo touch /opt/pihole/docker-compose.yml
   sudo touch /opt/pihole/resolv.conf
   cd /opt/pihole
   ```
4. The `docker-compose.yml` file should look something like the following.  This is what you'll be running each time you get a new container `pihole` with updates.
   ```
   version: "3"

   # https://github.com/pi-hole/docker-pi-hole/blob/master/README.md

   services:
   pihole:
       container_name: pihole
       image: pihole/pihole:latest
       # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
       ports:
       - "53:53/tcp"
       - "53:53/udp"
       - "67:67/udp"
       - "80:80/tcp"
       environment:
       TZ: 'America/Toronto'
       # WEBPASSWORD: 'set a secure password here or it will be random'
       # Volumes store your data between container upgrades
       volumes:
       - './etc-pihole:/etc/pihole'
       - './etc-dnsmasq.d:/etc/dnsmasq.d'
       - './resolv.conf:/etc/resolv.conf'
       #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
       cap_add:
       - NET_ADMIN
       restart: unless-stopped # Recommended but not required (DHCP needs NET_ADMIN)
    ```
5. The `resolv.conf` should look like the following.  This is required so you can update your lists in `pihole` without issues.
   ```
   search home
   nameserver 127.0.0.1
   options ndots:0
   ```

6. With the two files in place, simply run this command:
   ```
   docker-compose up -d
   ```
7. Once `pihole` is up and running, check your device's IP address and go to `http://<device-ip>/admin` to login with what ever password you've set.  If it's working, remove any old `pihole` images.
   ```
   docker image prune -f
   ```

## Summary
Because we're mounting files to the Docker container, we can pull the latest `pihole` image and spin up new containers with the `docker-compose.yml` file that mounts the persistant files back in place.  Because the `pihole` images get updated every month, you can automate this task with a shell script.  Create `update_pihole.sh` and add the following:
```
#! /bin/bash
docker pull pihole/pihole
docker stop pihole
docker rm -f pihole
docker-compose -f /opt/pihole/docker-compose.yml up -d
docker image prune -f
```

To automate this further, you can create a `cronjob` in Linux by doing the following:

1. Update your permissions to allow your script to execute
   ```
   chmod +x ~/update_pihole.sh
   ```
2. Start by entering `crontab -e` to edit our cronjobs.  (I used `/bin/nano` but you can use whatever you want.)
3. At the bottom of the file, paste this command.  This runs the `update_pihole.sh` script on the first of each month at midnight.
   ```
   0 0 1 * * /opt/pihole/update_pihole.sh
   ```
4. Save the file and exit.  You should see `crontab: installing new crontab`.
5. To check your cronjob:
   ```
   crontab -l
   ```
6. Check to make sure the cron service is running:
   ```
   service cron status
   ```
7. If `cron` is not running, start the service:
   ```
   sudo cron start
   ```

## Reference
1. https://github.com/pi-hole/pi-hole#method-3-using-docker-to-deploy-pi-hole
2. https://github.com/pi-hole/docker-pi-hole#quick-start
3. https://hub.docker.com/r/pihole/pihole/tags
