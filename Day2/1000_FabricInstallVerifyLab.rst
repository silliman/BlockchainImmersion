Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.4 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  During the installation process for this instance, no extra packages were selected for installation beyond the minimum offered by the installer.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.4 LTS* and the Linux kernel is at level *4.4.0-112*.

You will install the necessary software prerequisites to build and test a Hyperledger Fabric network, including

*	Docker and Docker Compose
*	Node.js and npm
*	Golang compiler
*	other necessary packages
During the lab, you will download three Hyperledger Fabric source code repositories, build the necessary artifacts from the source code, and run two comprehensive end-to-end tests to verify that your Hyperledger Fabric installation is in working order.

You will first download the *fabric* repository, which contains the source code for the core functionality of Hyperledger Fabric.  Using this source code, you will build the Docker images that comprise the core Hyperledger Fabric services. Then you will run an end-to-end test that utilizes the Hyperledger Fabric Command Line Interface (CLI) to verify that everything is working correctly.

Then you will download two more Hyperledger Fabric source code repositories, *fabric-ca* and *fabric-sdk-node*, and build from this source the artifacts necessary to run a second, more comprehensive end-to-end test of the Fabric Node.js SDK.

 
Section 2: Install prerequisite software needed by Hyperledger Fabric
=====================================================================

In this section, you will install a subset of the software prerequisites mentioned in the Lab Overview section-  the major software prerequisites that will be necessary to run the end-to-end command line interface (CLI) test in Section 3. You will install:

*	Docker
*	PIP, a python installer program (needed to install Docker Compose)
*	curl (used to download PIP) 
*	Docker Compose
*	Golang compiler
**Step 2.1:** Log in to your assigned Ubuntu 16.04.4 Linux on IBM Z instance using the PuTTY icon on your workstation using the IP address assigned to your team.  You will be greeted with a message like this::

  Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-112-generic s390x)

   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  bcuser@ubuntu16044:~$ 

**Step 2.2:** This system is a relatively “clean” Ubuntu 16.04.4 instance- “clean” in the sense that after this instance was created,
no additional software was installed.  You will therefore need to install several software prerequisites.  The first thing you will 
install is Docker. Docker is a software container platform that is an integral part of Hyperledger Fabric.  *Smart contracts*, also 
known as *chaincode*, run as Docker containers.  In addition, you will run the Hyperledger Fabric processes themselves in Docker 
containers.  You could choose to run the Hyperledger Fabric processes natively on your host operating system, that is, *not* in Docker 
containers, but even if you did this you would still need Docker to run chaincode.  For this lab, you will use Docker containers for *both* the chaincode and the Hyperledger Fabric processes.  

Issue this *which* command, which attempts to find the *docker* executable. The fact that it gives no response other than returning to 
the command prompt indicates that the program *docker* is not found::

  bcuser@ubuntu16044:~$ which docker
  bcuser@ubuntu16044:~$ 

**Note:** Some Linux distributions produce no output if the executable is not found, as in the above example.  Other Linux distributions
may produce an error message stating that the executable is not found.
   
**Step 2.3:** You will be using root authority for several commands throughout this lab.  You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  Use the *sudo* command to switch to the root user.  Observe the change in the command prompt after you enter this command::

 bcuser@ubuntu16044:~$ sudo su -
 root@ubuntu16044:~# 

**Step 2.4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. You will be 
using *apt* throughout this lab to install various software packages. The first thing you need to do is install the 
Docker Community Edition (CE).  This software is provided by Docker, so the next several steps will add a repository managed by Docker 
to your system’s list of repositories so that you can install Docker CE. Enter this command to update the *apt* package index::

 root@ubuntu16044:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease         
 Reading package lists... Done     
 
