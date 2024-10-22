Section 1: Prerequisite installation overview
=============================================
This document describes the installation of software prerequisites in support of Hyperledger Fabric 1.4 onto an Ubuntu 16.04.5 LTS Linux on IBM Z instance. This Ubuntu instance is at Linux kernel level *4.4.0-139*.

The necessary software prerequisites to suppor Hyperledger Fabric include:

*	Docker
*	PIP, a python installer program (needed to install Docker Compose)
*	curl (used to download PIP) 
*	Docker Compose
*	Golang compiler
* Node.js and npm

**Note:** The procedures described here to install Node.js and npm and Golang should work on RHEL 7.x and SLES 12.3.x as well. Docker and Docker Compose installation on RHEL and SLES will differ and is not discussed here.

**Note:** Most of the packages installed by *apt* can also be installed with *yum* on RHEL and *zypper* on RHEL but there may be slight variations in the names of packages and these differences are not discussed here.

Section 2: Install prereqs needed by Hyperledger Fabric and Hyperledger Composer
=====================================================================

In this section, you will install the software prerequisites mentioned in the prior overview section. You will install:

*	Docker
*	PIP, a python installer program (needed to install Docker Compose)
*	curl (used to download PIP) 
*	Docker Compose
*	Golang compiler
* Node.js and npm
* The gulp and istanbul npm packages

**Step 2.1:** Log in to your assigned Ubuntu 16.04.4 Linux on IBM Z instance ssh (for MacOS or Linux) or PuTTY (for Windows) using the IP address assigned to your team.  Here is example output using ssh on MacOS::

 Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 Last login: Thu Nov 29 16:07:58 2018 from 192.168.22.64
 bcuser@ubuntu16045:~$

**Step 2.2:** This system is a relatively “clean” Ubuntu 16.04.4 instance- “clean” in the sense that after this instance was created,
no additional software was installed.  You will therefore need to install several software prerequisites.  The first thing you will 
install is Docker. Docker is a software container platform that is an integral part of Hyperledger Fabric.  *Smart contracts* are deployed via an artifact  
known as *chaincode*, and each chaincode runs inside its own Docker container.  In addition, you will run the Hyperledger Fabric processes themselves in Docker 
containers.  You could choose to run the Hyperledger Fabric processes natively on your host operating system, that is, *not* in Docker 
containers, but even if you did this you would still need Docker to run chaincode.  For this lab, you will use Docker containers for *both* the chaincode and the Hyperledger Fabric processes.  

Issue this *which* command, which attempts to find the *docker* executable. The fact that it gives no response other than returning to 
the command prompt indicates that the program *docker* is not found::

 bcuser@ubuntu16045:~$ which docker
 bcuser@ubuntu16045:~$ 

**Note:** Some Linux distributions produce no output if the executable is not found, as in the above example.  Other Linux distributions
may produce an error message stating that the executable is not found.
   
**Step 2.3:** You will be using root authority for several commands throughout this document.  You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  Use the *sudo* command to switch to the root user.  Observe the change in the command prompt after you enter this command::

 bcuser@ubuntu16045:~$ sudo su -
 root@ubuntu16045:~# 

