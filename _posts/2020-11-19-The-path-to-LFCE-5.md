---
layout: post
section-type: post
title: "The path to LFCE Day5 : NTP and Syslog services"
category: 'linux'
tags: [ 'linux', 'lfce' ]
---

## Network Time Protocol

### let's create an NTP server
acheiving that is actually pretty simple as the package for NTP is usually already installed in most linux distributions, we only need to edit the `/etc/chronyd.conf` file and enable clients to sync their clock with us.
in the example below we are only allowing clients in the `172.31.96.0/20` subnet, we can also specify a domain name instead of a subnet address or even allow anyone to sync with us by not spcifying anything.

the example also enable our server to serve time even if it has lost connection to it's time source (an NTP server can also be a client to other NTP servers) for at most 10 seconds.

```
    # Allow NTP client access from local network.
    allow 172.31.96.0/20
    # Serve time even if not synchronized to a time source.
    local stratum 10
```

### now let's configure our client to get time from the server
check the currently used NTP servers `$chronyc sources`
```
    [cloud_user@mhenimerz2c ~]$ chronyc sources -v
    210 Number of sources = 4

    .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
    / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
    | /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
    ||                                                 .- xxxx [ yyyy ] +/- zzzz
    ||      Reachability register (octal) -.           |  xxxx = adjusted offset,
    ||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
    ||                                \     |          |  zzzz = estimated error.
    ||                                 |    |           \
    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    ^? ec2-3-217-79-242.compute>     0  10     0     -     +0ns[   +0ns] +/-    0ns
    ^? 50-205-57-38-static.hfc.>     0  10     0     -     +0ns[   +0ns] +/-    0ns
    ^? ntp1.doctor.com               0  10     0     -     +0ns[   +0ns] +/-    0ns
    ^? palmers.nobody.at             0  10     0     -     +0ns[   +0ns] +/-    0ns
```

to do that remove or comment out (add `#` to the beggining of the line) the lines that define the servers currently used.
and add the line with our previously configured server.

```
    # Use public servers from the pool.ntp.org project.
    # Please consider joining the pool (http://www.pool.ntp.org/join.html).
    #server 0.centos.pool.ntp.org iburst
    #server 1.centos.pool.ntp.org iburst
    #server 2.centos.pool.ntp.org iburst
    #server 3.centos.pool.ntp.org iburst
    server 172.31.100.94
```

check again which server is used
```
    [cloud_user@mhenimerz2c ~]$ chronyc sources -v
    210 Number of sources = 1
    ....

    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    ^* 172.31.100.94                10   9   377   271  +3852ns[+5442ns] +/-   76us

```

### NOTE
it is highly recomended to have multiple NTP servers for high availability as NTP is one the most important protocols in large and production infrastrutures, it is also very critical for logging.

## Creating a central syslog server
### why central logging ?
logging is an essential piece of a production environment especially for security purposes, linux systems by default log system events locally in the `/var/log/` directory.
having logs locally is alright for testing and developpment environments but for a more robust and secure production infrastructure we need to send logs out to a central server (not necessarly one server, there could be multiple replicas of the logs sent out to multiple servers).

### let's get going
to make your host a server by enabling log reception you can either use TCP or UDP and this is acheived by editing the `/etc/rsyslog.conf` file, you also have the option to choose a non default port (default is 514) by changing the port value
```
    # Provides UDP syslog reception
    #$ModLoad imudp
    #$UDPServerRun 514

    # Provides TCP syslog reception
    $ModLoad imtcp
    $InputTCPServerRun 514
```
next we need to specify where the received logs would be stored, we can do that by directly editing `/etc/rsyslog.conf` however for better readability it is recommended to define this configuration in a seperate file and make sure that it is referenced in the main config file.

the line bellow ensures that all config files under the `/etc/rsyslog.d/` directory are appended to the main config file.
```
    # Include all config files in /etc/rsyslog.d/
    $IncludeConfig /etc/rsyslog.d/*.conf
```

to define the storage directory for the received logs we create a new config file `/etc/rsyslog.d/172.31.101.79.conf`
and we specify the action to take when logs are received from a specfic client (this can be changed to match a hostname or more than one client)
this also can be modified to include more complex actions such as creating a new file every day with the date included in the file name
```
    if $fromhost-ip == '172.31.101.79' then{
            action(type="omfile" file="/var/log/mheni-server-2.log")
            stop
    }
```
### what about the client side
configring a client to send its logs to a remote server is also done by editing `/etc/rsyslog.conf` this time we want to specify the ip address (or hostname) of the server.
in the example below, the first section activates queueing logs in case connectivity to the remote syslog server is lost.
the second section defines what and where to send the logs 
- `*.*` means we are sending everything, if we wanted to only send apache logs we would use `httpd.*` instead
- `@@172.31.100.94:514` is the IP address and port of the syslog server

```
    # An on-disk queue is created for this action. If the remote host is
    # down, messages are spooled to disk and sent when it is up again.
    $ActionQueueFileName fwdRule1 # unique name prefix for spool files
    $ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
    $ActionQueueSaveOnShutdown on # save messages to disk on shutdown
    $ActionQueueType LinkedList   # run asynchronously
    $ActionResumeRetryCount -1    # infinite retries if host is down

    # remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
    *.* @@172.31.100.94:514
```
and finally our client is sending his logs to the server, now to test this out we can use the `logger` command ideally you'd have two windows one connected to each server.

on the syslog server use `$ tail -f /var/log/mheni-server-2.log` which is the filename we've specified previously, and on the client use `$ logger this-is-a test-log-message`
```
    #on the client
    [cloud_user@mhenimerz2c ~]$ logger this-is-a-test-log-message

    #on the server
    [root@mhenimerz1c cloud_user]# tail -f /var/log/mheni-server-2.log
    Nov 20 12:33:06 mhenimerz2c NetworkManager[821]: <info>  [1605875586.6589] dhcp6 (ens5):   preferred_lft 150
    Nov 20 12:34:01 mhenimerz2c systemd: Started Session 34 of user root.
    Nov 20 12:34:01 mhenimerz2c CROND[3393]: (root) CMD (/var/awslogs/bin/awslogs-nanny.sh > /dev/null 2>&1)
    Nov 20 12:34:01 mhenimerz2c systemd: Removed slice User Slice of root.
    ....
    Nov 20 12:34:15 mhenimerz2c cloud_user: this-is-a-test-log-message
```