**Step 2.5:** Install packages to allow *apt* to use a repository over HTTPS::

 root@ubuntu16044:~# apt-get install -y apt-transport-https ca-certificates curl software-properties-common
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 apt-transport-https is already the newest version (1.2.26).
 ca-certificates is already the newest version (20170717~16.04.1).
 The following additional packages will be installed:
   python3-pycurl python3-software-properties unattended-upgrades xz-utils
 Suggested packages:
   libcurl4-gnutls-dev python-pycurl-doc python3-pycurl-dbg bsd-mailx mail-transport-agent
 The following NEW packages will be installed:
   curl python3-pycurl python3-software-properties software-properties-common unattended-upgrades xz-utils
 0 upgraded, 6 newly installed, 0 to remove and 3 not upgraded.
 Need to get 317 kB of archives.
 After this operation, 1552 kB of additional disk space will be used.
 Get:1 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x curl s390x 7.47.0-1ubuntu2.7 [137 kB]
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x python3-pycurl s390x 7.43.0-1ubuntu1 [39.9 kB]
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x python3-software-properties all 0.96.20.7 [20.3 kB]
 Get:4 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x software-properties-common all 0.96.20.7 [9452 B]
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x xz-utils s390x 5.1.1alpha+20120614-2ubuntu2 [78.4 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x unattended-upgrades all 0.90ubuntu0.9 [32.3 kB]
 Fetched 317 kB in 0s (1572 kB/s)               
 Preconfiguring packages ...
 Selecting previously unselected package curl.
 (Reading database ... 64250 files and directories currently installed.)
 Preparing to unpack .../curl_7.47.0-1ubuntu2.7_s390x.deb ...
 Unpacking curl (7.47.0-1ubuntu2.7) ...
 Selecting previously unselected package python3-pycurl.
 Preparing to unpack .../python3-pycurl_7.43.0-1ubuntu1_s390x.deb ...
 Unpacking python3-pycurl (7.43.0-1ubuntu1) ...
 Selecting previously unselected package python3-software-properties.
 Preparing to unpack .../python3-software-properties_0.96.20.7_all.deb ...
 Unpacking python3-software-properties (0.96.20.7) ...
 Selecting previously unselected package software-properties-common.
 Preparing to unpack .../software-properties-common_0.96.20.7_all.deb ...
 Unpacking software-properties-common (0.96.20.7) ...
 Selecting previously unselected package xz-utils.
 Preparing to unpack .../xz-utils_5.1.1alpha+20120614-2ubuntu2_s390x.deb ...
 Unpacking xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 Selecting previously unselected package unattended-upgrades.
 Preparing to unpack .../unattended-upgrades_0.90ubuntu0.9_all.deb ...
 Unpacking unattended-upgrades (0.90ubuntu0.9) ...
 Processing triggers for man-db (2.7.5-1) ...
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.1) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 Setting up curl (7.47.0-1ubuntu2.7) ...
 Setting up python3-pycurl (7.43.0-1ubuntu1) ...
 Setting up python3-software-properties (0.96.20.7) ...
 Setting up software-properties-common (0.96.20.7) ...
 Setting up xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
 Setting up unattended-upgrades (0.90ubuntu0.9) ...

 Creating config file /etc/apt/apt.conf.d/50unattended-upgrades with new version
 Synchronizing state of unattended-upgrades.service with SysV init with /lib/systemd/systemd-sysv-install...
 Executing /lib/systemd/systemd-sysv-install enable unattended-upgrades
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.1) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 root@ubuntu16044:~#

**Step 2.6:**  Add Docker’s official GPG key::

 root@ubuntu16044:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16044:~#

**Step 2.7:** Verify that the key fingerprint is *9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88*::
 
 root@ubuntu16044:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22
 
 root@ubuntu16044:~#

**Step 2.8:** Enter the following command to add the *stable* repository that is provided by Docker::

 root@ubuntu16044:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16044:~#