**Step 2.4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. You will be 
using *apt* throughout this lab to install various software packages. The first thing you need to do is install  
Docker Community Edition (CE).  This software is provided by Docker, so the next several steps will add a repository managed by Docker 
to your system’s list of repositories so that you can install Docker CE. Enter this command to update the *apt* package index::

 root@ubuntu16045:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease [109 kB]                    
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease [107 kB]                           
 Get:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease [109 kB]    
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x Packages [664 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main Translation-en [370 kB]  
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe s390x Packages [605 kB] 
 Get:8 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe Translation-en [305 kB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/multiverse s390x Packages [11.0 kB]
 Get:10 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports/main s390x Packages [7280 B]
 Get:11 http://ports.ubuntu.com/ubuntu-ports xenial-security/main s390x Packages [426 kB]   
 Get:12 http://ports.ubuntu.com/ubuntu-ports xenial-security/main Translation-en [256 kB]
 Get:13 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe s390x Packages [336 kB]
 Get:14 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe Translation-en [172 kB]
 Get:15 http://ports.ubuntu.com/ubuntu-ports xenial-security/multiverse s390x Packages [1716 B]
 Get:16 http://ports.ubuntu.com/ubuntu-ports xenial-security/multiverse Translation-en [2676 B]
 Fetched 3481 kB in 0s (3563 kB/s)  

**Step 2.5:** Install packages to allow *apt* to use a repository over HTTPS::

 root@ubuntu16045:~# apt-get install -y apt-transport-https ca-certificates curl software-properties-common
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
  libcurl3-gnutls python3-pycurl python3-software-properties unattended-upgrades xz-utils
 Suggested packages:
   libcurl4-gnutls-dev python-pycurl-doc python3-pycurl-dbg bsd-mailx mail-transport-agent
 The following NEW packages will be installed:
   curl python3-pycurl python3-software-properties software-properties-common unattended-upgrades xz-utils
 The following packages will be upgraded:
   apt-transport-https ca-certificates libcurl3-gnutls
 3 upgraded, 6 newly installed, 0 to remove and 55 not upgraded.
 Need to get 684 kB of archives.
 After this operation, 1552 kB of additional disk space will be used.
 Get:1 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x libcurl3-gnutls s390x 7.47.0-1ubuntu2.12 [175 kB]
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x apt-transport-https s390x 1.2.29ubuntu0.1 [25.0 kB]
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x ca-certificates all 20170717~16.04.2 [167 kB]
 Get:4 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x curl s390x 7.47.0-1ubuntu2.12 [137 kB]
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x python3-pycurl s390x 7.43.0-1ubuntu1 [39.9 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x python3-software-properties all 0.96.20.8 [20.2 kB]
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x software-properties-common all 0.96.20.8 [9440 B]
 Get:8 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x xz-utils s390x 5.1.1alpha+20120614-2ubuntu2 [78.4 kB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x unattended-upgrades all 0.90ubuntu0.10 [32.3 kB]
 Fetched 684 kB in 0s (3541 kB/s)             
 Preconfiguring packages ...
 (Reading database ... 64431 files and directories currently installed.)
 Preparing to unpack .../libcurl3-gnutls_7.47.0-1ubuntu2.12_s390x.deb ...
 Unpacking libcurl3-gnutls:s390x (7.47.0-1ubuntu2.12) over (7.47.0-1ubuntu2.11) ...
 Preparing to unpack .../apt-transport-https_1.2.29ubuntu0.1_s390x.deb ...
 Unpacking apt-transport-https (1.2.29ubuntu0.1) over (1.2.29) ...
 Preparing to unpack .../ca-certificates_20170717~16.04.2_all.deb ...
 Unpacking ca-certificates (20170717~16.04.2) over (20170717~16.04.1) ...
 Selecting previously unselected package curl.
 Preparing to unpack .../curl_7.47.0-1ubuntu2.12_s390x.deb ...
 Unpacking curl (7.47.0-1ubuntu2.12) ...
 Selecting previously unselected package python3-pycurl.
 Preparing to unpack .../python3-pycurl_7.43.0-1ubuntu1_s390x.deb ...
 Unpacking python3-pycurl (7.43.0-1ubuntu1) ...
 Selecting previously unselected package python3-software-properties.
 Preparing to unpack .../python3-software-properties_0.96.20.8_all.deb ...
 Unpacking python3-software-properties (0.96.20.8) ...
 Selecting previously unselected package software-properties-common.
 Preparing to unpack .../software-properties-common_0.96.20.8_all.deb ...
 Unpacking software-properties-common (0.96.20.8) ...
 Selecting previously unselected package xz-utils.
 Preparing to unpack .../xz-utils_5.1.1alpha+20120614-2ubuntu2_s390x.deb ...
 Unpacking xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 Selecting previously unselected package unattended-upgrades.
 Preparing to unpack .../unattended-upgrades_0.90ubuntu0.10_all.deb ...
 Unpacking unattended-upgrades (0.90ubuntu0.10) ...
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for man-db (2.7.5-1) ...
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 Setting up libcurl3-gnutls:s390x (7.47.0-1ubuntu2.12) ...
 Setting up apt-transport-https (1.2.29ubuntu0.1) ...
 Setting up ca-certificates (20170717~16.04.2) ...
 Setting up curl (7.47.0-1ubuntu2.12) ...
 Setting up python3-pycurl (7.43.0-1ubuntu1) ...
 Setting up python3-software-properties (0.96.20.8) ...
 Setting up software-properties-common (0.96.20.8) ...
 Setting up xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
 Setting up unattended-upgrades (0.90ubuntu0.10) ... 

 Creating config file /etc/apt/apt.conf.d/50unattended-upgrades with new version
 Synchronizing state of unattended-upgrades.service with SysV init with /lib/systemd/systemd-sysv-install...
 Executing /lib/systemd/systemd-sysv-install enable unattended-upgrades
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for ca-certificates (20170717~16.04.2) ...
 Updating certificates in /etc/ssl/certs...
 0 added, 0 removed; done.
 Running hooks in /etc/ca-certificates/update.d...
 done.
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 root@ubuntu16045:~# 

**Step 2.6:**  Add Docker’s official GPG key::

 root@ubuntu16045:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16045:~#

**Step 2.7:** Verify that the key fingerprint is *9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88*::
 
 root@ubuntu16045:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22

 root@ubuntu16045:~# 

**Step 2.8:** Enter the following command to add the *stable* repository that is provided by Docker::

 root@ubuntu16045:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16045:~#

**Step 2.9:** Update the *apt* package index again:: 

 root@ubuntu16045:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease                               
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [66.2 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [3880 B]
 Fetched 70.1 kB in 0s (73.6 kB/s)
 Reading package lists... Done
 root@ubuntu16045:~# 

**Step 2.10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16045:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 18.06.3~ce~3-0~ubuntu
   Version table:
      18.06.3~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.2~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.1~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.0~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.03.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.03.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.12.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.12.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.09.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.09.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.2~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
 root@ubuntu16045:~# 

Some key takeaways from the command output:

*	Docker is not currently installed *(Installed: (none))*
*	*18.06.3~ce~3-0~ubuntu* is the candidate version to install- it is the latest version available at the time of the writing of this document
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 2.11:** Enter this *apt-get* command to install Docker CE.  (Enter Y when prompted to continue)::

 root@ubuntu16045:~# apt-get install docker-ce
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch pigz
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki  git-svn
   diffutils-doc
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch pigz
 0 upgraded, 9 newly installed, 0 to remove and 55 not upgraded.
 Need to get 33.8 MB of archives.
 After this operation, 202 MB of additional disk space will be used.
 Do you want to continue? [Y/n]  Y
    .
    .   (remaining output not shown here)
    .

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**Step 2.12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16045:~# which docker
 /usr/bin/docker
 root@ubuntu16045:~#

**Step 2.13:** Enter the *docker version* command and you should see that version *18.03.1-ce* was installed::

 root@ubuntu16045:~# docker version
 Client:
  Version:           18.06.3-ce
  API version:       1.38
  Go version:        go1.10.3
  Git commit:        d7080c1
  Built:             Wed Feb 20 02:27:09 2019
  OS/Arch:           linux/s390x
  Experimental:      false

 Server:
  Engine:
   Version:          18.06.3-ce
   API version:      1.38 (minimum version 1.12)
   Go version:       go1.10.3
   Git commit:       d7080c1
   Built:            Wed Feb 20 02:26:03 2019
   OS/Arch:          linux/s390x
   Experimental:     false
 root@ubuntu16045:~# 

**Step 2.14:** Enter *docker info* to see even more information about your Docker environment::

 root@ubuntu16045:~# docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 18.06.3-ce
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
 runc version: a592beb5bc4c4092b1b1bac971afed27687340c5
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.4.0-139-generic
 Operating System: Ubuntu 16.04.5 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 3.733GiB
 Name: ubuntu16045
 ID: TAPY:UDIZ:CJW6:NTTE:J3LS:SASV:66FK:2WCM:5HP3:6BMP:PMMQ:IR7K
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support
 root@ubuntu16045:~# 

**Step 2.15:** After the Docker installation, non-root users cannot run Docker commands. One way to get around this for a non-root userid is to add that userid to a group named *docker*.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16045:~# usermod -aG docker bcuser
 root@ubuntu16045:~# 
 
**Note:** This method of authorizing a non-root userid to enter Docker commands, while suitable for a controlled sandbox environment, may not be suitable for a production environemnt due to security considerations. 

**Step 2.16:** Exit so that you are no longer running as root::

 root@ubuntu16045:~# exit
 logout
 bcuser@ubuntu16045:~$ 
 
**Step 2.17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this 
change to take effect.  To prove this, before you log out, enter the *docker info* command and you will receive a permissions error::

 bcuser@ubuntu16045:~$ docker info
 Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.38/info: dial unix /var/run/docker.sock: connect: permission denied
 bcuser@ubuntu16045:~$ 

**Step 2.18:** Now log out::

 bcuser@ubuntu16045:~$ exit
 logout
 Connection to 192.168.22.119 closed.
 Barrys-MacBook-Pro:notes silliman$ 

**Step 2.19:** Log in again.  (These instructions show logging in again using *ssh* from a command shell.  If you are using PuTTY you may need to start a new PuTTY session and log in)::

 $ ssh bcuser@192.168.22.119
 Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 New release '18.04.2 LTS' available.
 Run 'do-release-upgrade' to upgrade to it.

 Last login: Sat Mar  9 18:17:32 2019 from 192.168.215.249
 bcuser@ubuntu16045:~$

**Step 2.20:** Now try *docker info* and this time it should work from your non-root userid::

 bcuser@ubuntu16045:~$ docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 18.06.3-ce
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
 runc version: a592beb5bc4c4092b1b1bac971afed27687340c5
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.4.0-139-generic
 Operating System: Ubuntu 16.04.5 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 3.733GiB
 Name: ubuntu16045
 ID: TAPY:UDIZ:CJW6:NTTE:J3LS:SASV:66FK:2WCM:5HP3:6BMP:PMMQ:IR7K
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support
 bcuser@ubuntu16045:~$ 

**Step 2.21:** You will need to get right back in as root to install *Docker Compose*.  Docker Compose is a tool provided by Docker to 
help make it easier to run an application that consists of multiple Docker containers.  On some platforms, it is installed along with 
the Docker package but on Linux on IBM Z it is installed separately.  It is written in Python and you will install it with a tool 
called Pip.  But first you will install Pip itself!  You will do this as root, so enter this again::

 bcuser@ubuntu16045:~$ sudo su -
 root@ubuntu16045:~#

**Step 2.22:** Install the *python-pip* package which will provide a tool named *Pip* which is used to install Python packages from a public repository::

 root@ubuntu16045:~# apt-get -y install python-pip

This will bring in a lot of prerequisites and will produce a lot of output which is not shown here.

**Step 2.23:** Run this command just to verify that *docker-compose* is not currently available on the system::

 root@ubuntu16045:~# which docker-compose
 root@ubuntu16045:~# 

**Step 2.24:** Use Pip to install Docker Compose::

 root@ubuntu16045:~# pip install docker-compose
 
**Step 2.25:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel pretty good about the output from this command::

 root@ubuntu16045:~# docker-compose --version
 docker-compose version 1.23.2, build 1110ad0
 root@ubuntu16045:~# 

**Note:** If the version of Docker Compose shown in your output differs from what is shown here, that's okay, as long as it is at least *1.14.x*.

**Step 2.26:** Leave root behind and become a normal user again::

 root@ubuntu16045:~# exit
 logout
 bcuser@ubuntu16045:~$

**Step 2.27:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, 
check for the version again now that you are no longer root::

 bcuser@ubuntu16045:~$ docker-compose --version
 docker-compose version 1.23.2, build 1110ad0
 bcuser@ubuntu16045:~$ 

**Step 2.28:** The next thing you are going to install is the *Golang* programming language. You are going to install Golang version 
1.11.1.  Go to the /tmp directory::

 bcuser@ubuntu16045:~$ cd /tmp
 bcuser@ubuntu16045:/tmp$ 

**Step 2.29:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  And now is a good time to tell you 
that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::

 bcuser@ubuntu16045:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.11.1.linux-s390x.tar.gz
 --2019-03-09 18:57:45--  https://storage.googleapis.com/golang/go1.11.1.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.164.176, 2607:f8b0:4004:815::2010
 Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.164.176|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 100474359 (96M) [application/octet-stream]
 Saving to: 'go1.11.1.linux-s390x.tar.gz'

 go1.11.1.linux-s390x.tar.gz         100%[================================================================>]  95.82M  56.3MB/s    in 1.7s    

 2019-03-09 18:57:47 (56.3 MB/s) - 'go1.11.1.linux-s390x.tar.gz' saved [100474359/100474359]

 bcuser@ubuntu16045:/tmp$ 

**Step 2.30:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16045:/tmp$ tar -xvf go1.11.1.linux-s390x.tar.gz
   .
   .  (output not shown here)
   .

**Step 2.31:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a 
single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   Enter this command::

 bcuser@ubuntu16045:/tmp$ sudo mv -iv go /opt 
 'go' -> '/opt/go'
 bcuser@ubuntu16045:/tmp$ 

**Step 2.32:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16045:/tmp$ env | grep GO
 bcuser@ubuntu16045:/tmp$
 
That command, *grep*, is looking for any lines of input that contain the characters *GO*.  Its input is the output of the previous *env*
command, which prints all of your environment variables. Right now you should not see any output.

**Step 2.33:**  You will set these values now.  You will make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16045:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see
 bcuser@ubuntu16045:~$ 

**Step 2.34:** Enter the *cp* command to make a backup copy of *.bashrc* to allow a recovery in the infinitesimally slim chance that you make a mistake in the subsequent five steps which will append information to *.bashrc*.  I know you wouldn't ever make a mistake, but not everyone else is as sharp as you, right? Enter this::

 bcuser@ubuntu16045:~$ cp -ipv .bashrc .bashrc_orig
 '.bashrc' -> '.bashrc_orig'
 bcuser@ubuntu16045:~$

**Step 2.35:** The next five steps- *Steps 2.35 through 2.39* - are each *echo* commands which will append to the end of *.bashrc*.  The first and last of these steps just adds a blank line for readability.  Enter these exactly as shown in each step.  It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you 
enter them.  This appends the arguments of the *echo* commands to the end of the *.bashrc* file.  If you only enter one ‘>’ sign, you 
will overwrite the file’s contents.  I’d rather you not do that. Although *Step 2.34* does create a backup copy of the file,
just in case.  So first, add a blank line::

 bcuser@ubuntu16045:~$ echo '' >> .bashrc   # that is two single quotes, not one double-quote
 bcuser@ubuntu16045:~$ 

**Step 2.36:** Add this line to set your *GOPATH* environment variable::

 bcuser@ubuntu16045:~$ echo export GOPATH=/home/bcuser/git >> .bashrc
 bcuser@ubuntu16045:~$ 

**Step 2.37:** Add this line to set your *GOROOT* environment variable::

 bcuser@ubuntu16045:~$ echo export GOROOT=/opt/go >> .bashrc
 bcuser@ubuntu16045:~$
 
**Step 2.38:** Add this line to update your *PATH* environment variable::

 bcuser@ubuntu16045:~$ echo export PATH=/opt/go/bin:/home/bcuser/bin:\$PATH >> .bashrc
 bcuser@ubuntu16045:~$ 
 
**Step 2.39:** Finally, add another blank line for readability::

 bcuser@ubuntu16045:~$ echo '' >> .bashrc   
 bcuser@ubuntu16045:~$ 
 
**Step 2.40:** Let’s see how you did.  Enter this command::

 bcuser@ubuntu16045:~$ head .bashrc
 # ~/.bashrc: executed by bash(1) for non-login shells.
 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
 # for examples

 # If not running interactively, don't do anything
 case $- in
     *i*) ;;
       *) return;;
 esac

 bcuser@ubuntu16045:~$ 

If your output looked like the above, congratulations, you did not stomp all over your file. *head* prints the top of the file.  Had 
you made and mistake and used a single '>' instead two ‘>>’ like I told you, you would have whacked this stuff.  Your stuff is at the bottom.  If *head* 
prints the top of the file, guess what command prints the bottom of the file.

**Step 2.41:** Try this::

 bcuser@ubuntu16045:~$ tail -5 .bashrc

 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:/home/bcuser/bin:$PATH

 bcuser@ubuntu16045:~$ 

**Step 2.42:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16045:~$ source .bashrc
 bcuser@ubuntu16045:~$ 

**Step 2.43:** Try this to see if your changes took::

 bcuser@ubuntu16045:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git
 bcuser@ubuntu16045:~$ 

**Step 2.44:**  Then try this::

 bcuser@ubuntu16045:~$ go version
 go version go1.11.1 linux/s390x
 bcuser@ubuntu16045:~$ 

**Step 2.45:** Now you will install and configure Node.js, which also includes a program called *npm*, which is the de facto Node.js package manager.  Change to the */tmp* directory::

 bcuser@ubuntu16045:~$ cd /tmp
 bcuser@ubuntu16045:/tmp$ 

**Step 2.46:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16045:/tmp$ wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-s390x.tar.xz
 --2019-03-09 19:07:33--  https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.22.46, 104.20.23.46, 2606:4700:10::6814:172e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.22.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10939304 (10M) [application/x-xz]
 Saving to: 'node-v8.11.3-linux-s390x.tar.xz'

 node-v8.11.3-linux-s390x.tar.xz     100%[================================================================>]  10.43M  --.-KB/s    in 0.1s    

 2019-03-09 19:07:34 (83.1 MB/s) - 'node-v8.11.3-linux-s390x.tar.xz' saved [10939304/10939304]

 bcuser@ubuntu16045:/tmp$

**Step 2.47:** Extract the package underneath your home directory, */home/bcuser*. This will cause the executables to wind up in */home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16045:~$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.11.3-linux-s390x.tar.xz
 bcuser@ubuntu16045:~$ 

**Step 2.48:** Issue this command to see where *node* resides within your path::

 bcuser@ubuntu16045:~$ which node
 /home/bcuser/bin/node
 bcuser@ubuntu16045:~$ 
 
**Step 2.49:** Issue this command to see where *npm* resides within your path::
 
 bcuser@ubuntu16045:~$ which npm
 /home/bcuser/bin/npm
 bcuser@ubuntu16045:~$
 
**Step 2.50:** Issue this command to see which version of *node* is installed::

 bcuser@ubuntu16045:~$ node --version
 v8.11.3
 bcuser@ubuntu16045:~$ 
 
**Step 2.51:** Issue this command to see which version of *npm* is installed::
 
 bcuser@ubuntu16045:~$ npm --version
 5.6.0
 bcuser@ubuntu16045:~$ 

**Recap:** Here is a summary of the major tasks you performed with the help of this document:

*	You installed Docker and added *bcuser* to the *docker* group so that *bcuser* can issue Docker commands
*	You installed Docker Compose (and Pip, which was needed to install it)
*	You installed Go
*	You updated your *.bashrc* profile to make necessary environment changes
*	You installed Node.js and npm

*** End of Document ***
