---
layout: post
section-type: post
title: "The path to LFCE Day2 : Running commands on multiple hosts simultanousely"
category: [ 'linux' , 'lfce' ]
tags: [ 'linux', 'lfce' ]
---

## Running commands on multiple hosts simultanousely

### what's the usecase for this?
The diff command enables us to compare two files, the output of the example below essentially tells us that the first line is the same in both files and that file2 has  two additional lines.

### how do we do it?

first off the pssh package is not included in the default packages, which means we need to add the `epel-release` package to our system's repo
    [cloud_user@ip-10-0-1-31 ~]$ sudo yum install epel-release
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
    * base: d36uatko69830t.cloudfront.net
    * extras: d36uatko69830t.cloudfront.net
    * updates: d36uatko69830t.cloudfront.net
    Resolving Dependencies
    --> Running transaction check
    ---> Package epel-release.noarch 0:7-11 will be installed
    --> Finished Dependency Resolution

    Dependencies Resolved
    ...

    Installing : epel-release-7-11.noarch                                                                                                                         1/1
    Verifying  : epel-release-7-11.noarch                                                                                                                         1/1

    Installed:
    epel-release.noarch 0:7-11

    Complete!

next we will install the `pssh` package

    [cloud_user@ip-10-0-1-31 ~]$ sudo yum install pssh -y
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
    epel/x86_64/metalink                                                                                                                         |  17 kB  00:00:00
    ...

    Install  1 Package

    ...

    Installing : pssh-2.3.1-5.el7.noarch                                                                                                                          1/1
    Verifying  : pssh-2.3.1-5.el7.noarch                                                                                                                          1/1

    Installed:
    pssh.noarch 0:2.3.1-5.el7

    Complete!

now we need some remote hosts to run commands on, as i'm writing this i don't have access to additional servers so i will use 3 docker containers running the ubuntu image, you can use the `pssh` command on your servers if you have them or use containers for practicing.

install docker
    [cloud_user@ip-10-0-1-31 ~]$ sudo yum install docker -y
    Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-manager

    Complete!
enable and start the docker service

    [cloud_user@ip-10-0-1-31 ~]$ sudo systemctl enable docker
    Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
    [cloud_user@ip-10-0-1-31 ~]$ sudo systemctl start docker
    [cloud_user@ip-10-0-1-31 ~]$ systemctl status docker
    ● docker.service - Docker Application Container Engine
    Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
    Active: active (running) since Fri 2020-07-31 09:52:59 EDT; 1min 58s ago
        Docs: http://docs.docker.com
    Main PID: 1865 (dockerd-current)
    CGroup: /system.slice/docker.service
            ├─1865 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgro...
            └─1870 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --sta...

start 3 containers 


    [cloud_user@ip-10-0-1-31 ~]$ sudo docker run -dt --name host01 rastasheep/ubuntu-sshd
    Unable to find image 'rastasheep/ubuntu-sshd:latest' locally
    Trying to pull repository docker.io/rastasheep/ubuntu-sshd ...
    latest: Pulling from docker.io/rastasheep/ubuntu-sshd
    ...
    0ae38e059780: Pull complete
    ca79c723275f: Pull complete
    Digest: sha256:1a4010f95f6b3292f95fb26e442f85885d523f9a0bb82027b718df62fdd0d9e9
    Status: Downloaded newer image for docker.io/rastasheep/ubuntu-sshd:latest
    0e20b1972dde680271b898f519136b4a7da92c28c0fa1c65c428e9d547ca4b7a
    
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker run -dt --name host02 rastasheep/ubuntu-sshd
    3eaafe2c7526811015a895c415b11f10c1bb9e1d62e58f2bb19d227f7d0eee5e
    
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker run -dt --name host03 rastasheep/ubuntu-sshd
    68090f1939562925d70be46b43ba863d9ac514483c5bb6068ba3990757ff4612


check status
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker container ls -a
    CONTAINER ID        IMAGE                    COMMAND               CREATED             STATUS              PORTS               NAMES
    68090f193956        rastasheep/ubuntu-sshd   "/usr/sbin/sshd -D"   20 seconds ago      Up 19 seconds       22/tcp              host03
    3eaafe2c7526        rastasheep/ubuntu-sshd   "/usr/sbin/sshd -D"   29 seconds ago      Up 28 seconds       22/tcp              host02
    0e20b1972dde        rastasheep/ubuntu-sshd   "/usr/sbin/sshd -D"   38 seconds ago      Up 37 seconds       22/tcp              host01


get the ip addresses for the containers and write them to a file.
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' host01 > hosts_for_pssh
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' host02 >> hosts_for_pssh
    [cloud_user@ip-10-0-1-31 ~]$ sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' host03 >> hosts_for_pssh
    
    [cloud_user@ip-10-0-1-31 ~]$ cat hosts_for_pssh
    172.17.0.2
    172.17.0.3
    172.17.0.4

make sure you have the fingerprint of all servers and that they have the same username and password.
    [cloud_user@ip-10-0-1-31 ~]$ ssh root@172.17.0.3
    The authenticity of host '172.17.0.3 (172.17.0.3)' can't be established.
    ECDSA key fingerprint is SHA256:YtTfuoRRR5qStSVA5UuznGamA/dvf+djbIT6Y48IYD0.
    ECDSA key fingerprint is MD5:43:3f:41:e9:89:45:06:6f:f6:42:c4:6a:70:37:f8:1d.
    Are you sure you want to continue connecting (yes/no)? yes

    [cloud_user@ip-10-0-1-31 ~]$ ssh root@172.17.0.4
    The authenticity of host '172.17.0.4 (172.17.0.4)' can't be established.
    ECDSA key fingerprint is SHA256:YtTfuoRRR5qStSVA5UuznGamA/dvf+djbIT6Y48IYD0.
    ECDSA key fingerprint is MD5:43:3f:41:e9:89:45:06:6f:f6:42:c4:6a:70:37:f8:1d.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '172.17.0.4' (ECDSA) to the list of known hosts.
    root@172.17.0.4's password:

add username to the hosts file, [see previous](http://merzouki.com/linux/lfce/2020/07/30/The-path-to-LFCE-Day2.html) post about `sed`
    [cloud_user@ip-10-0-1-31 ~]$ sed -i 's/^/root@/' hosts_for_pssh
    [cloud_user@ip-10-0-1-31 ~]$ cat hosts_for_pssh
    root@172.17.0.2
    root@172.17.0.3
    root@172.17.0.4

and now we can the date command to test, use `-A` to force pssh to ask for a password which should be the same on all hosts (this not very secure in producion we would use ssh keys instead of passwords)

    [cloud_user@ip-10-0-1-31 ~]$ pssh -h hosts_for_pssh -A -i "date"
    Warning: do not enter your password if anyone else has superuser
    privileges or access to your account.
    Password:
    [1] 10:37:48 [SUCCESS] root@172.17.0.2
    Fri Jul 31 14:37:48 UTC 2020
    [2] 10:37:48 [SUCCESS] root@172.17.0.3
    Fri Jul 31 14:37:48 UTC 2020
    [3] 10:37:49 [SUCCESS] root@172.17.0.4
    Fri Jul 31 14:37:49 UTC 2020