**Step 2.9:** Update the *apt* package index again:: 

 root@ubuntu16044:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease [102 kB]                    
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease [102 kB]                           
 Get:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease [102 kB] 
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [65.8 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [2601 B]
 Fetched 375 kB in 0s (782 kB/s)     
 Reading package lists... Done

**Step 2.10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16044:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 17.12.1~ce-0~ubuntu
   Version table:
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
  root@ubuntu16044:~# 


Some key takeaways from the command output:

*	Docker is not currently installed *(Installed: (none))*
*	*17.12.0~ce-0~ubuntu* is the candidate version to install- it is the latest version available
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 2.11:** Enter this *apt-get* command to install Docker CE version 17.06.2.  It is very important to install this particular version.  (Enter Y when prompted to continue)::

 root@ubuntu16044:~# apt-get install docker-ce=17.06.2~ce-0~ubuntu
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki  git-svn diffutils-doc
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch
 0 upgraded, 8 newly installed, 0 to remove and 3 not upgraded.
 Need to get 22.7 MB of archives.
 After this operation, 125 MB of additional disk space will be used.
 Do you want to continue? [Y/n] Y
   .
   .   (remaining output not shown here)
   .

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**Step 2.12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16044:~# which docker
 /usr/bin/docker

**Step 2.13:** Enter the *docker version* command and you should see that version *17.06.2-ce* was installed::

 root@ubuntu16044:~# docker version
 Client:
  Version:      17.06.2-ce
  API version:  1.30
  Go version:   go1.8.3
  Git commit:   cec0b72
  Built:        Tue Sep  5 20:02:38 2017
  OS/Arch:      linux/s390x

 Server:
  Version:      17.06.2-ce
  API version:  1.30 (minimum version 1.12)
  Go version:   go1.8.3
  Git commit:   cec0b72
  Built:        Tue Sep  5 20:00:51 2017
  OS/Arch:      linux/s390x
  Experimental: false

**Step 2.14:** Enter *docker info* to see even more information about your Docker environment::

 root@ubuntu16044:~# docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 17.06.2-ce
 Storage Driver: aufs
  Root Dir: /var/lib/docker/aufs
  Backing Filesystem: extfs
  Dirs: 0
  Dirperm1 Supported: true
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
 containerd version: 6e23458c129b551d5c9871e5174f6b1b7f6d1170
 runc version: 810190ceaa507aa2727d7ae6f4790c76ec150bd2
 init version: 949e6fa
 Security Options:
  apparmor
 Kernel Version: 4.4.0-112-generic
 Operating System: Ubuntu 16.04.4 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 1.717GiB
 Name: ubuntu16044
 ID: NOQT:SZJX:HW5J:3PML:URFT:2WH7:L63S:BLBD:LM4C:NFEB:UGV2:MALK
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support


**Step 2.15:** After the Docker installation, non-root users cannot run Docker commands. One way to get around this for a non-root userid is to add that userid to a group named *docker*.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16044:~# usermod -aG docker bcuser
 
**Note:** This method of authorizing a non-root userid to enter Docker commands, while suitable for a controlled sandbox environment, may not be suitable for a production environemnt due to security considerations. 

**Step 2.16:** Exit so that you are no longer running as root::

 root@ubuntu16044:~# exit
 logout
 bcuser@ubuntu16044:~$
 
**Step 2.17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this 
change to take effect.  To prove this, before you log out, enter the *docker info* command and you will receive a permissions error::

 bcuser@ubuntu16044:~$ docker info
 Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get  http://%2Fvar%2Frun%2Fdocker.sock/v1.30/info: dial unix /var/run/docker.sock: connect: permission denied

**Step 2.18:** Now log out::

 bcuser@ubuntu16044:~$ exit
 logout
 Connection to 192.168.22.119 closed.

**Step 2.19:** Log in again.  (These instructions show logging in again using *ssh* from a command shell.  If you are using PuTTY you may need to start a new PuTTY session and log in)::

 $ ssh bcuser@192.168.22.119
 Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-112-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 bcuser@ubuntu16044:~$ 

before you log out and then again after you log in.  (You will 
need to start a new PuTTY session after you logged out so that you can get back in).

**Step 2.20:** Now try *docker info* and this time it should work from your non-root userid::

 bcuser@ubuntu16044:~$ docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 17.06.2-ce
 Storage Driver: aufs
  Root Dir: /var/lib/docker/aufs
  Backing Filesystem: extfs
  Dirs: 0
  Dirperm1 Supported: true
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
 containerd version: 6e23458c129b551d5c9871e5174f6b1b7f6d1170
 runc version: 810190ceaa507aa2727d7ae6f4790c76ec150bd2
 init version: 949e6fa
 Security Options:
  apparmor
 Kernel Version: 4.4.0-112-generic
 Operating System: Ubuntu 16.04.4 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 1.717GiB
 Name: ubuntu16044
 ID: NOQT:SZJX:HW5J:3PML:URFT:2WH7:L63S:BLBD:LM4C:NFEB:UGV2:MALK
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false 

 WARNING: No swap limit support

**Step 2.21:** You will need to get right back in as root to install *Docker Compose*.  Docker Compose is a tool provided by Docker to 
help make it easier to run an application that consists of multiple Docker containers.  On some platforms, it is installed along with 
the Docker package but on Linux on IBM Z it is installed separately.  It is written in Python and you will install it with a tool 
called Pip.  But first you will install Pip itself!  You will do this as root, so enter this again::

 bcuser@ubuntu16044:~$ sudo su -
 root@ubuntu16044:~#

**Step 2.22:** Install the *python-pip* package which will provide a tool named *Pip* which is used to install Python packages from a public repository::

 root@ubuntu16044:~# apt-get -y install python-pip

This will bring in a lot of prerequisites and will produce a lot of output which is not shown here.

**Step 2.23:** Run this command just to verify that *docker-compose* is not currently available on the system::

 root@ubuntu16044:~# which docker-compose
 root@ubuntu16044:~# 

**Step 2.24:** Use Pip to install Docker Compose::

 root@ubuntu16044:~# pip install docker-compose==1.20.0
 
You can ignore the suggestion at the end of the output of this command to consider upgrading *pip*- that isn't necessary for this lab.

**Step 2.25:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel pretty good about the output from this command::

 root@ubuntu16044:~# docker-compose --version
 docker-compose version 1.20.0, build ca8d3c6

**Note:** If the version of Docker Compose shown in your output differs from what is shown here, that's okay.

**Step 2.26:** Leave root behind and become a normal user again::

 root@ubuntu16044:~# exit
 logout
 bcuser@ubuntu16044:~$

**Step 2.27:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, 
check for the version again now that you are no longer root::

 bcuser@ubuntu16044:~$ docker-compose --version
 docker-compose version 1.20.0, build ca8d3c6

**Step 2.28:** The next thing you are going to install is the *Golang* programming language. You are going to install Golang version 
1.9.3.  Go to the /tmp directory::

 bcuser@ubuntu16044:~$ cd /tmp
 bcuser@ubuntu16044:/tmp$

**Step 2.29:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  And now is a good time to tell you 
that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::
 bcuser@ubuntu16044:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.9.3.linux-s390x.tar.gz
 --2018-03-19 18:38:40--  https://storage.googleapis.com/golang/go1.9.3.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.2.48, 2607:f8b0:4002:808::2010
 Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.2.48|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 88969164 (85M) [application/octet-stream]
 Saving to: 'go1.9.3.linux-s390x.tar.gz'

 go1.9.3.linux-s390x.tar.gz           100%[=====================================================================>]  84.85M  15.8MB/s    in 5.6s    

 2018-03-19 18:38:46 (15.1 MB/s) - 'go1.9.3.linux-s390x.tar.gz' saved [88969164/88969164]

**Step 2.30:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16044:/tmp$ tar -xvf go1.9.3.linux-s390x.tar.gz
   .
   .  (output not shown here)
   .

**Step 2.31:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a 
single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   Enter this command::

 bcuser@ubuntu16044:/tmp$ sudo mv -iv go /opt 
  'go' -> '/opt/go'

**Step 2.32:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16044:/tmp$ env | grep GO

That command, *grep*, is looking for any lines of input that contain the characters *GO*.  Its input is the output of the previous *env*
command, which prints all of your environment variables. Right now you should not see any output.

**Step 2.33:**  You will set these values now.  You will make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16044:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see 
 bcuser@ubuntu16044:~$

**Step 2.34:** Enter the *cp* command to make a backup copy of *.bashrc* to allow a recovery in the infinitesimally slim chance that you make a mistake in the subsequent five steps which will append information to *.bashrc*.  I know you wouldn't ever make a mistake, but the joker sitting next to you will.  Trust me.::

 bcuser@ubuntu16044:~$ cp -ipv .bashrc .bashrc_orig
 '.bashrc' -> '.bashrc_orig'

**Step 2.35:** The next five steps- *Steps 2.35 through 2.39* - are each *echo* commands which will append to the end of *.bashrc*.  The first and last of these steps just adds a blank line for readability.  Enter these exactly as shown in each step.  It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you 
enter them.  This appends the arguments of the *echo* commands to the end of the *.bashrc* file.  If you only enter one ‘>’ sign, you 
will overwrite the file’s contents.  I’d rather you not do that. Although *Step 2.34* does create a backup copy of the file,
just in case.  So first, add a blank line::

 bcuser@ubuntu16044:~$ echo '' >> .bashrc   # that is two single quotes, not one double-quote
 
**Step 2.36:** Add this line to set your *GOPATH* environment variable::

 bcuser@ubuntu16044:~$ echo export GOPATH=/home/bcuser/git >> .bashrc
 
**Step 2.37:** Add this line to set your *GOROOT* environment variable::

 bcuser@ubuntu16044:~$ echo export GOROOT=/opt/go >> .bashrc
 
**Step 2.38:** Add this line to update your *PATH* environment variable::

 bcuser@ubuntu16044:~$ echo export PATH=/opt/go/bin:/home/bcuser/bin:\$PATH >> .bashrc
 
**Step 2.39:** Finally, add another blank line for readability::

 bcuser@ubuntu16044:~$ echo '' >> .bashrc  

**Step 2.40:** Let’s see how you did.  Enter this command::

 bcuser@ubuntu16044:~$ head .bashrc
 # ~/.bashrc: executed by bash(1) for non-login shells.
 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
 # for examples
 
 # If not running interactively, don't do anything
 case $- in
     *i*) ;;
       *) return;;
 esac

