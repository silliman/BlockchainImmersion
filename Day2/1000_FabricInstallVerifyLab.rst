Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.3 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  During the installation process for this instance, no extra packages were selected for installation beyond the minimum offered by the installer.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.3 LTS* and the Linux kernel is at level *4.4.0-112*.

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
**Step 2.1:** Log in to your assigned Ubuntu 16.04.3 Linux on IBM Z instance using the PuTTY icon on your workstation using the IP address assigned to your team.  You will be greeted with a message like this::

  Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-112-generic s390x)

   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  Last login: Fri Feb  2 12:39:45 2018 from 192.168.22.64
  bcuser@ubuntu16043:~$ 

**Step 2.2:** This system is a relatively “clean” Ubuntu 16.04.3 instance- “clean” in the sense that after this instance was created,
no additional software was installed.  You will therefore need to install several software prerequisites.  The first thing you will 
install is Docker. Docker is a software container platform that is an integral part of Hyperledger Fabric.  *Smart contracts*, also 
known as *chaincode*, run as Docker containers.  In addition, you will run the Hyperledger Fabric processes themselves in Docker 
containers.  You could choose to run the Hyperledger Fabric processes natively on your host operating system, that is, *not* in Docker 
containers, but even if you did this you would still need Docker to run chaincode.  For this lab, you will use Docker containers for *both* the chaincode and the Hyperledger Fabric processes.  

Issue this *which* command, which attempts to find the *docker* executable. The fact that it gives no response other than returning to 
the command prompt indicates that the program *docker* is not found::

    bcuser@ubuntu16043:~$ which docker
    bcuser@ubuntu16043:~$

**Note:** Some Linux distributions produce no output if the executable is not found, as in the above example.  Other Linux distributions
may produce an error message stating that the executable is not found.
   
**Step 2.3:** You will be using root authority for several commands throughout this lab.  You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  Use the *sudo* command to switch to the root user.  Observe the change in the command prompt after you enter this command::

  bcuser@ubuntu16043:~$ sudo su -
  root@ubuntu16043:~#