If your output looked like the above, congratulations, you did not stomp all over your file. *head* prints the top of the file.  Had 
you made and mistake and used a single '>' instead two ‘>>’ like I told you, you would have whacked this stuff.  Your stuff is at the bottom.  If *head* 
prints the top of the file, guess what command prints the bottom of the file.

**Step 2.41:** Try this::

 bcuser@ubuntu16044:~$ tail -5 .bashrc
 
 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:$PATH

**Step 2.42:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16044:~$ source .bashrc

**Step 2.43:** Try this to see if your changes took::

 bcuser@ubuntu16044:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git

**Step 2.44:**  Then try this::

 bcuser@ubuntu16044:~$ go version
 go version go1.9.3 linux/s390x

**Recap:** Before you move on, here is a summary of the major tasks you performed in this section.

*	You installed Docker and added *bcuser* to the *docker* group so that *bcuser* can issue Docker commands
*	You installed Docker Compose (and Pip, which was needed to install it)
*	You installed Go
*	You updated your *.bashrc* profile to make necessary environment changes

In the next section, you will download the Hyperledger Fabric source code, build it, and run a comprehensive verification test using 
the Hyperledger Fabric Command Line Interface, or CLI.
 
Section 3: Download, build and test the Hyperledger Fabric CLI
==============================================================

In this section, you will:

*	Install some support packages using the Ubuntu package manager, *apt-get*
*	Download the source code repository containing the core Hyperledger Fabric functionality
*	Use the source code to build Docker images that contain the core Hyperledger Fabric functionality
*	Test for success by running the comprehensive end-to-end CLI test.

**Step 3.1:** There are some software packages necessary to be able to successfully build the Hyperledger Fabric source code.  Install them with 
this command. Observe the output, not shown here, to see the different packages 
installed::

 bcuser@ubuntu16044:~$ sudo apt-get install -y build-essential libltdl3-dev
 
**Step 3.2:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it. If you are following these steps exactly, you already are.  If you strayed away from your home directory, I'm assuming you're smart enough to get back there. (Or see *Step 2.33* if you accidentally left home and are too embarrassed to ask for help)::

 bcuser@ubuntu16044:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16044:~$
 
**Step 3.3:** Navigate to the directory you just created::

 bcuser@ubuntu16044:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$
 
**Step 3.4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place 
where it lives.  The *-b v1.1.0* argument specifies that you want the v1.1.0 release level::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ git clone -b v1.1.0 https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 4376, done
 remote: Finding sources: 100% (63/63)
 remote: Total 58781 (delta 9), reused 58753 (delta 9)
 Receiving objects: 100% (58781/58781), 75.03 MiB | 2.21 MiB/s, done.
 Resolving deltas: 100% (26968/26968), done.
 Checking connectivity... done.
 Note: checking out '523f644b909d5699fbd992a011e7ed6031e96a9a'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>


*Note:* The numbers in the various output messages may differ from what you see listed here, and this may be the case for any other times you do a *git clone* in the remainder of these labs.

**Step 3.5:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$

**Step 3.6:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 3.7:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You 
can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, 
including ‘wall clock’ time, required to build the images (See how it took over five minutes on my system.  It will probably take you a similar amount of time, so either check your email, fiddle with your smartphone, watch the output scroll by, or go to the bathroom really really quick)::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real	7m45.520s
 user	0m7.680s
 sys	0m0.810s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ 

**Step 3.8:** Run *docker images* again and you will see several Docker images that were just created. You will notice that many of the Docker images at the top of the output were created in the last few minutes.  These were created by the *make docker* command.  The Docker 
images that are several days or weeks old were downloaded from the Hyperledger Fabric's public 
DockerHub repository.  Your output should look similar to 
that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* 
will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-tools       latest              2669e1ed5d68        About a minute ago   1.37GB
 hyperledger/fabric-tools       s390x-1.1.0         2669e1ed5d68        About a minute ago   1.37GB
 hyperledger/fabric-testenv     latest              d8919b8bd414        About a minute ago   1.45GB
 hyperledger/fabric-testenv     s390x-1.1.0         d8919b8bd414        About a minute ago   1.45GB
 hyperledger/fabric-buildenv    latest              47e2cffaac5b        2 minutes ago        1.36GB
 hyperledger/fabric-buildenv    s390x-1.1.0         47e2cffaac5b        2 minutes ago        1.36GB
 hyperledger/fabric-orderer     latest              f80d36c050c6        2 minutes ago        203MB
 hyperledger/fabric-orderer     s390x-1.1.0         f80d36c050c6        2 minutes ago        203MB
 hyperledger/fabric-peer        latest              f4f7d97666d1        2 minutes ago        210MB
 hyperledger/fabric-peer        s390x-1.1.0         f4f7d97666d1        2 minutes ago        210MB
 hyperledger/fabric-javaenv     latest              6f236f0a0f7d        3 minutes ago        1.38GB
 hyperledger/fabric-javaenv     s390x-1.1.0         6f236f0a0f7d        3 minutes ago        1.38GB
 hyperledger/fabric-ccenv       latest              eb82f367e77a        3 minutes ago        1.3GB
 hyperledger/fabric-ccenv       s390x-1.1.0         eb82f367e77a        3 minutes ago        1.3GB
 hyperledger/fabric-zookeeper   latest              103c1abf45ff        4 weeks ago          1.34GB
 hyperledger/fabric-zookeeper   s390x-0.4.6         103c1abf45ff        4 weeks ago          1.34GB
 hyperledger/fabric-kafka       latest              db99e941fe20        4 weeks ago          1.35GB
 hyperledger/fabric-kafka       s390x-0.4.6         db99e941fe20        4 weeks ago          1.35GB
 hyperledger/fabric-couchdb     latest              2aecbce9f786        4 weeks ago          1.56GB
 hyperledger/fabric-couchdb     s390x-0.4.6         2aecbce9f786        4 weeks ago          1.56GB
 hyperledger/fabric-baseimage   s390x-0.4.6         234d9beb079b        4 weeks ago          1.27GB
 hyperledger/fabric-baseos      s390x-0.4.6         0eaed2e8996f        4 weeks ago          173MB

**Step 3.9:** Navigate to the directory where the “end-to-end” test lives::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ cd examples/e2e_cli/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$

**Step 3.10:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a 
process, and it is based on a Docker image.  Run this command, which shows all Docker containers, however right now there will be no 
output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 3.11:** Run the end-to-end test with this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 3.12:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                     PORTS                                                                       NAMES
 9ca69f0d2128        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.a..."   42 seconds ago       Up 41 seconds                                                                                           dev-peer1.org2.example.com-mycc-1.0
 5db95fdf7c9c        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.a..."   About a minute ago   Up 59 seconds                                                                                           dev-peer0.org1.example.com-mycc-1.0
 e53a8a16aeea        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.a..."   About a minute ago   Up About a minute                                                                                       dev-peer0.org2.example.com-mycc-1.0
 767ed190d28a        hyperledger/fabric-tools                                                                               "/bin/bash -c './s..."   2 minutes ago        Exited (0) 31 seconds ago                                                                               cli
 69444f336fcb        hyperledger/fabric-orderer                                                                             "orderer"                2 minutes ago        Up 2 minutes                0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 faf771f1110c        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 ce11df66c093        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 f1f4bd1c7b77        hyperledger/fabric-kafka                                                                               "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka0
 f94e92ea6a0f        hyperledger/fabric-kafka                                                                               "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka1
 5fd6364b8392        hyperledger/fabric-kafka                                                                               "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka2
 aa4b90dffeb5        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 8fc50a5b5a14        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 f64ba4f7b1fd        hyperledger/fabric-kafka                                                                               "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka3
 99ce8283647f        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32774->3888/tcp   zookeeper2
 ec5bf9d56a26        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 4e413ab38874        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 593b801b97fc        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 d73ae16c9842        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                0.0.0.0:32773->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper1
 a02f236e3a75        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoin..."   2 minutes ago        Up 2 minutes                0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper0
 c2781779f70e        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3


The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each 
had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one 
orderer container. The orderer service uses *Kafka* for consensus, and so is supported by four Kafka containers and three Zookeeper containers. There was a container created to run the CLI itself, and that container stopped running ten seconds after the 
test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 3.13:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh down

**Step 3.14:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Recap:** In this section, you:

*	Downloaded the main Hyperledger Fabric source code repository
*	Installed prerequisite tools required to build the Hyperledger Fabric project
*	Ran *make* to build the project’s Docker images
*	Ran the Hyperledger Fabric command line interface (CLI) end-to-end test
*	Cleaned up afterwards
 
Section 4: Install the Hyperledger Fabric Certificate Authority
===============================================================

In the prior section, the end-to-end test that you ran supplied its own security-related material such as keys and certificates- everything it needed to perform its test.  Therefore it did not need the services of a Certificate Authority.