**Step 2.4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. You will be 
using *apt* throughout this lab to install various software packages. The first thing you need to do is install the 
Docker Community Edition (CE).  This software is provided by Docker, so the next several steps will add a repository managed by Docker 
to your system’s list of repositories so that you can install Docker CE. Enter this command to update the *apt* package index::

 root@ubuntu16043:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease [102 kB]            
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease [102 kB]          
 Get:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease [102 kB]    
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x Packages [547 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports/universe s390x Packages [5668 B]
 Fetched 859 kB in 0s (1871 kB/s)                                                
 Reading package lists... Done
 root@ubuntu16043:~# 

 
**Step 2.5:** Install packages to allow *apt* to use a repository over HTTPS::

 root@ubuntu16043:~# apt-get install -y apt-transport-https ca-certificates curl software-properties-common
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 apt-transport-https is already the newest version (1.2.24).
 ca-certificates is already the newest version (20170717~16.04.1).
 The following additional packages will be installed:
   python3-pycurl python3-software-properties unattended-upgrades xz-utils
 Suggested packages:
   libcurl4-gnutls-dev python-pycurl-doc python3-pycurl-dbg bsd-mailx mail-transport-agent
 The following NEW packages will be installed:
   curl python3-pycurl python3-software-properties software-properties-common unattended-upgrades xz-utils
 0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
 Need to get 317 kB of archives.
 After this operation, 1552 kB of additional disk space will be used.
 Get:1 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x curl s390x 7.47.0-1ubuntu2.6 [137 kB]
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x python3-pycurl s390x 7.43.0-1ubuntu1 [39.9 kB]
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x python3-software-properties all 0.96.20.7 [20.3 kB]
 Get:4 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x software-properties-common all 0.96.20.7 [9452 B]
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x xz-utils s390x 5.1.1alpha+20120614-2ubuntu2 [78.4 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x unattended-upgrades all 0.90ubuntu0.9 [32.3 kB]
 Fetched 317 kB in 0s (2139 kB/s)              
 Preconfiguring packages ...
 Selecting previously unselected package curl.
 (Reading database ... 64248 files and directories currently installed.)
 Preparing to unpack .../curl_7.47.0-1ubuntu2.6_s390x.deb ...
 Unpacking curl (7.47.0-1ubuntu2.6) ...
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
 Processing triggers for systemd (229-4ubuntu21) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 Setting up curl (7.47.0-1ubuntu2.6) ...
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
 Processing triggers for systemd (229-4ubuntu21) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 root@ubuntu16043:~# 


**Step 2.6:**  Add Docker’s official GPG key::

 root@ubuntu16043:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16043:~#

**Step 2.7:** Verify that the key fingerprint is *9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88*::
 
 root@ubuntu16043:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22
 
 root@ubuntu16043:~#

**Step 2.8:** Enter the following command to add the *stable* repository that is provided by Docker::

 root@ubuntu16043:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16043:~#

**Step 2.9:** Update the *apt* package index again:: 

 root@ubuntu16043:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease         
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [65.8 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [2504 B]
 Fetched 68.3 kB in 0s (100 kB/s)    
 Reading package lists... Done


**Step 2.10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16043:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 17.12.0~ce-0~ubuntu
   Version table:
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
 root@ubuntu16043:~# 


Some key takeaways from the command output:

*	Docker is not currently installed *(Installed: (none))*
*	*17.12.0~ce-0~ubuntu* is the candidate version to install- it is the latest version available
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 2.11:** Enter this *apt-get* command to install Docker CE version 17.06.2.  It is very important to install this particular version.  (Enter Y when prompted to continue)::

 root@ubuntu16043:~# apt-get install docker-ce=17.06.2~ce-0~ubuntu
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki  git-svn
   diffutils-doc
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch
 0 upgraded, 8 newly installed, 0 to remove and 0 not upgraded.
 Need to get 25.8 MB of archives.
 After this operation, 145 MB of additional disk space will be used.

 Do you want to continue? [Y/n] Y
   .
   .   (remaining output not shown here)
   .

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**Step 2.12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16043:~# which docker
 /usr/bin/docker

**Step 2.13:** Enter the *docker version* command and you should see that version *17.06.2-ce* was installed::

 root@ubuntu16043:~# docker version
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

 root@ubuntu16043:~# docker info
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
 Operating System: Ubuntu 16.04.3 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 1.717GiB
 Name: ubuntu16043
 ID: S36S:W7Z2:2SWO:7WSO:V4CB:5YEJ:6KTU:JOXP:J4WH:3EIY:U3XK:KSD4
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

**Step 2.15:** After the Docker installation, non-root users cannot run Docker commands. One way to get around this for a non-root userid is to add that userid to a group named *docker*.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16043:~# usermod -aG docker bcuser
 
**Note:** This method of authorizing a non-root userid to enter Docker commands, while suitable for a controlled sandbox environment, may not be suitable for a production environemnt due to security considerations. 

**Step 2.16:** Exit so that you are no longer running as root::

 root@ubuntu16043:~# exit
 logout
 bcuser@ubuntu16043:~$
 
**Step 2.17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this 
change to take effect.  To prove this, enter the *docker info* command before you log out and then again after you log in.  (You will 
need to start a new PuTTY session after you logged out so that you can get back in).

**Step 2.18:** You will need to get right back in as root to install *Docker Compose*.  Docker Compose is a tool provided by Docker to 
help make it easier to run an application that consists of multiple Docker containers.  On some platforms, it is installed along with 
the Docker package but on Linux on IBM Z it is installed separately.  It is written in Python and you will install it with a tool 
called Pip.  But first you will install Pip itself!  You will do this as root, so enter this again::

 bcuser@ubuntu16043:~$ sudo su -
 root@ubuntu16043:~#

**Step 2.19:** Install the *python-pip* package which will provide a tool named *Pip* which is used to install Python packages from a public repository::

 root@ubuntu16043:~# apt-get -y install python-pip

This will bring in a lot of prerequisites and will produce a lot of output which is not shown here.

**Step 2.20:** Run this command just to verify that *docker-compose* is not currently available on the system::

 root@ubuntu16043:~# which docker-compose
 root@ubuntu16043:~# 

**Step 2.21:** Use Pip to install Docker Compose::

 root@ubuntu16043:~# pip install docker-compose
 
**Step 2.22:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel pretty good about the output from this command::

 root@ubuntu16043:~# docker-compose --version
 docker-compose version 1.19.0, build 9e633ef

**Note:** If the version of Docker Compose shown in your output differs from what is shown here, that's okay.

**Step 2.23:** Leave root behind and become a normal user again::

 root@ubuntu16043:~# exit
 logout
 bcuser@ubuntu16043:~$

**Step 2.24:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, 
check for the version again now that you are no longer root::

 bcuser@ubuntu16043:~$ docker-compose --version
 docker-compose version 1.19.0, build 9e633ef

**Step 2.25:** The next thing you are going to install is the *Golang* programming language. You are going to install Golang version 
1.9.3.  Go to the /tmp directory::

 bcuser@ubuntu16043:~$ cd /tmp
 bcuser@ubuntu16043:/tmp$

**Step 2.26:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  And now is a good time to tell you 
that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::
 bcuser@ubuntu16043:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.9.3.linux-s390x.tar.gz
 --2018-02-02 14:05:17--  https://storage.googleapis.com/golang/go1.9.3.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 74.125.21.128, 2607:f8b0:4002:c06::80
 Connecting to storage.googleapis.com (storage.googleapis.com)|74.125.21.128|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 88969164 (85M) [application/octet-stream]
 Saving to: 'go1.9.3.linux-s390x.tar.gz'
 
 go1.9.3.linux-s390x.tar.gz           100%[===================================================================>]  84.85M  9.23MB/s    in 8.0s    

 2018-02-02 14:05:25 (10.6 MB/s) - 'go1.9.3.linux-s390x.tar.gz' saved [88969164/88969164]


**Step 2.27:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16043:/tmp$ tar -xvf go1.9.3.linux-s390x.tar.gz
   .
   .  (output not shown here)
   .

**Step 2.28:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a 
single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   Enter this command::

 bcuser@ubuntu16043:/tmp$ sudo mv -iv go /opt 
  'go' -> '/opt/go'

**Step 2.29:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16043:/tmp$ env | grep GO

That command, *grep*, is looking for any lines of input that contain the characters *GO*.  Its input is the output of the previous *env*
command, which prints all of your environment variables. Right now you should not see any output.

**Step 2.30:**  You will set these values now.  You will make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16043:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see 
 bcuser@ubuntu16043:~$

**Step 2.31:** There are six commands in this step- a *cp* command (to make a backup) and then five *echo* commands. Enter them exactly as shown.  It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you 
enter the five *echo* commands in this step.  This appends the arguments of the *echo* commands to the end of the *.bashrc* file.  If you only enter one ‘>’ sign, you 
will overwrite the file’s contents.  I’d rather you not do that. Although the first command shown does create a backup copy of the file,
just in case::

 bcuser@ubuntu16043:~$ cp -ipv .bashrc .bashrc_orig
 '.bashrc' -> '.bashrc_orig'
 bcuser@ubuntu16043:~$ echo '' >> .bashrc   # that is two single quotes, not one double-quote
 bcuser@ubuntu16043:~$ echo export GOPATH=/home/bcuser/git >> .bashrc
 bcuser@ubuntu16043:~$ echo export GOROOT=/opt/go >> .bashrc
 bcuser@ubuntu16043:~$ echo export PATH=/opt/go/bin:/home/bcuser/bin:\$PATH >> .bashrc
 bcuser@ubuntu16043:~$ echo '' >> .bashrc  # that is two single quotes, not one double-quote

**Step 2.32:** Let’s see how you did.  Enter this command::

 bcuser@ubuntu16043:~$ head .bashrc
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

**Step 2.33:** Try this::

 bcuser@ubuntu16043:~$ tail -5 .bashrc
 
 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:$PATH

**Step 2.34:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16043:~$ source .bashrc

**Step 2.35:** Try this to see if your changes took::

 bcuser@ubuntu16043:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git

**Step 2.36:**  Then try this::

 bcuser@ubuntu16043:~$ go version
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

 bcuser@ubuntu16043:~$ sudo apt-get install -y build-essential libltdl3-dev
 
**Step 3.2:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it. If you are following these steps exactly, you already are.  If you strayed away from your home directory, I'm assuming you're smart enough to get back there. (Or see *Step 2.30* if you accidentally left home and are too embarrassed to ask for help)::

 bcuser@ubuntu16043:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16043:~$
 
**Step 3.3:** Navigate to the directory you just created::

 bcuser@ubuntu16043:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$
 
**Step 3.4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place 
where it lives::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 4732, done
 remote: Finding sources: 100% (18/18)
 remote: Total 55714 (delta 4), reused 55707 (delta 4)
 Receiving objects: 100% (55714/55714), 67.60 MiB | 2.97 MiB/s, done.
 Resolving deltas: 100% (25790/25790), done.
 Checking connectivity... done.

*Note:* The numbers in the various output messages will almost certainly be different that what you see listed here since code is continuously being created or modified.  This will probably be the case for any other times you do a *git clone* in the remainder of these labs.

**Step 3.5:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$

**Step 3.6:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean
 
You pulled down, by default, the *master* branch of the Hyperledger Fabric code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 3.7:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.1.0-alpha*, specified in 
this example.  If not, use the level from this example::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ git checkout v1.1.0-alpha
 Note: checking out 'v1.1.0-alpha'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

 HEAD is now at 0f38dbc... FAB-7782 prepare fabric for v1.1.0-alpha


**Step 3.8:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 3.9:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You 
can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, 
including ‘wall clock’ time, required to build the images (See how it took over five minutes on my system.  It will probably take you a similar amount of time, so either check your email, fiddle with your smartphone, watch the output scroll by, or go the bathroom really really quick)::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real	5m42.284s
 user	0m7.726s
 sys	0m0.855s
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ 

**Step 3.10:** Run *docker images* again and you will see several Docker images that were just created. You will notice that many of the Docker images at the top of the output were created in the last few minutes.  These were created by the *make docker* command.  The Docker 
images that are several days old were downloaded from the Hyperledger Fabric's public 
DockerHub repository.  Your output should look similar to 
that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* 
will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
 hyperledger/fabric-tools       latest              c1401c06d27d        10 minutes ago      1.39GB
 hyperledger/fabric-tools       s390x-1.1.0-alpha   c1401c06d27d        10 minutes ago      1.39GB
 hyperledger/fabric-testenv     latest              513c0c7a26c9        11 minutes ago      1.48GB
 hyperledger/fabric-testenv     s390x-1.1.0-alpha   513c0c7a26c9        11 minutes ago      1.48GB
 hyperledger/fabric-buildenv    latest              8a36c724ab27        11 minutes ago      1.38GB
 hyperledger/fabric-buildenv    s390x-1.1.0-alpha   8a36c724ab27        11 minutes ago      1.38GB
 hyperledger/fabric-orderer     latest              2fc8442b76b1        12 minutes ago      227MB
 hyperledger/fabric-orderer     s390x-1.1.0-alpha   2fc8442b76b1        12 minutes ago      227MB
 hyperledger/fabric-peer        latest              561492d179fe        12 minutes ago      233MB
 hyperledger/fabric-peer        s390x-1.1.0-alpha   561492d179fe        12 minutes ago      233MB
 hyperledger/fabric-javaenv     latest              17854e9941f4        12 minutes ago      1.41GB
 hyperledger/fabric-javaenv     s390x-1.1.0-alpha   17854e9941f4        12 minutes ago      1.41GB
 hyperledger/fabric-ccenv       latest              56d21a730ee7        12 minutes ago      1.32GB
 hyperledger/fabric-ccenv       s390x-1.1.0-alpha   56d21a730ee7        12 minutes ago      1.32GB
 hyperledger/fabric-zookeeper   latest              b0c2e8c5bb5a        10 days ago         1.45GB
 hyperledger/fabric-zookeeper   s390x-0.4.5         b0c2e8c5bb5a        10 days ago         1.45GB
 hyperledger/fabric-kafka       latest              40368c670c52        10 days ago         1.44GB
 hyperledger/fabric-kafka       s390x-0.4.5         40368c670c52        10 days ago         1.44GB
 hyperledger/fabric-couchdb     latest              b4c1f99eddbc        10 days ago         1.7GB
 hyperledger/fabric-couchdb     s390x-0.4.5         b4c1f99eddbc        10 days ago         1.7GB
 hyperledger/fabric-baseimage   s390x-0.4.5         53ef7b4cc042        12 days ago         1.3GB
 hyperledger/fabric-baseos      s390x-0.4.5         3f9fd299a01a        12 days ago         197MB


**Step 3.11:** Navigate to the directory where the “end-to-end” test lives::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric$ cd examples/e2e_cli/
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$

**Step 3.12:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a 
process, and it is based on a Docker image.  Run this command, which shows all Docker containers, and right now there will be no 
output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 3.13:** Run the end-to-end test with this command::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 3.14:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                     PORTS                                                                       NAMES
 2a7301793091        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   20 seconds ago       Up 18 seconds                                                                                          dev-peer1.org2.example.com-mycc-1.0
 f5861cbc95bc        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   38 seconds ago       Up 37 seconds                                                                                          dev-peer0.org1.example.com-mycc-1.0
 46251467c30d        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   56 seconds ago       Up 55 seconds                                                                                          dev-peer0.org2.example.com-mycc-1.0
 5f1f4d676a1a        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   About a minute ago   Exited (0) 8 seconds ago                                                                               cli
 74510a47b7e6        hyperledger/fabric-orderer                                                                             "orderer"                About a minute ago   Up About a minute          0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 0ebdeb0132ee        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka2
 a00711597766        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka1
 ea5b2c86a8cb        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka3
 a1db3b9f179c        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka0
 b3c2ac17e9e6        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 ccaf8f7b042b        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 31cdcbb934ab        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 66ab8606db93        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 dd73d4901d5b        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32773->3888/tcp   zookeeper2
 8148f8cf3ac1        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32774->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper1
 b92a5e64094b        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 d67b083f0bea        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 d31bf5016c64        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 0ebc4097b66f        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 660d6cda2ac3        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper0


The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each 
had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one 
orderer container. The orderer service uses *Kafka* for consensus, and so is supported by four Kafka containers and three Zookeeper containers. There was a container created to run the CLI itself, and that container stopped running ten seconds after the 
test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 3.15:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh down

**Step 3.16:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
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

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ../../..
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$

**Step 4.2:** Get the source code for the CA using *git*::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 12, done
 remote: Total 9710 (delta 0), reused 9710 (delta 0)
 Receiving objects: 100% (9710/9710), 24.35 MiB | 2.67 MiB/s, done.
 Resolving deltas: 100% (3367/3367), done.
 Checking connectivity... done.

**Step 4.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just 
downloaded::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.4:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean

You pulled down, by default, the master branch of the Hyperledger Fabric CA code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 4.5:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.1.0-alpha*, specified in this 
example.  If not, use the level from this example::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$ git checkout v1.1.0-alpha
 Note: checking out 'v1.1.0-alpha'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

 HEAD is now at 948da5a... FAB-7783 prepare fabric-ca for v1.1.0-alpha
 
**Step 4.6:** Enter the following command, which will build the Hyperledger Fabric CA images.  You can ‘wrap’ the *make* command, which 
is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time,
required to build the images::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca $ time make docker
   .
   .  (output not shown here)
   .
 real	2m2.411s
 user	0m0.125s
 sys	0m0.148s
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.7:** Enter the *docker images* command and you will see at the top of the output the Docker images that were just created for 
the Certificate Authority::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca-tools     latest              c910ceb22bcd        48 seconds ago       1.44GB
 hyperledger/fabric-ca-tools     s390x-1.1.0-alpha   c910ceb22bcd        48 seconds ago       1.44GB
 hyperledger/fabric-ca-peer      latest              ea8adddd1b40        About a minute ago   286MB
 hyperledger/fabric-ca-peer      s390x-1.1.0-alpha   ea8adddd1b40        About a minute ago   286MB
 hyperledger/fabric-ca-orderer   latest              d89e2f889289        About a minute ago   280MB
 hyperledger/fabric-ca-orderer   s390x-1.1.0-alpha   d89e2f889289        About a minute ago   280MB
 hyperledger/fabric-ca           latest              367aef7bf819        About a minute ago   297MB
 hyperledger/fabric-ca           s390x-1.1.0-alpha   367aef7bf819        About a minute ago   297MB

   .
   . (remaining output not shown here)
   .

You may have noticed that for many of the images, the *Image ID* appears twice, once with a tag of *latest*, and once with a tag such as *s390x-1.1.0-alpha*. An image can be actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive 
tag, *s390x-1.1.0-alpha*, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker 
images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.1.0-alpha*, and in all three cases you are asking for the same image, the image with ID *367aef7bf819*.

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

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-ca$ cd /tmp
 bcuser@ubuntu16043:/tmp$

**Step 5.2:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16043:/tmp$ wget https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-s390x.tar.xz
 --2018-02-02 15:39:12--  https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.22.46, 104.20.23.46, 2400:cb00:2048:1::6814:162e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.22.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10977964 (10M) [application/x-xz]
 Saving to: 'node-v8.9.4-linux-s390x.tar.xz'

 node-v8.9.4-linux-s390x.tar.xz                    100%[==========================================================================================================>]  10.47M  20.1MB/s    in 0.5s    

 2018-02-02 15:39:13 (20.1 MB/s) - 'node-v8.9.4-linux-s390x.tar.xz' saved [10977964/10977964]

**Step 5.3:** Extract the package underneath your home directory, */home/bcuser*. This will cause the executables to wind up in 
*/home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16043:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.9.4-linux-s390x.tar.xz

**Step 5.4:** In this step, you will issue some commands that will show you where *node* and *npm* reside, and what version of each is installed::

 bcuser@ubuntu16043:/tmp$ which node
 /home/bcuser/bin/node
 bcuser@ubuntu16043:/tmp $ which npm
 /home/bcuser/bin/npm
 bcuser@ubuntu16043:/tmp $ node --version
 v8.9.4
 bcuser@ubuntu16043:/tmp$ npm --version
 5.6.0

**Step 5.5:** Switch to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16043:~$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger$

**Step 5.6:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger $ git clone https://gerrit.hyperledger.org/r/fabric-sdk-node

 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 9, done
 remote: Total 6704 (delta 0), reused 6704 (delta 0)
 Receiving objects: 100% (6704/6704), 4.07 MiB | 1.16 MiB/s, done.
 Resolving deltas: 100% (3263/3263), done.
 Checking connectivity... done.

**Step 5.7:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.8:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean

You pulled down, by default, the master branch of the Hyperledger Fabric Node.js SDK code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 5.9:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.1.0-alpha*, specified in this 
example.  If not, use the level from this example::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ git checkout v1.1.0-alpha
 Note: checking out 'v1.1.0-alpha'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

 HEAD is now at ad919a2... FAB-7784 prepare fabric-sdk-node for 1.1.0-alpha

**Step 5.10:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.1.0-alpha /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)

 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.11:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"}  (current: {"os":"linux","arch":"s390x"})

 added 1020 packages in 63.632s


You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 5.12:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what 
everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong 
with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16043: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.13:** Now you will install an automation tool named *gulp* at a global level, using the *-g* argument to the *npm install* 
command.  This makes the package installed available on a system-wide basis. Run the *which* command before and after the *npm install* 
command to verify success::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g gulp
   .
   .  (output not shown here)
   .
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 /home/bcuser/bin/gulp
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.14:** Next you will install a code coverage testing tool named *istanbul*, also at a global level::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g istanbul
   .
   .  (output not shown here)
   .
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 /home/bcuser/bin/istanbul
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Installed Node.js and npm
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
*	Installed the *gulp* and *istanbul* packages so that you are ready to run the Hyperledger Fabric Node.js SDK end-to-end test (which you will do in the next section)
 
Section 6: Run the Hyperledger Fabric Node.js SDK end-to-end test
=================================================================
In this section, you will run two tests  provided by the Hyperledger Fabric Node.js SDK, verify their successful 
operation, and clean up afterwards.

The first test is a quick test that takes a little over twenty seconds, and does not bring up any chaincode containers.  The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take a few minutes.

**Step 6.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
 1..835
 # tests 835
 # pass  835

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    50.11 |    40.28 |    39.47 |    50.11 |                |
   AffiliationService.js        |        5 |        0 |        0 |        5 |... 194,195,198 |
   FabricCAClientImpl.js        |    64.67 |    61.71 |    48.33 |    64.67 |... 917,919,922 |
   IdentityService.js           |    21.05 |        0 |        0 |    21.05 |... 254,255,258 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    63.68 |     56.9 |    65.78 |     63.8 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |    70.42 |       50 |    72.22 |    70.85 |... 2,1344,1345 |
   CertificateAuthority.js      |     96.3 |       90 |      100 |     96.3 |             52 |
   Channel.js                   |    48.74 |    44.24 |    61.11 |    48.62 |... 4,2235,2238 |
   ChannelEventHub.js           |    63.02 |    55.08 |    65.22 |    63.38 |... 2,1293,1295 |
   Client.js                    |    52.49 |     50.8 |    58.46 |    52.43 |... 0,1901,1902 |
   Config.js                    |    94.29 |     87.5 |      100 |    94.29 |         83,100 |
   Constants.js                 |      100 |      100 |      100 |      100 |                |
   EventHub.js                  |    68.52 |    65.29 |    64.52 |    68.97 |... 815,820,827 |
   Orderer.js                   |    48.87 |    37.04 |     62.5 |    48.87 |... 290,291,294 |
   Organization.js              |    82.98 |       80 |    86.67 |    84.09 |... 124,125,128 |
   Packager.js                  |     91.3 |    91.67 |      100 |     91.3 |          54,55 |
   Peer.js                      |    78.26 |    56.25 |    88.89 |    78.26 |... 140,142,143 |
   Policy.js                    |    99.03 |    92.16 |      100 |    99.03 |            162 |
   Remote.js                    |      100 |    92.19 |      100 |      100 |                |
   TransactionID.js             |       96 |     87.5 |      100 |       96 |             48 |
   User.js                      |    88.24 |    67.31 |       80 |    88.24 |... 226,246,253 |
   api.js                       |      100 |      100 |        0 |      100 |                |
   client-utils.js              |    73.91 |    57.14 |    73.33 |    73.91 |... 217,219,221 |
   hash.js                      |    45.59 |        0 |    10.53 |    45.59 |... 137,148,157 |
   utils.js                     |    79.57 |    73.68 |    77.14 |    79.57 |... 396,398,457 |
  fabric-client/lib/impl/       |    61.27 |    49.81 |     60.2 |    61.34 |                |
   CouchDBKeyValueStore.js      |    76.92 |       60 |    93.33 |    77.63 |... 162,175,176 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    84.34 |    80.22 |       75 |    84.34 |... 395,397,398 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |     90.3 |    68.45 |    94.12 |    90.09 |... 390,407,408 |
   bccsp_pkcs11.js              |     17.8 |    16.89 |     8.33 |    18.07 |... 8,1052,1053 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    49.63 |    36.05 |       50 |    51.54 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |    11.69 |        0 |        0 |     12.5 |... 163,164,166 |
  fabric-client/lib/msp/        |    77.19 |    67.61 |    67.86 |    77.51 |                |
   identity.js                  |       90 |       76 |    76.92 |       90 |... ,86,104,212 |
   msp-manager.js               |    76.36 |    72.73 |    83.33 |    77.36 |... 129,130,159 |
   msp.js                       |    68.18 |    54.17 |    44.44 |    68.18 |... 137,138,180 |
  fabric-client/lib/packager/   |    88.17 |    64.29 |    76.92 |    88.17 |                |
   BasePackager.js              |    77.78 |       50 |    66.67 |    77.78 |... ,95,118,136 |
   Car.js                       |       60 |      100 |        0 |       60 |          23,24 |
   Golang.js                    |      100 |      100 |      100 |      100 |                |
   Node.js                      |    95.65 |       50 |      100 |    95.65 |             77 |
 -------------------------------|----------|----------|----------|----------|----------------|
 All files                      |    62.48 |    52.97 |    61.39 |    62.63 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 62.48% ( 3698/5919 )
 Branches     : 52.97% ( 1426/2692 )
 Functions    : 61.39% ( 423/689 )
 Lines        : 62.63% ( 3677/5871 )
 ================================================================================
 [16:03:17] Finished 'test-headless' after 23 s


You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this is that of 835 tests, all of them passed.  


 
**Step 6.2:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..1457
 # tests 1457
 # pass  1457

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    89.32 |    81.27 |    90.79 |    89.32 |                |
   AffiliationService.js        |    76.67 |       70 |      100 |    76.67 |... 178,184,195 |
   FabricCAClientImpl.js        |    94.01 |    88.57 |       90 |    94.01 |... 911,919,922 |
   IdentityService.js           |    78.95 |    65.38 |    88.89 |    78.95 |... 246,249,255 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    86.27 |    75.74 |    83.56 |    86.45 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |    90.08 |     62.5 |    98.15 |    90.68 |... 6,1341,1342 |
   CertificateAuthority.js      |     96.3 |       90 |      100 |     96.3 |             52 |
   Channel.js                   |    81.78 |    68.79 |    86.11 |    81.95 |... 4,2235,2238 |
   ChannelEventHub.js           |    88.49 |    82.63 |    89.13 |    88.61 |... 1,1288,1295 |
   Client.js                    |    84.53 |    74.94 |    86.15 |    84.47 |... 0,1901,1902 |
   Config.js                    |    94.29 |     87.5 |      100 |    94.29 |         83,100 |
   Constants.js                 |      100 |      100 |      100 |      100 |                |
   EventHub.js                  |    91.98 |     84.3 |    93.55 |    92.48 |... 755,811,820 |
   Orderer.js                   |     79.7 |    61.11 |     87.5 |     79.7 |... 290,291,294 |
   Organization.js              |    82.98 |       80 |    86.67 |    84.09 |... 124,125,128 |
   Packager.js                  |     91.3 |    91.67 |      100 |     91.3 |          54,55 |
   Peer.js                      |    93.48 |    81.25 |      100 |    93.48 |    135,142,143 |
   Policy.js                    |    99.03 |    92.16 |      100 |    99.03 |            162 |
   Remote.js                    |      100 |    95.31 |      100 |      100 |                |
   TransactionID.js             |       96 |     87.5 |      100 |       96 |             48 |
   User.js                      |    91.76 |    73.08 |    86.67 |    91.76 |... 220,225,226 |
   api.js                       |      100 |      100 |        0 |      100 |                |
   client-utils.js              |    93.91 |    74.29 |      100 |    93.91 |... 204,217,219 |
   hash.js                      |    45.59 |        0 |    10.53 |    45.59 |... 137,148,157 |
   utils.js                     |    79.57 |    74.56 |    77.14 |    79.57 |... 396,398,457 |
  fabric-client/lib/impl/       |    62.39 |    51.12 |    61.22 |    62.36 |                |
   CouchDBKeyValueStore.js      |    88.46 |       70 |      100 |    88.16 |... 162,175,176 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    84.34 |    80.22 |       75 |    84.34 |... 395,397,398 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |    90.72 |    70.83 |    94.12 |    90.52 |... 390,407,408 |
   bccsp_pkcs11.js              |     17.8 |    16.89 |     8.33 |    18.07 |... 8,1052,1053 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    49.63 |    36.05 |       50 |    51.54 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |    11.69 |        0 |        0 |     12.5 |... 163,164,166 |
  fabric-client/lib/msp/        |    78.36 |    69.01 |    71.43 |     78.7 |                |
   identity.js                  |       94 |       80 |    84.62 |       94 |      42,86,104 |
   msp-manager.js               |    76.36 |    72.73 |    83.33 |    77.36 |... 129,130,159 |
   msp.js                       |    68.18 |    54.17 |    44.44 |    68.18 |... 137,138,180 |
  fabric-client/lib/packager/   |    88.17 |    64.29 |    76.92 |    88.17 |                |
   BasePackager.js              |    77.78 |       50 |    66.67 |    77.78 |... ,95,118,136 |
   Car.js                       |       60 |      100 |        0 |       60 |          23,24 |
   Golang.js                    |      100 |      100 |      100 |      100 |                |
   Node.js                      |    95.65 |       50 |      100 |    95.65 |             77 |
 -------------------------------|----------|----------|----------|----------|----------------|
 All files                      |    81.52 |    69.35 |    78.96 |    81.74 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 81.52% ( 4825/5919 )
 Branches     : 69.35% ( 1867/2692 )
 Functions    : 78.96% ( 544/689 )
 Lines        : 81.74% ( 4799/5871 )
 ================================================================================
 [16:16:33] Finished 'test' after 7.25 min
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 6.3:** (Optional) What I really like about the second end-to-end test is that it cleans itself up really well at the beginning- that is, it will remove any artifacts left running at the end of the prior test, so if you wanted to, you could simply enter *gulp test* again if you'd like to see this for yourself and have several minutes to spare.  If you're pressed for time, skip this step::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (output not shown here)
   . 

**Step 6.4:** You will now clean up after the test completes. You will do this by running only the parts "hidden" within the *gulp test* command execution that did the initial cleanup. I have listed five commands in this step.  Only the middle one, the *gulp* command, does the actual cleanup.  The first two and last two *docker* commands are suggested so that you can see before and after the Docker containers and the chaincode Docker images are removed.  The output of these commands is not shown, but pay attention to it as you enter them::

 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 bcuser@ubuntu16043:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward.
This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