Almost all "real world" Hyperledger Fabric networks will not be this static-  new users, peers and organizations will probably join the network.  They will need PKI x.509 certificates in order to participate.  The Hyperledger Fabric Certificate Authority (CA) is provided by the Hyperledger Fabric project in order to issue these certificates.

The next major goal in this lab is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  This test makes calls to the Hyperledger Fabric 
Certificate Authority (CA). Therefore, before we can run that test, you will get started by downloading and building the Hyperledger Fabric CA.

**Step 4.1:** Use *cd* to navigate three directory levels up, to the *hyperledger* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ~/git/src/github.com/hyperledger
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$

**Step 4.2:** Get the source code for the v1.1.0 release of the Fabric CA using *git*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ git clone -b v1.1.0 https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 15, done
 remote: Total 10048 (delta 0), reused 10048 (delta 0)
 Receiving objects: 100% (10048/10048), 24.65 MiB | 2.66 MiB/s, done.
 Resolving deltas: 100% (3535/3535), done.
 Checking connectivity... done.
 Note: checking out 'f69f53bfc4248d5f17e7a56072b634032decab35'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name

**Step 4.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just 
downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.4:** Enter the following command, which will build the Hyperledger Fabric CA images.  Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which 
is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time,
required to build the images::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca $ time make docker
   .
   .  (output not shown here)
   .
 real	2m0.509s
 user	0m0.148s
 sys	0m0.195s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.5:** Enter the *docker images* command and you will see at the top of the output the Docker images that were just created for 
the Fabric Certificate Authority::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca-tools     latest              c001ed8ba789        56 seconds ago       1.43GB
 hyperledger/fabric-ca-tools     s390x-1.1.0         c001ed8ba789        56 seconds ago       1.43GB
 hyperledger/fabric-ca-peer      latest              6fb441f2c0bd        About a minute ago   271MB
 hyperledger/fabric-ca-peer      s390x-1.1.0         6fb441f2c0bd        About a minute ago   271MB
 hyperledger/fabric-ca-orderer   latest              06391fff8d54        About a minute ago   265MB
 hyperledger/fabric-ca-orderer   s390x-1.1.0         06391fff8d54        About a minute ago   265MB
 hyperledger/fabric-ca           latest              2ac752a91a56        About a minute ago   292MB
 hyperledger/fabric-ca           s390x-1.1.0         2ac752a91a56        About a minute ago   292MB

   .
   . (remaining output not shown here)
   .

You may have noticed that for many of the images, the *Image ID* appears twice, once with a tag of *latest*, and once with a tag such as *s390x-1.1.0*. An image can be actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive 
tag, *s390x-1.1.0*, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker 
images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.1.0*, and in all three cases you are asking for the same image, the image with ID *2ac752a91a56*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 5: Install Hyperledger Fabric Node.js SDK and its prerequisite software
===============================================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that 
exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming 
language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will install and configure Node.js, which also includes a program called *npm*, which is the de facto Node.js 
package manager.  

Then you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 5.1:** Change to the */tmp* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$ cd /tmp
 bcuser@ubuntu16044:/tmp$

**Step 5.2:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16044:/tmp$ wget https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-s390x.tar.xz
 --2018-03-20 10:02:59--  https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.23.46, 104.20.22.46, 2400:cb00:2048:1::6814:162e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.23.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10977964 (10M) [application/x-xz]
 Saving to: 'node-v8.9.4-linux-s390x.tar.xz'

 node-v8.9.4-linux-s390x.tar.xz                   100%[=========================================================================================================>]  10.47M  16.6MB/s    in 0.6s    

 2018-03-20 10:03:00 (16.6 MB/s) - 'node-v8.9.4-linux-s390x.tar.xz' saved [10977964/10977964]

**Step 5.3:** Extract the package underneath your home directory, */home/bcuser*. This will cause the executables to wind up in 
*/home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16044:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.9.4-linux-s390x.tar.xz

**Step 5.4:** Issue this command to see where *node* resides within your path::

 bcuser@ubuntu16044:/tmp$ which node
 /home/bcuser/bin/node
 
**Step 5.5:** Issue this command to see where *npm* resides within your path::
 
 bcuser@ubuntu16044:/tmp $ which npm
 /home/bcuser/bin/npm
 
**Step 5.6:** Issue this command to see which version of *node* is installed::

 bcuser@ubuntu16044:/tmp $ node --version
 v8.9.4
 
**Step 5.7:** Issue this command to see which version of *npm* is installed::
 
 bcuser@ubuntu16044:/tmp$ npm --version
 5.6.0

**Step 5.8:** Switch to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16044:~$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$

**Step 5.9:** Now you will download the version 1.1.0 release of the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger $ git clone -b v1.1.0 https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 387, done
 remote: Finding sources: 100% (3/3)
 remote: Total 7475 (delta 0), reused 7474 (delta 0)
 Receiving objects: 100% (7475/7475), 4.60 MiB | 2.18 MiB/s, done.
 Resolving deltas: 100% (3627/3627), done.
 Checking connectivity... done.
 Note: checking out '46fc443fa8560032e8e77d4689581718190926c5'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 5.10:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.11:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.1.0 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.12:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 1048 packages in 290.505s


You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 5.13:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what 
everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong 
with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.14:** The tests use an automation tool named *gulp*, but before you install it, 
run the *which* command. The silent treatment it gives you confirms it is not available to you::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ 

**Step 5.15:** Now install *gulp* at a global level, using the *-g* argument to the *npm install*. This makes the package  available on a system-wide basis::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g gulp
   .
   .  (output not shown here)
   .
 
**Step 5.16:** Running *which* again shows that *gulp* is available to you now::
 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 /home/bcuser/bin/gulp
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.17:** Next you will install a code coverage testing tool named *istanbul*, also at a global level.  But first, use *which* to prove it isn't there yet::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ 
 
**Step 5.18:** Install it globally::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g istanbul
 /home/bcuser/bin/istanbul -> /home/bcuser/lib/node_modules/istanbul/lib/cli.js
 + istanbul@0.4.5
 added 60 packages in 1.211s

**Step 5.19:** Prove it worked::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 /home/bcuser/bin/istanbul
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Installed Node.js and npm
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
*	Installed the *gulp* and *istanbul* packages so that you are ready to run the Hyperledger Fabric Node.js SDK end-to-end test (which you will do in the next section)
 
Section 6: Run the Hyperledger Fabric Node.js SDK end-to-end test
=================================================================
In this section, you will run two tests provided by the Hyperledger Fabric Node.js SDK, verify their successful 
operation, and clean up afterwards.

The first test is a quick test that takes a little over twenty seconds, and does not bring up any chaincode containers.  The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take a few minutes.

**Step 6.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
 1..1083
 # tests 1083
 # pass  1083

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    65.29 |     61.4 |    55.26 |    65.29 |                |
   AffiliationService.js        |    66.67 |       70 |      100 |    66.67 |... 194,195,198 |
   FabricCAClientImpl.js        |    64.26 |    61.02 |    48.33 |    64.26 |... 924,926,929 |
   IdentityService.js           |    65.79 |       50 |    66.67 |    65.79 |... 254,255,258 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    67.73 |    62.67 |     68.7 |    67.85 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |     71.5 |       52 |    72.22 |    71.83 |... 4,1326,1327 |
   CertificateAuthority.js      |      100 |      100 |      100 |      100 |                |
   Channel.js                   |    49.58 |     45.4 |    57.89 |    49.47 |... 1,2343,2346 |
   ChannelEventHub.js           |    62.98 |    55.08 |    65.22 |    63.34 |... 3,1294,1296 |
   Client.js                    |    72.79 |    72.44 |    77.94 |    72.82 |... 6,1929,1932 |
   Config.js                    |    91.43 |       75 |      100 |    91.43 |      65,83,100 |
   Constants.js                 |      100 |      100 |      100 |      100 |                |
   EventHub.js                  |    69.91 |    65.85 |    67.74 |    70.37 |... 821,826,833 |
   Orderer.js                   |       50 |    35.71 |     62.5 |       50 |... 285,286,289 |
   Organization.js              |    84.78 |       80 |    93.33 |    86.05 |... 122,123,126 |
   Packager.js                  |    91.67 |    91.67 |      100 |    91.67 |          57,58 |
   Peer.js                      |    80.43 |     62.5 |    88.89 |    80.43 |... 140,142,143 |
   Policy.js                    |    99.07 |    92.16 |      100 |    99.07 |            169 |
   Remote.js                    |    97.78 |       90 |      100 |    97.78 |        102,114 |
   TransactionID.js             |       96 |     87.5 |      100 |       96 |             48 |
   User.js                      |    88.24 |    67.31 |       80 |    88.24 |... 226,246,253 |
   api.js                       |      100 |      100 |        0 |      100 |                |
   client-utils.js              |    73.95 |    58.97 |    73.33 |    73.95 |... 223,225,227 |
   hash.js                      |    45.59 |        0 |    10.53 |    45.59 |... 137,148,157 |
   utils.js                     |    79.41 |    72.88 |    77.78 |    79.41 |... 400,459,538 |
  fabric-client/lib/impl/       |    65.28 |    60.39 |    61.22 |     65.3 |                |
   CouchDBKeyValueStore.js      |    77.33 |       60 |    93.33 |    78.08 |... 158,171,172 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    84.34 |    80.22 |       75 |    84.34 |... 395,397,398 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |    98.74 |    85.63 |      100 |    98.72 |    147,412,413 |
   bccsp_pkcs11.js              |    24.86 |    32.24 |     8.33 |    24.93 |... 9,1113,1114 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    49.63 |    36.05 |       50 |    51.54 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |    11.69 |        0 |        0 |     12.5 |... 163,164,166 |
  fabric-client/lib/msp/        |    77.19 |    67.61 |    67.86 |    77.51 |                |
   identity.js                  |       90 |       76 |    76.92 |       90 |... ,86,104,212 |
   msp-manager.js               |    76.36 |    72.73 |    83.33 |    77.36 |... 129,130,159 |
   msp.js                       |    68.18 |    54.17 |    44.44 |    68.18 |... 137,138,180 |
  fabric-client/lib/packager/   |    90.35 |    77.27 |    76.92 |    90.35 |                |
   BasePackager.js              |    84.62 |    66.67 |       75 |    84.62 |... 150,173,191 |
   Car.js                       |       60 |      100 |        0 |       60 |          23,24 |
   Golang.js                    |      100 |      100 |      100 |      100 |                |
   Node.js                      |    96.15 |       75 |      100 |    96.15 |             82 |
 -------------------------------|----------|----------|----------|----------|----------------|
 All files                      |     67.2 |    61.04 |    65.24 |    67.36 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary =============================== 
 Statements   : 67.2% ( 4039/6010 )
 Branches     : 61.04% ( 1695/2777 )
 Functions    : 65.24% ( 456/699 )
 Lines        : 67.36% ( 4018/5965 )
 ================================================================================
 [11:28:50] Finished 'test-headless' after 23 s


You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this is that of the 1083 tests, all of them passed.  


**Step 6.2:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..1776
 # tests 1776
 # pass  1776

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    94.36 |    89.47 |    90.79 |    94.36 |                |
   AffiliationService.js        |    98.33 |       96 |      100 |    98.33 |            195 |
   FabricCAClientImpl.js        |    94.04 |     88.7 |       90 |    94.04 |... 918,926,929 |
   IdentityService.js           |    92.11 |    84.62 |    88.89 |    92.11 |... 246,249,255 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    86.84 |    78.72 |     83.7 |    86.98 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |    91.71 |       65 |    98.15 |    92.17 |... 8,1323,1324 |
   CertificateAuthority.js      |      100 |      100 |      100 |      100 |                |
   Channel.js                   |    78.38 |    71.31 |    81.58 |    78.45 |... 1,2343,2346 |
   ChannelEventHub.js           |    88.55 |    82.63 |    89.13 |    88.68 |... 2,1289,1296 |
   Client.js                    |    90.47 |    83.37 |    89.71 |    90.43 |... 6,1929,1932 |
   Config.js                    |    94.29 |     87.5 |      100 |    94.29 |         83,100 |
   Constants.js                 |      100 |      100 |      100 |      100 |                |
   EventHub.js                  |     92.1 |    84.55 |    93.55 |    92.59 |... 761,817,826 |
   Orderer.js                   |    79.23 |     62.5 |     87.5 |    79.23 |... 285,286,289 |
   Organization.js              |    84.78 |       80 |    93.33 |    86.05 |... 122,123,126 |
   Packager.js                  |    91.67 |    91.67 |      100 |    91.67 |          57,58 |
   Peer.js                      |    93.48 |    81.25 |      100 |    93.48 |    135,142,143 |
   Policy.js                    |    99.07 |    92.16 |      100 |    99.07 |            169 |
   Remote.js                    |      100 |    95.71 |      100 |      100 |                |
   TransactionID.js             |       96 |     87.5 |      100 |       96 |             48 |
   User.js                      |    91.76 |    73.08 |    86.67 |    91.76 |... 220,225,226 |
   api.js                       |      100 |      100 |        0 |      100 |                |
   client-utils.js              |    94.12 |    76.92 |      100 |    94.12 |... 210,223,225 |
   hash.js                      |    45.59 |        0 |    10.53 |    45.59 |... 137,148,157 |
   utils.js                     |    79.83 |    74.58 |    77.78 |    79.83 |... 400,459,538 |
  fabric-client/lib/impl/       |    66.16 |    61.27 |    62.24 |    66.08 |                |
   CouchDBKeyValueStore.js      |       88 |       70 |      100 |    87.67 |... 158,171,172 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    84.34 |    80.22 |       75 |    84.34 |... 395,397,398 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |    98.74 |    86.78 |      100 |    98.72 |    147,412,413 |
   bccsp_pkcs11.js              |    24.86 |    32.24 |     8.33 |    24.93 |... 9,1113,1114 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    49.63 |    36.05 |       50 |    51.54 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |    11.69 |        0 |        0 |     12.5 |... 163,164,166 |
  fabric-client/lib/msp/        |    78.36 |    69.01 |    71.43 |     78.7 |                |
   identity.js                  |       94 |       80 |    84.62 |       94 |      42,86,104 |
   msp-manager.js               |    76.36 |    72.73 |    83.33 |    77.36 |... 129,130,159 |
   msp.js                       |    68.18 |    54.17 |    44.44 |    68.18 |... 137,138,180 |
  fabric-client/lib/packager/   |    90.35 |    77.27 |    76.92 |    90.35 |                |
   BasePackager.js              |    84.62 |    66.67 |       75 |    84.62 |... 150,173,191 |
   Car.js                       |       60 |      100 |        0 |       60 |          23,24 |
   Golang.js                    |      100 |      100 |      100 |      100 |                |
   Node.js                      |    96.15 |       75 |      100 |    96.15 |             82 |
 -------------------------------|----------|----------|----------|----------|----------------|
 All files                      |    82.91 |    74.11 |    79.26 |     83.1 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary =============================== 
 Statements   : 82.91% ( 4983/6010 )
 Branches     : 74.11% ( 2058/2777 )
 Functions    : 79.26% ( 554/699 )
 Lines        : 83.1% ( 4957/5965 )
 ================================================================================
 [11:39:54] Finished 'test' after 7.82 min
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 6.3:** (Optional) What I really like about the second end-to-end test is that it cleans itself up really well at the beginning- that is, it will remove any artifacts left running at the end of the prior test, so if you wanted to, you could simply enter *gulp test* again if you'd like to see this for yourself and have several minutes to spare.  If you're pressed for time, skip this step::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (output not shown here)
   . 

**Step 6.4:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a

**Step 6.5:** Enter this command to see that some Docker images for chaincode have been created as part of the test.  These are the images that start with *dev-*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 
**Step 6.6:** You will now clean up. You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 
**Step 6.7:** Now observe that all Docker containers have been stopped and removed by entering this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 
**Step 6.8:** And enter this comand and see that all chaincode images (those starting with *dev-*) have been removed::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward.
This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
