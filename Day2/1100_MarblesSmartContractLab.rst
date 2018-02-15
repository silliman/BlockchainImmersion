Section 1 - Overview of Hyperledger Fabric Smart Contract installation lab
==========================================================================
In this lab, you will use the Hyperledger Fabric instance that you installed and tested in the previous lab, “Hyperledger Fabric 
installation and verification on IBM Z”.

You will use Docker Compose to bring up a Fabric network in which two organizations will participate.  There will be one orderer 
service for the entire network, and each organization will use its own certificate authority service and have two peer nodes.  Each peer node 
will use CouchDB for its ledger store. Each peer node's CouchDB will run in a separate Docker container.  That makes eleven Docker 
containers, as follows:

*	1 orderer service Docker container
*	2 certificate authority (CA) Docker containers (one for each organization)
*	4 peer node Docker containers  (each of the two organizations has two peers)
*	4 CouchDB Docker containers (each Peer node has its own separate CouchDB ledger store)

You will also bring up a twelfth Docker container that we will call the *CLI* container.  You will use it as a convenience to enter 
Hyperledger Fabric commands targeted to specific peers.  You will see how this is done later in the lab.

The network you bring up will use Transport Layer Security (TLS) which provides secure, encrypted communication between the peer nodes 
and the orderer, just as most real-world implementations will require.

You will **install** a Smart Contract on the peer nodes, **instantiate** the Smart Contract, and **invoke** functions of the Smart
Contract.  I will explain later in the lab the difference between the install and instantiate actions and what each one does.

When you invoke functions of the Smart Contract, some of them will produce transactions on the blockchain and some of them will not.   
*Spoiler alert*:  Functions that create, update or delete ledger data always produce a transaction, while functions that only query ledger data do not.  
 
Section 2	- Description of the subsequent sections in this lab
==============================================================
This section provides a brief description of the subsequent sections in the lab, where you will get hands-on experience with the Hyperledger Fabric command line interface.

1.	You will extract the artifacts necessary to run the lab in Section 3.  All the artifacts necessary for the lab are provided in a zip file.  
2.	You will use Docker Compose to bring up the twelve Docker containers that comprise the Hyperledger Fabric network in Section 4.  You will see that all twelve Docker containers that we mentioned in Section 1 are brought up with a single docker-compose command, and I will explain some of the more interesting bits of what is going on under the covers.
3.	You will create a channel in the Hyperledger Fabric network in Section 5.  In Hyperledger Fabric, each channel is essentially its own blockchain.  
4.	You will instruct each peer node to join the channel in Section 6.  We will join all four Peer nodes to the channel.  Peer nodes can be members of more than one channel, but for our lab we are only creating one channel.
5.	You will define an “anchor” peer for each organization in the channel in Section 7.  An anchor peer for an organization is a peer that is known by all the other organizations in a channel.  Not all peers for an organization need to be known by outside organizations.  Peers not defined as anchor peers are visible only within their own organization.
6.	You will install the Smart Contract, or chaincode, on the peer nodes in Section 8. Installing chaincode simply puts the chaincode executable on the file system of the peer.  It is a necessary step before you execute that chaincode on the peer, but the next step is also required.
7.	You will instantiate the chaincode on the channel in Section 9.  This step is a prerequisite to being able to run chaincode on a channel.  It only needs to be performed on one peer that is a member of the channel.  This causes a transaction to be recorded on the channel’s blockchain to indicate that the chaincode can be run on the channel.
8.	You will invoke functions on the chaincode that will create, read, update and delete (CRUD) data stored on the blockchain in Section 10. If you hear programmers use the word CRUD, unless they are talking about last night’s hockey game, they are probably talking about Creating, Reading, Updating, or Deleting data.   Blocks of transactions in a blockchain are always added (i.e., Created), and they can be Read, but they should never, ever, ever, in normal operations, be Updated or Deleted.   However, although the blocks in a chain are not updated or deleted, the transactions themselves operate on Key/Value pairs that can have all CRUD operations performed on them.  This collection of Key/Value pairs is often referred to as state data. 


 
Section 3 -	Extract the artifacts necessary to run the lab
==========================================================

**Step 3.1:**	Navigate to the home directory by entering *cd ~* (the “tilde” character, i.e., ‘*~*’, represents the user’s home directory in Linux).  
This directory is also usually set in the $HOME environment variable, so *cd $HOME* will also usually get you to your home directory.  
E.g., observe the following commands which illustrate this::
 bcuser@ubuntu16043:~$ cd /usr/lib
 bcuser@ubuntu16043:/usr/lib$ # starting in some random dir
 bcuser@ubuntu16043:/usr/lib$ # bash interprets '#' as starting a comment
 bcuser@ubuntu16043:/usr/lib$ pwd # prints the current directory you are in
 /usr/lib
 bcuser@ubuntu16043:/usr/lib$ cd ~ # will take you to your home directory
 bcuser@ubuntu16043:~$ pwd
 /home/bcuser
 bcuser@ubuntu16043:~$ cd - # takes you back to the previous directory 
 /usr/lib
 bcuser@ubuntu16043:/usr/lib$ echo $HOME # print your HOME environment variable
 /home/bcuser
 bcuser@ubuntu16043:/usr/lib$ cd $HOME # will be the same as cd ~
 bcuser@ubuntu16043:~$ pwd
 /home/bcuser
 bcuser@ubuntu16043:~$
 
**Step 3.2:** Retrieve the zmarbles compressed tarball prepared for this lab with the following command::

 bcuser@ubuntu16043:~$ wget https://raw.githubusercontent.com/silliman/fabric-lab-IBM-Z/master/zmarbles.tar.gz
 ---2018-02-03 11:22:52--  https://raw.githubusercontent.com/silliman/fabric-lab-IBM-Z/master/zmarbles.tar.gz
 Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.200.133
 Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.200.133|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 1532905 (1.5M) [application/octet-stream]
 Saving to: 'zmarbles.tar.gz'

 zmarbles.tar.gz                                   100%[==========================================================================================================>]   1.46M  --.-KB/s    in 0.1s    
 2018-02-03 11:22:53 (12.3 MB/s) - 'zmarbles.tar.gz' saved [1532905/1532905]
 
**Step 3.3:**	List the *zmarbles* directory with this *ls* command::

 bcuser@ubuntu16043:~$ ls zmarbles     
 ls: cannot access 'zmarbles': No such file or directory
 
Don’t panic!  It wasn’t supposed to be there.  It will be after the next step.

**Step 3.4:**	Extract the *zmarbles.tar.gz* file which will create the missing directory (and lots of subdirectories).  
If you are not giddy yet, try tucking the “*v*” switch into the options in the command below.  That is, use *-xzvf* instead of *-xzf*.  
So, enter the commands highlighted below as shown, or by substituting *-xzvf* for *-xzf* in the tar command (the “*v*” is for “*verbose*”)
::

 bcuser@ubuntu16043:~$ tar -xzf zmarbles.tar.gz 
 bcuser@ubuntu16043:~$ ls zmarbles
 base               configtx.yaml       docker-compose-couch.yaml     examples              hostScripts  scripts
 channel-artifacts  crypto-config.yaml  docker-compose-template.yaml  generateArtifacts.sh  marblesUI
 bcuser@ubuntu16043:~$

Congratulations!  You are now ready to get to the hard part of the lab!  Proceed to the next section please.  
 
Section 4	- Bring up the twelve Docker containers that comprise the Hyperledger Fabric network
==============================================================================================

**Step 4.1:**	Change to the *zmarbles* directory with the *cd* command and then list its contents with the *ls* command::

 bcuser@ubuntu16043:~$ cd zmarbles/ 
 bcuser@ubuntu16043:~/zmarbles$ ls -l
 total 52
 drwxr-xr-x  2 bcuser bcuser 4096 Aug 24 15:42 base
 drwxr-xr-x  2 bcuser bcuser 4096 Sep  6 15:42 channel-artifacts
 -rw-r--r--  1 bcuser bcuser 5017 Jun 18  2017 configtx.yaml
 -rw-r--r--  1 bcuser bcuser 3861 Jun 18  2017 crypto-config.yaml
 -rw-r--r--  1 bcuser bcuser 2003 Aug 30 13:47 docker-compose-couch.yaml
 -rw-r--r--  1 bcuser bcuser 6029 Feb  8 16:24 docker-compose-template.yaml
 drwxr-xr-x  3 bcuser bcuser 4096 Jun 18  2017 examples
 -rwxr-xr-x  1 bcuser bcuser 3612 Feb  8 15:45 generateArtifacts.sh
 drwxr-xr-x  2 bcuser bcuser 4096 Oct  1 18:51 hostScripts
 drwxr-xr-x 12 bcuser bcuser 4096 Sep  6 15:43 marblesUI
 drwxr-xr-x  2 bcuser bcuser 4096 Sep  6 12:38 scripts
 bcuser@ubuntu16043:~/zmarbles$
 
**Step 4.2:**	You are going to run a script named *generateArtifacts.sh* that will create some configuration information that is 
necessary to get your Hyperledger Fabric network set up.  There is one optional parameter you may pass to the script, and that is the 
name of the channel you will be creating.  If you do not specify this parameter, the channel name defaults to *mychannel*. You may 
choose to specify your own channel name.  E.g., if you wish to name your channel *Tim*, then you will 
enter *./generateArtifacts.sh Tim* instead of just *./generateArtifacts.sh* as shown in the below snippet.

So, enter just *one* of these two commands (the first one is recommended)::

 source ./generateArtifacts.sh    # will use the default channel name of mychannel
 source ./generateArtifacts.sh yourfancychannelname   # please pick a shorter name for your own sake!
 
**Note:** Only enter one of the above commands.  If you pick your own channel name, it must start with a lowercase character, and only contain lowercase characters, numbers, or the dash ('-') character.  

By the way, if you enter a command and end it with #, everything after the # is considered a comment and is ignored by the shell.  
So, if you see me place comments after any commands you do not have to enter them but if you do, it will not hurt anything.  

Here is output from entering the first command,  which does not specify the channel name and thus accepts the default name of *mychannel*::

 bcuser@ubuntu16043:~/zmarbles$ source ./generateArtifacts.sh  # not all output is shown below
 Using cryptogen -> /home/bcuser/git/src/github.com/hyperledger/fabric/release/linux-s390x/bin/cryptogen

 ##########################################################
 ##### Generate certificates using cryptogen tool #########
 ##########################################################
 unitedmarbles.com
 marblesinc.com

 Using configtxgen -> /home/bcuser/git/src/github.com/hyperledger/fabric/release/linux-s390x/bin/configtxgen
 ##########################################################
 #########  Generating Orderer Genesis block ##############
 ##########################################################
 2018-02-03 11:32:49.608 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-02-03 11:32:49.614 EST [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
 2018-02-03 11:32:49.615 EST [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block

 #################################################################
 ### Generating channel configuration transaction 'channel.tx' ###
 #################################################################
 2018-02-03 11:32:49.649 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-02-03 11:32:49.654 EST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
 2018-02-03 11:32:49.655 EST [common/tools/configtxgen] main -> CRIT 003 Error on outputChannelCreateTx: config update  generation failure: could not parse application to application group: setting up the MSP manager failed: the supplied  identity is not valid: x509: certificate signed by unknown authority (possibly because of "x509: ECDSA verification failure" while trying to verify candidate authority certificate "ca.unitedmarbles.com")

 #################################################################
 #######    Generating anchor peer update for Org0MSP   ##########
 #################################################################
 2018-02-03 11:32:49.689 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-02-03 11:32:49.695 EST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
 2018-02-03 11:32:49.695 EST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

 #################################################################
 #######    Generating anchor peer update for Org1MSP   ##########
 #################################################################
 2018-02-03 11:32:49.729 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-02-03 11:32:49.734 EST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
 2018-02-03 11:32:49.734 EST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update


This script calls two Hyperledger Fabric utilites- *cryptogen*, which creates security material (certificates and keys) 
and *configtxgen* (Configuration Transaction Generator), which is called four times, to create four things::

1.	An **orderer genesis block** – this will be the first block on the orderer’s system channel. The location of this block is 
specified to the Orderer when it is started up via the ORDERER_GENERAL_GENESISFILE environment variable.

2.	A **channel transaction** – later in the lab, this is sent to the orderer and will cause a new channel to be created when you run 
the **peer channel create** command.

3.	An **anchor peer update** for Org0MSP.  An anchor peer is a peer that is set up so that peers from other organizations may 
communicate with it.  The concept of anchor peers allows an organization to create multiple peers, perhaps to provide extra capacity 
or throughput or resilience (or all the above) but not have to advertise this to outside organizations.

4.	An anchor peer update for Org1MSP.   You will perform the anchor peer updates for both Org0MSP and Org1MSP later in the lab 
via **peer channel create** commands.

**Step 4.3:**	Issue the following command which will show you all files that have been modified in the last 15 minutes::

 bcuser@ubuntu16043:~/zmarbles$ find . -name '*' -mmin -15
 
 ./channel-artifacts/Org0MSPanchors.tx
 ./channel-artifacts/Org1MSPanchors.tx
 ./channel-artifacts/genesis.block
 ./channel-artifacts/channel.tx
 ./docker-compose.yaml
   .
   .  # lots of cryptographic material in crypto-config/
   .

These are the files that have been created from running the *generateArtifacts.sh* script in the previous step. You will see later 
how some of them are used.

**Step 4.4:**	You are going to look inside the Docker Compose configuration file a little bit.   Enter the following command::

 vi -R docker-compose.yaml  

You can enter ``Ctrl-f`` to scroll forward in the file and ``Ctrl-b`` to scroll back in the file.  The *-R* flag opens the file in 
read-only mode, so if you accidentally change something in the file, it’s okay.  It will not be saved.

The statements within *docker-compose.yaml* are in a markup language called *YAML*, which stands 
for *Y*\ et *A*\ nother *M*\ arkup *L*\ anguage.  (Who says nerds do not have a sense of humor).  We will go over some highlights here.

There are twelve “services”, or Docker containers, defined within this file.  They all start in column 3 and have several statements
to describe them.  For example, the first service defined is **ca0**, and there are *image*, *environment*, *ports*, *command*, *volumes*, and 
*container_name* statements that describe it.  If you scroll down in the file with ``Ctrl-f`` you will see all the services.  Not 
every service has the same statements describing it.

The twelve services are:

**ca0** – The certificate authority service for “Organization 0” (unitedmarbles.com)

**ca1** – The certificate authority service for “Organization 1” (marblesinc.com)

**orderer.blockchain.com** – The single ordering service that both organizations will use

**peer0.unitedmarbles.com** – The first peer node for “Organization 0”	

**peer1.unitedmarbles.com** – The second peer node for “Organization 0”	

**peer0.marblesinc.com** – The first peer node for “Organization 1”	

**peer1.marblesinc.com** – The second peer node for “Organization 1”	

**couchdb0** – The CouchDB server for peer0.unitedmarbles.com  

**couchdb1** – The CouchDB server for peer1.unitedmarbles.com  

**couchdb2** – The CouchDB server for peer0.marblesinc.com

**couchdb3** – The CouchDB server for peer1.marblesinc.com

**cli** – The Docker container from which you will enter Hyperledger Fabric command line interface (CLI) commands targeted 
towards a peer node.

I will describe how several statements work within the file, but time does not permit me to address every single line in the file!

*image* statements define which Docker image file the Docker container will be created from.  Basically, the Docker image file is a 
static file that, once created, is read-only.  A Docker container is based on a Docker image, and any changes to the file system 
within a Docker container are stored within the container.  So, multiple Docker containers can be based on the same Docker image, 
and each Docker container keeps track of its own changes.  For example, the containers built for the **ca0** and **ca1** service will 
be based on the *hyperledger/fabric-ca:latest* Docker image because they both have this statement in their definition::

        image: hyperledger/fabric-ca    

*environment* statements define environment variables that are available to the Docker container.  The Hyperledger Fabric processes 
make ample use of environment variables.  In general, you will see that the certificate authority environment variables start with 
*FABRIC_CA*, the orderer’s environment variables start with *ORDERER_GENERAL*, and the peer node’s environment variables start with 
*CORE*.  These variables control behavior of the Hyperledger Fabric code, and in many cases, will override values that are specified 
in configuration files. Notice that all the peers and the orderer have an environment variable to specify that TLS is 
enabled-   *CORE_PEER_TLS_ENABLED=true* for the peers and *ORDERER_GENERAL_TLS_ENABLED=true* for the orderer.  You will notice there 
are other TLS-related variables to specify private keys, certificates and root certificates.

*ports* statements map ports on our Linux on IBM Z host to ports within the Docker container.  The syntax is *<host port>:<Docker 
container port>*.  For example, the service for **ca1** has this port statement::
 
     ports:
       - "8054:7054"

This says that port 7054 in the Docker container for the **ca1** node will be mapped to port 8054 on your Linux on IBM Z host.   This 
is how you can run two CA nodes in two Docker containers and four peer nodes in four Docker containers and keep things straight-  
within each CA node they are both using port 7054, and within each peer node Docker container, they are all using port 7051 for the 
same thing, but if you want to get to one of the peers from your host or even the outside world, you would target the appropriate 
host-mapped port. **Note:** To see the port mappings for the peers you have to look in *base/docker-compose.yaml*.  See if you can 
figure out why.

*container_name* statements are used to create hostnames that the Docker containers spun up by the docker-compose command use to 
communicate with each other.  A separate, private network will be created by Docker where the 12 Docker containers can communicate 
with each other via the names specified by *container_name*.  So, they do not need to worry about the port mappings from the *ports* 
statements-  those are used for trying to get to the Docker containers from outside the private network created by Docker.

*volumes* statements are used to map file systems on the host to file systems within the Docker container.  Just like with ports, the 
file system on the host system is on the left and the file system name mapped within the Docker container is on the right. For 
example, look at this statement from the **ca0** service::
 
     volumes:
       - ./crypto-config/peerOrganizations/unitedmarbles.com/ca/:/etc/hyperledger/fabric-ca-server-config

The security-related files that were created from the previous step where you ran *generateArtifacts.sh* were all within 
the *crypto-config* directory on your Linux on IBM Z host.  The prior *volumes* statement is how this stuff is made accessible to the 
**ca1** service that will run within the Docker container.   Similar magic is done for the other services as well, except for 
the CouchDB services.

*extends* statements are used by the peer nodes.  What this does is merge in other statements from another file.  For example, you 
may notice that the peer nodes do not contain an images statement.  How does Docker know what Docker image file to base the 
container on?  That is defined in the file, *base/peer-base.yaml*, specified in the *extends* section of *base/docker-compose.yaml*, 
which is specified in the *extends* section of *docker-compose.yaml* for the peer nodes.

*command* statements define what command is run when the Docker container is started.  This is how the actual Hyperledger Fabric 
processes get started.  You can define default commands when you create the Docker image.  This is why you do not see *command*
statements for the **cli** service or for the CouchDB services.   For the peer nodes, the command statement is specified in the 
*base/peer-base.yaml* file.

*working_dir* statements define what directory the Docker container will be in when its startup commands are run.  Again, defaults 
for this can be defined when the Docker image is created. 

When you are done reviewing the *docker-compose.yaml* file, exit the *vi* session by typing ``:q!``  (that’s “colon”, “q”, 
“exclamation point”) which will exit the file and discard any changes you may have accidentally made while browsing through the file.  
If ``:q!`` doesn’t work right away, you may have to hit the escape key first before trying it.  If that still doesn’t work, ask an 
instructor for help-  *vi* can be tricky if you are not used to it.

If you would like to see what is in the *base/docker-compose-base.yaml* and *base/peer-base.yaml* files I mentioned, take a quick 
peek with ``vi -R base/docker-compose-base.yaml`` and ``vi -R base/peer-base.yaml`` and exit with the ``:q!`` key sequence when you 
have had enough.

**Step 4.5:**	Start the Hyperledger Fabric network by entering the command shown below::

 bcuser@ubuntu16043:~/zmarbles$ docker-compose up -d
 Creating network "zmarbles_default" with the default driver
 Creating couchdb0 ... 
 Creating couchdb1 ... 
 Creating orderer.blockchain.com ... 
 Creating couchdb0
 Creating couchdb1
 Creating orderer.blockchain.com
 Creating couchdb2 ... 
 Creating ca_Org0 ... 
 Creating couchdb2
 Creating couchdb3 ... 
 Creating ca_Org0
 Creating ca_Org1 ... 
 Creating couchdb3
 Creating ca_Org1 ... done
 Creating peer0.unitedmarbles.com ... 
 Creating peer0.marblesinc.com ... 
 Creating peer1.marblesinc.com ... 
 Creating peer1.unitedmarbles.com ... 
 Creating peer1.marblesinc.com
 Creating peer0.marblesinc.com
 Creating peer0.unitedmarbles.com
 Creating peer0.marblesinc.com ... done
 Creating cli ... 
 Creating cli ... done

**Step 4.6:**	Verify that all twelve services are *Up* and none of them say *Exited*.  The *Exited* status means something went 
wrong, and you should check with an instructor for help if you see any of them in *Exited* status.

If, however, all twelve of your Docker containers are in *Up* status, as in the output below, you are ready to proceed to the next 
section::

 bcuser@ubuntu16043:~/zmarbles$ docker ps -a
 CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                              NAMES
 acd1f96d8807        hyperledger/fabric-tools     "bash"                   39 seconds ago      Up 38 seconds                                                                                   cli
 c37eaf50e8c8        hyperledger/fabric-peer      "peer node start"        41 seconds ago      Up 39 seconds       0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.marblesinc.com
 5b8935302c61        hyperledger/fabric-peer      "peer node start"        41 seconds ago      Up 39 seconds       0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.unitedmarbles.com
 f3cb324af064        hyperledger/fabric-peer      "peer node start"        41 seconds ago      Up 39 seconds       0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.marblesinc.com
 8b5d1e843535        hyperledger/fabric-peer      "peer node start"        41 seconds ago      Up 39 seconds       0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.unitedmarbles.com
 d6a065e923b3        hyperledger/fabric-couchdb   "tini -- /docker-e..."   43 seconds ago      Up 41 seconds       4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 de256b18bdb2        hyperledger/fabric-orderer   "orderer"                43 seconds ago      Up 40 seconds       0.0.0.0:7050->7050/tcp                                                      orderer.blockchain.com
 cfcc713084b6        hyperledger/fabric-ca        "sh -c 'fabric-ca-..."   43 seconds ago      Up 41 seconds       0.0.0.0:8054->7054/tcp                                                      ca_Org1
 f7e26311249a        hyperledger/fabric-couchdb   "tini -- /docker-e..."   43 seconds ago      Up 41 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 110b263777be        hyperledger/fabric-couchdb   "tini -- /docker-e..."   43 seconds ago      Up 41 seconds       4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 debe8e24f371        hyperledger/fabric-couchdb   "tini -- /docker-e..."   43 seconds ago      Up 42 seconds       4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 5234d365344b        hyperledger/fabric-ca        "sh -c 'fabric-ca-..."   43 seconds ago      Up 42 seconds       0.0.0.0:7054->7054/tcp           
 bcuser@ubuntu16043:~/zmarbles$ 

Section 5	- Create a channel in the Hyperledger Fabric network
==============================================================
In a Hyperledger Fabric v1.1.0 network, multiple channels can be created.  Each channel can have its own policies for things such as 
requirements for endorsement and what organizations may join the channel.  This allows for a subset of network participants to 
participate in their own channel.  

Imagine a scenario where OrgA, OrgB and OrgC are three organizations participating in the network. You could set up a channel in which 
all three organizations participate.   You could also set up a channel where only OrgA and OrgB participate.   In this case, the peers 
in OrgC would not see the transactions occurring in that channel.    OrgA could participate in another channel with only OrgC, in 
which case OrgB does not have visibility.  And so on.  

You could create channels with the same participants, but have different policies.  For example, perhaps one channel with OrgA, OrgB, 
and OrgC could require all three organizations to endorse a transaction proposal, but another channel with OrgA, OrgB and OrgC could 
require just two, or even just one, of the three organizations to endorse a transaction proposal.

The decision on how many channels to create and what policies they have will usually be driven by the requirements of the particular 
business problem being solved.

**Step 5.1:**	Access the *cli* Docker container::

 bcuser@ubuntu16043:~/zmarbles$ docker exec -it cli bash
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer#ic/peer#

Observe that your command prompt changes when you enter the Docker container’s shell.

The *docker exec* command runs a command against an existing Docker container.  The *-it* flags basically work together to say, 
“we want an interactive terminal session with this Docker container”.  *cli* is the name of the Docker container (this came from the 
*container_name* statement in the *docker-compose.yaml* file for the *cli* service).  *bash* is the name of the command you want to 
enter.   In other words, you are entering a Bash shell within the *cli* Docker container.  For most of the rest of the lab, you will be 
entering commands within this Bash shell.

Instead of working as user *bcuser* on the ubuntu16043 server in the *~/zmarbles* directory, you are now inside the Docker container with 
ID *acd1f96d8807* (your ID will differ), working in the */opt/gopath/src/github.com/hyperledger/fabric/peer* directory.  It is no 
coincidence that that directory is the value of the *working_dir* statement for the *cli* service in your *docker-compose.yaml* file.

**Step 5.2:** Read on to learn about a convenience script to point to a particular peer from the *cli* Docker container. A convenience 
script named *setpeer* is provided within the *cli* container that is in the *scripts* subdirectory of your current working directory. 
This script will set the environment variables to the values necessary to point to a particular peer.   The script takes two 
arguments.  This first argument is either 0 or 1 for Organization 0 or Organization 1 respectively, and the second argument is for 
either Peer 0 or Peer 1 of the organization selected by the first argument.   Therefore, throughout the remainder of this lab, before
sending commands to a peer, you will enter one of the following four valid combinations, depending on which peer you want to run the 
command on:

*source scripts/setpeer 0 0*   # to target Org 0, peer 0  (peer0.unitedmarbles.com)

*source scripts/setpeer 0 1*   # to target Org 0, peer 1  (peer1.united marbles.com)

*source scripts/setpeer 1 0*   # to target Org 1, peer 0  (peer0.marblesinc.com)

*source scripts/setpeer 1 1*   # to target Org 1, peer 1  (peer1.marblesinc.com)

**Step 5.3:** Choose your favorite peer and use one of the four *source scripts/setpeer* commands listed in the prior step.   Although 
you are going to join all four peers to our channel, you only need to issue the channel creation command once.  You can issue it from 
any of the four peers, so pick your favorite peer and issue the source command.  In this screen snippet, I have chosen Org 1, peer 1::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer1.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.marblesinc.com:7051
 root@fbe81505b8a2:/opt/gopath/src/github.com/hyperledger/fabric/peer#

The last environment variable listed, *CORE_PEER_ADDRESS*, determines to which peer your commands will be routed.  

**Step 5.4:**	The Hyperledger Fabric network is configured to require TLS, so when you enter your peer commands, you need to add a 
flag that indicates TLS is enabled, and you need to add an argument that points to the root signer certificate of the certificate 
authority for the orderer service.

What you are going to do next is set an environment variable that will specify these arguments for you, and that way you will not 
have to type out the hideously long path for the CA’s root signer certificate every time. Enter this command exactly as shown::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# export FABRIC_TLS="--tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/blockchain.com/orderers/orderer.blockchain.com/msp/tlscacerts/tlsca.blockchain.com-cert.pem"

**Note:** This above is intended to be entered without any line breaks-  if you are cutting and pasting this, depending on the medium 
you are using, line breaks may have been introduced.  There only needs to be one space between the **--cafile** and the long path name 
to the CA certificate file.  I apologize for the complexity of this command, but once you get it right, you won’t have to hassle with 
it again as long as you do not exit the cli Docker container’s bash shell.

**Step 5.5:**	Verify that you entered the FABRIC_TLS environment variable correctly.  (Note that when setting, or exporting, the variable 
you did not prefix the variable with a “$”, but when referencing it you do prefix it with a “$”.   Your output should look like this::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $FABRIC_TLS 
 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/blockchain.com/orderers/orderer.blockchain.com/msp/cacerts/ca.blockchain.com-cert.pem

**Step 5.6:** Now enter this command::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel create -o orderer.blockchain.com:7050  -f channel-artifacts/channel.tx  $FABRIC_TLS -c $CHANNEL_NAME
 
If this goes well, after a few seconds, you are going to see a whole bunch of gibberish and then the last line before you get 
your command prompt back will end with the reassuring phrase, “Exiting…..”.   Here is a screen snippet of my output, and you can feel good if your gibberish looks like my gibberish.  Trust me, it is working as coded!
::

 2018-02-03 19:00:52.229 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:00:52.229 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:00:52.233 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:00:52.234 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
 2018-02-03 19:00:52.234 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
 2018-02-03 19:00:52.234 UTC [msp] GetLocalMSP -> DEBU 006 Returning existing local MSP
 2018-02-03 19:00:52.234 UTC [msp] GetDefaultSigningIdentity -> DEBU 007 Obtaining default signing identity
 2018-02-03 19:00:52.234 UTC [msp/identity] Sign -> DEBU 008 Sign: plaintext: 0A9A060A074F7267314D5350128E062D...53616D706C65436F6E736F727469756D 
 2018-02-03 19:00:52.234 UTC [msp/identity] Sign -> DEBU 009 Sign: digest: 6993538FE97DCEE2CC6A6A11D3BAA40993BC188D8E6560E3BFF1424C5B87CC6D 
 2018-02-03 19:00:52.235 UTC [msp] GetLocalMSP -> DEBU 00a Returning existing local MSP
 2018-02-03 19:00:52.235 UTC [msp] GetDefaultSigningIdentity -> DEBU 00b Obtaining default signing identity
 2018-02-03 19:00:52.235 UTC [msp] GetLocalMSP -> DEBU 00c Returning existing local MSP
 2018-02-03 19:00:52.235 UTC [msp] GetDefaultSigningIdentity -> DEBU 00d Obtaining default signing identity
 2018-02-03 19:00:52.235 UTC [msp/identity] Sign -> DEBU 00e Sign: plaintext: 0AD1060A1508021A0608E48DD8D30522...D9904606232EA381683E6CFDB4EC5BA5 
 2018-02-03 19:00:52.235 UTC [msp/identity] Sign -> DEBU 00f Sign: digest: 72AD12D8676305CB9DC38B46892B1785278C10367D268DEFA54919AD4B1D21DB 
 2018-02-03 19:00:52.277 UTC [msp] GetLocalMSP -> DEBU 010 Returning existing local MSP
 2018-02-03 19:00:52.277 UTC [msp] GetDefaultSigningIdentity -> DEBU 011 Obtaining default signing identity
 2018-02-03 19:00:52.277 UTC [msp] GetLocalMSP -> DEBU 012 Returning existing local MSP
 2018-02-03 19:00:52.277 UTC [msp] GetDefaultSigningIdentity -> DEBU 013 Obtaining default signing identity
 2018-02-03 19:00:52.277 UTC [msp/identity] Sign -> DEBU 014 Sign: plaintext: 0AD1060A1508021A0608E48DD8D30522...3D5D01C6227812080A021A0012021A00 
 2018-02-03 19:00:52.277 UTC [msp/identity] Sign -> DEBU 015 Sign: digest: D868AEB71531A184546070C6E8289C53797ABF74373AE63551CDDDC4032F158D 
 2018-02-03 19:00:52.279 UTC [channelCmd] readBlock -> DEBU 016 Got status: &{NOT_FOUND}
 2018-02-03 19:00:52.279 UTC [msp] GetLocalMSP -> DEBU 017 Returning existing local MSP
 2018-02-03 19:00:52.279 UTC [msp] GetDefaultSigningIdentity -> DEBU 018 Obtaining default signing identity
 2018-02-03 19:00:52.294 UTC [channelCmd] InitCmdFactory -> INFO 019 Endorser and orderer connections initialized
 2018-02-03 19:00:52.495 UTC [msp] GetLocalMSP -> DEBU 01a Returning existing local MSP
 2018-02-03 19:00:52.495 UTC [msp] GetDefaultSigningIdentity -> DEBU 01b Obtaining default signing identity
 2018-02-03 19:00:52.495 UTC [msp] GetLocalMSP -> DEBU 01c Returning existing local MSP
 2018-02-03 19:00:52.495 UTC [msp] GetDefaultSigningIdentity -> DEBU 01d Obtaining default signing identity
 2018-02-03 19:00:52.495 UTC [msp/identity] Sign -> DEBU 01e Sign: plaintext: 0AD1060A1508021A0608E48DD8D30522...649DD053C36012080A021A0012021A00 
 2018-02-03 19:00:52.495 UTC [msp/identity] Sign -> DEBU 01f Sign: digest: BA9343FF4D9AC77B8CB9E53DE151B10A9C5B82E74A45A1938D55BCD9B92B2A99 
 2018-02-03 19:00:52.499 UTC [channelCmd] readBlock -> DEBU 020 Received block: 0
 2018-02-03 19:00:52.499 UTC [main] main -> INFO 021 Exiting.....


Proceed to the next section where you will join each peer to the channel.
 
Section 6	- Instruct each peer node to join the channel
=======================================================

In the last section, you issued the *peer channel create* command from one of the peers.   Now any peer that you want to join the 
channel may join- you will issue the *peer channel join* command from each peer.

For a peer to be eligible to join a channel, it must be a member of an organization that is authorized to join the channel.  When you 
created your channel, you authorized *Org0MSP* and *Org1MSP* to join the channel.  Each of your four peers belongs to one of those two 
organizations- two peers for each one- so they will be able to join successfully.   If someone from an organization other than *Org0MSP* 
or *Org1MSP* attempted to join their peers to this channel, the attempt would fail.

You are going to repeat the following steps for each of the four peer nodes, in order to show that the peer successfully joined the 
channel:

1.	Use the *scripts/setpeer* script to point the CLI to the peer

2.	Use the *peer channel list* command to show that the peer is not joined to any channels

3.	Use the *peer channel join* command to join the peer to your channel

4.	Use the *peer channel list* command again to see that the peer has joined your channel

**Step 6.1:**	Point the *cli* to *peer0* for *Org0MSP*::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051

**Step 6.2:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:04:43.380 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:04:43.380 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:04:43.384 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:04:43.384 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext:  0AA0070A5C08031A0C08CB8FD8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:04:43.384 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 76730D606028A05BCA26F785E86B2F2D4A5B9F7FE13A2A764F6ADB0F8D1701E0 
 Channels peers has joined: 
 2018-02-03 19:04:43.388 UTC [main] main -> INFO 006 Exiting.....
 

**Step 6.3:** Issue *peer channel join -b $CHANNEL_NAME.block* to join the channel you set up when you ran *generateArtifacts.sh* a little while ago.  Among the many things that script did, it exported an environment variable named $CHANNEL_NAME set to the channel name you specified (or *mychannel* if you did not specify your own name), and then the Docker Compose file for is set up to pass this environment variable to the *cli* container.  If you are still on the happy path, your output will look similar to 
this::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-02-03 19:09:23.782 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:09:23.782 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:09:23.793 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:09:23.794 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0AA0070A5C08011A0C08E391D8D30510...2A57C968A2081A080A000A000A000A00 
 2018-02-03 19:09:23.794 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 3570F6942D59516B918135D81D54EADAA122B86ADFB13D272F78A4D0E798600A 
 2018-02-03 19:09:23.876 UTC [channelCmd] executeJoin -> INFO 006 Successfully submitted proposal to join channel
 2018-02-03 19:09:23.876 UTC [main] main -> INFO 007 Exiting.....
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# 

**Step 6.4:**	Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:10:34.736 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:10:34.736 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:10:34.739 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:10:34.739 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0AA0070A5C08031A0C08AA92D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:10:34.739 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 8854DFF1F1992516D7540C2E983EDF8640FB3018B62FA7E32CEFBB8ADFC67AE9 
 Channels peers has joined: 
 mychannel
 2018-02-03 19:10:34.742 UTC [main] main -> INFO 006 Exiting.....

**Step 6.5:**	Point the *cli* to *peer1* for *Org0MSP*::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer1.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.unitedmarbles.com:7051

**Step 6.6:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:11:46.612 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:11:46.612 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:11:46.618 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:11:46.618 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0AA0070A5C08031A0C08F292D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:11:46.618 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 92CBFF36A48B25FE9AD4DC1CFF72D00AC0B1986DBB0B55EC1E1673EAF84D9F24 
 Channels peers has joined: 
 2018-02-03 19:11:46.625 UTC [main] main -> INFO 006 Exiting.....

**Step 6.7:**	Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. Your output should look 
similar to this::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-02-03 19:12:33.205 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:12:33.205 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:12:33.208 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:12:33.208 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A9F070A5B08011A0B08A193D8D30510...2A57C968A2081A080A000A000A000A00 
 2018-02-03 19:12:33.208 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: F4DA59A7D6506C38EF4DE5B5721A782820EA7A5468B2704427563AE417C095DF 
 2018-02-03 19:12:33.305 UTC [channelCmd] executeJoin -> INFO 006 Successfully submitted proposal to join channel
 2018-02-03 19:12:33.305 UTC [main] main -> INFO 007 Exiting.....
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer#

**Step 6,8:** Repeat the *peer channel list* command and now you should see your channel listed::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:13:13.191 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:13:13.191 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:13:13.195 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:13:13.195 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A9F070A5B08031A0B08C993D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:13:13.195 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 027FF6ABC0972DD3D2039B739641768B98909A016E3C85496D39DA90951CB6BA 
 Channels peers has joined: 
 mychannel
 2018-02-03 19:13:13.197 UTC [main] main -> INFO 006 Exiting.....


**Step 6.9:**	Point the *cli* to *peer0* for *Org1MSP*::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer0.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.marblesinc.com:7051

**Step 6.10:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:14:12.574 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:14:12.574 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:14:12.578 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:14:12.578 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A98070A5C08031A0C088494D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:14:12.578 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 2EF82AFA8C0AE42E8059EB47AF10F9B047752654D975CDD0A3FC3CF455F4DD6C 
 Channels peers has joined: 


**Step 6.11:** Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. Your output should look 
similar to this::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-02-03 19:14:12.581 UTC [main] main -> INFO 006 Exiting.....
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block
 2018-02-03 19:16:34.073 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:16:34.073 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:16:34.077 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:16:34.077 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A97070A5B08011A0B089295D8D30510...2A57C968A2081A080A000A000A000A00 
 2018-02-03 19:16:34.077 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 3E75711B16C7E3DD950EFAFAFBF74D792574A999A600CF37B8043211A5CA5499 
 2018-02-03 19:16:34.174 UTC [channelCmd] executeJoin -> INFO 006 Successfully submitted proposal to join channel
 2018-02-03 19:16:34.174 UTC [main] main -> INFO 007 Exiting.....
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# 

**Step 6.12:** Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:17:10.501 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:17:10.501 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:17:10.505 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:17:10.505 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A98070A5C08031A0C08B695D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:17:10.505 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: F24FF9434700F5F221101850E9E93029B10BB3E52293BF8E0C51184C26E14B79 
 Channels peers has joined: 
 mychannel
 2018-02-03 19:17:10.508 UTC [main] main -> INFO 006 Exiting.....

**Step 6.13:**	Point the *cli* to *peer1* for *Org1MSP*::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer1.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.marblesinc.com:7051

The output from this should be familiar to you by now so from now on I will not bother showing it anymore in the remainder of these 
lab instructions.

**Step 6.14:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:18:13.529 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:18:13.529 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:18:13.532 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:18:13.533 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A98070A5C08031A0C08F595D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:18:13.533 UTC [msp/identity] Sign -> DEBU 005 Sign: digest:  287857C7940115C1264D513117FA47B6C4DF1566D3A908E6C48BFA5CFA0E64E7 
 Channels peers has joined: 
 2018-02-03 19:18:13.536 UTC [main] main -> INFO 006 Exiting.....

**Step 6.15:** Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. (Am I being redundant? Am I repeating myself? Am I saying the same thing over and over again?) Your output should look 
similar to this::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-02-03 19:20:00.954 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:20:00.954 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:20:00.957 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:20:00.957 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A98070A5C08011A0C08E096D8D30510...2A57C968A2081A080A000A000A000A00 
 2018-02-03 19:20:00.958 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 89C220C75B16BEE0B9CFBD638C575562D8AE241D80743DED26199C500277F6FA 
 2018-02-03 19:20:01.050 UTC [channelCmd] executeJoin -> INFO 006 Successfully submitted proposal to join channel
 2018-02-03 19:20:01.050 UTC [main] main -> INFO 007 Exiting.....
 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer#

**Step 6.16:**	Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-02-03 19:20:45.686 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:20:45.686 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:20:45.690 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-02-03 19:20:45.690 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A98070A5C08031A0C088D97D8D30510...631A0D0A0B4765744368616E6E656C73 
 2018-02-03 19:20:45.690 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: 3FBD9C993A7EF8300B65A2DB13FEFD3C8FA565E091C0AE8407AC82EE8B49DC71 
 Channels peers has joined: 
 mychannel
 2018-02-03 19:20:45.693 UTC [main] main -> INFO 006 Exiting.....

 
Section 7	- Define an “anchor” peer for each organization in the channel
========================================================================
An anchor peer for an organization is a peer that can be known by all the other organizations in a channel.  Not all peers for an 
organization need to be known by outside organizations.  Peers not defined as anchor peers are visible only within their own 
organization.

In a production environment, an organization will typically define more than one peer as an anchor peer for availability and 
resilience. In our lab, we will just define one of the two peers for each organization as an anchor peer.

The definition of an anchor peer took place back in section 4 when you ran the *generateArtifacts.sh* script.  Two of the output files 
from that step were *Org0MSPanchors.tx* and *Org1MSPanchors.tx.*  These are input files to define the anchor peers for Org0MSP and 
Org1MSP respectively.  After the channel is created, each organization needs to run this command.  You will do that now-  this process 
is a little bit confusing in that the command to do this starts with *peer channel create …* but the command will actually *update* the 
existing channel with the information about the desired anchor peer.  Think of *peer channel create* here as meaning, “create an update 
transaction for a channel”.

Issue the following commands which will define the two anchor peers::

 source scripts/setpeer 0 0   # to switch to Peer 0 for Org0MSP
 peer channel create -o orderer.blockchain.com:7050 -f channel-artifacts/Org0MSPanchors.tx $FABRIC_TLS -c $CHANNEL_NAME 
 source scripts/setpeer 1 0   # to switch to Peer 0 for Org1MSP
 peer channel create -o orderer.blockchain.com:7050 -f channel-artifacts/Org1MSPanchors.tx $FABRIC_TLS -c $CHANNEL_NAME
 
Section 8	- Install the chaincode on the peer nodes
===================================================

Installing chaincode on the peer nodes puts the chaincode binary executable on a peer node. If you want the peer to be an endorser on a 
channel for a chaincode, then you must install the chaincode on that peer.  If you only want the peer to be a committer on a channel 
for a chaincode, then you do not have to install the chaincode on that peer.  In this section, you will install the chaincode on two of 
your peers.

**Step 8.1:** Enter ``source scripts/setpeer 0 0`` to switch to Peer0 in Org0MSP.

**Step 8.2:**	Install the marbles chaincode on Peer0 in Org0MSP. You are looking for a message near the end of the output similar to what 
is shown here::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles 
 2018-02-03 19:23:54.061 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:23:54.061 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:23:54.061 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
 2018-02-03 19:23:54.061 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
 2018-02-03 19:23:54.061 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode disabled
 2018-02-03 19:23:54.099 UTC [golang-platform] getCodeFromFS -> DEBU 006 getCodeFromFS github.com/hyperledger/fabric/examples/chaincode/go/marbles
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 007 Discarding GOROOT package bytes
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 008 Discarding GOROOT package encoding/json
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 009 Discarding GOROOT package errors
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 00a Discarding GOROOT package fmt
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 00b Discarding provided package github.com/hyperledger/fabric/core/chaincode/shim
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 00c Discarding provided package github.com/hyperledger/fabric/protos/peer
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 00d Discarding GOROOT package strconv
 2018-02-03 19:23:54.179 UTC [golang-platform] func1 -> DEBU 00e Discarding GOROOT package strings
 2018-02-03 19:23:54.179 UTC [golang-platform] GetDeploymentPayload -> DEBU 00f done
 2018-02-03 19:23:54.179 UTC [container] WriteFileToPackage -> DEBU 010 Writing file to tarball: src/github.com/hyperledger/fabric/examples/chaincode/go/marbles/lib.go
 2018-02-03 19:23:54.180 UTC [container] WriteFileToPackage -> DEBU 011 Writing file to tarball: src/github.com/hyperledger/fabric/examples/chaincode/go/marbles/marbles.go
 2018-02-03 19:23:54.181 UTC [container] WriteFileToPackage -> DEBU 012 Writing file to tarball: src/github.com/hyperledger/fabric/examples/chaincode/go/marbles/read_ledger.go
 2018-02-03 19:23:54.181 UTC [container] WriteFileToPackage -> DEBU 013 Writing file to tarball: src/github.com/hyperledger/fabric/examples/chaincode/go/marbles/write_ledger.go
 2018-02-03 19:23:54.182 UTC [msp/identity] Sign -> DEBU 014 Sign: plaintext: 0A9F070A5B08031A0B08CA98D8D30510...C7CFFF060000FFFF5004329000800000 
 2018-02-03 19:23:54.182 UTC [msp/identity] Sign -> DEBU 015 Sign: digest: 58F7D117A1CC8BB5AEDCC7CD0F35A9A33D10717E4EAA48BDAA7A38722AAB4650 
 2018-02-03 19:23:54.199 UTC [chaincodeCmd] install -> DEBU 016 Installed remotely response:<status:200 payload:"OK" > 
 2018-02-03 19:23:54.199 UTC [main] main -> INFO 017 Exiting.....


**Step 8.3:** Enter ``source scripts/setpeer 1 0`` to switch to Peer0 in Org1MSP.

**Step 8.4:** Enter 
::
 peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles 

which will install the marbles chaincode on Peer0 in Org1MSP.  You should receive messages similar to what you received in *step 8.2*.

An interesting thing to note is that for the *peer chaincode install* command you did not need to specify the $FABRIC_TLS environment 
variable.  This is because this operation does not cause the peer to communicate with the orderer. Also, you did not need to specify the $CHANNEL_NAME environment variable.  This is because the *peer chaincode install* command only installs the chaincode on the peer node.  You only need to do this once per peer.  That is, even if you wanted to install the same chaincode on multiple channels on a peer, you only install the chaincode once on that peer.

Installing chaincode on a peer is a necessary step, but not the only step needed, in order to execute chaincode on that peer.  The 
chaincode must also be instantiated on a channel that the peer participates in.  You will do that in the next section.
 
Section 9	- Instantiate the chaincode on the channel
====================================================

In the previous section, you installed chaincode on two of your four peers.  Chaincode installation is a peer-level operation.  
Chaincode instantiation, however, is a channel-level operation.  It only needs to be performed once on the channel, no matter how many 
peers have joined the channel.

Chaincode instantiation causes a transaction to occur on the channel, so even if a peer on the channel does not have the chaincode 
installed, it will be made aware of the instantiate transaction, and thus be aware that the chaincode exists and be able to commit 
transactions from the chaincode to the ledger-  it just would not be able to endorse a transaction on the chaincode.

**Step 9.1:**	You want to stay signed in to the *cli* Docker container, however, you will also want to issue some Docker commands from your 
Linux on IBM Z host, so at this time open up a second PuTTY session and sign in to your Linux on IBM Z host.   For the remainder of 
this lab, I will refer to the session where you are in the *cli* Docker container as *PuTTY Session 1*, and this new session where you 
are at the Linux on IBM Z host as *PuTTY Session 2*.

**Step 9.2:**	You are going to confirm that you do not have any chaincode Docker images created, nor any Docker chaincode containers 
running currently, by issuing several Docker commands from PuTTY Session 2.

Enter ``docker images`` and observe that all of your images begin with *hyperledger*.  If your output screen is “too busy”, try 
entering ``docker images dev-*`` and you should see very little output except for some column headings.   This will show only those 
images that begin with *dev-\**, of which there should not be any at this point in the lab.

Now do essentially the same thing with *docker ps*.   Enter ``docker ps`` and you should see all of the Docker containers for the 
Hyperledger Fabric processes and CouchDB, but no chaincode-related Docker containers.  Entering ``docker ps | grep -v hyperledger`` will 
make this fact stand out more as you should only see column headers in your output. (The *-v* flag for *grep* says “do not show me 
anything that contains the string “hyperledger”).

Now that you have established that you have no chaincode-related Docker images or containers present, try to instantiate the chaincode.

**Step 9.3:**	On PuTTY Session 1, switch to Peer 0 of Org0MSP by entering ``source scripts/setpeer 0 0``

**Step 9.4:** On PuTTY Session 1, issue the command to instantiate the chaincode on the channel::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode instantiate -o orderer.blockchain.com:7050 -n marbles -v 1.0 -c '{"Args":["init","1"]}' -P "OR ('Org0MSP.member','Org1MSP.member')" $FABRIC_TLS -C $CHANNEL_NAME
 2018-02-03 19:31:42.473 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
 2018-02-03 19:31:42.473 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
 2018-02-03 19:31:42.476 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
 2018-02-03 19:31:42.476 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
 2018-02-03 19:31:42.477 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode disabled
 2018-02-03 19:31:42.477 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AAB070A6708031A0C089E9CD8D30510...314D53500A04657363630A0476736363 
 2018-02-03 19:31:42.477 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: A018073CD196D38BD6DF0706FE713EAEC6B6B89219119954867859B129180B4D 
 2018-02-03 19:31:54.590 UTC [msp/identity] Sign -> DEBU 008 Sign: plaintext: 0AAB070A6708031A0C089E9CD8D30510...AE81CFB1F3F50254311A8479D7A3F9A9 
 2018-02-03 19:31:54.590 UTC [msp/identity] Sign -> DEBU 009 Sign: digest: 29AB9F1452BDA29D75E3F237175CF0E9108FD0A57CD61947DA13076B117109C4 
 2018-02-03 19:31:54.594 UTC [main] main -> INFO 00a Exiting.....

**Note:**  In your prior commands, when specifying the channel name, you used lowercase ‘c’ as the argument, e.g., *-c $CHANNEL_NAME*.  
In the *peer chaincode instantiate* command however, you use an uppercase ‘C’ as the argument to specify the channel name, e.g., 
*-C mychannel*, because -c is used to specify the arguments given to the chaincode.  Why *c* for arguments you may ask?  Well, the ‘*c*’ 
is short for ‘*ctor*’, which itself is an abbreviation for constructor, which is a fancy word object-oriented programmers use to refer 
to the initial arguments given when creating an object.  Some people do not like being treated as objects, but evidently chaincode 
does not object to being objectified.

**Step 9.5:**	You may have noticed a longer than usual pause while that last command was being run.  The reason for this is that as part of 
the instantiate, a Docker image for the chaincode is created and then a Docker container is started from the image.  To prove this to 
yourself, on PuTTY Session 2, enter *docker images dev-** and *docker ps | grep -v hyperledger*::

 bcuser@ubuntu16043:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              67c5bde2de7a        2 minutes ago bcuser@ubuntu16043:~$ 
 bcuser@ubuntu16043:~$ docker ps | grep -v hyperledger 
 CONTAINER ID        IMAGE                                                                                                      COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
 562b9a662bd8        dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.a..."   3 minutes ago       Up 3 minutes                                                                                    dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16043:~$ 

The naming convention used by Hyperledger Fabric v1.1.0 for the Docker images it creates for chaincode is *HyperledgerFabricNetworkName-PeerName-ChaincodeName-ChaincodeVersion-SHA256Hash*. In our case of *dev-peer0.unitedmarbles.com-marbles-1.0-*, the 
default name of a Hyperledger Fabric network is *dev*, and you did not change it.  *peer0.unitedmarbles.com* is the peer name of 
peer0 of Org0MSP, and you specified this via the CORE_PEER_ID environment variable in the Docker Compose YAML file. *marbles* is the 
name you gave this chaincode in the *-n* argument of the *peer chaincode install* command, and *1.0* is the version of the chaincode 
you used in the *-v* argument of the *peer chaincode install* command.

Note that a chaincode Docker container was only created for the peer on which you entered the *peer chaincode instantiate* command.  
Docker containers will not be created on the other peers until you run a *peer chaincode invoke* or *peer chaincode query* command on 
that peer.
 

Section 10 - Invoke chaincode functions
=======================================

You are now ready to invoke chaincode functions that will create, read, update and delete data in the ledger.

In this section, you will enter *scripts/setpeer* and *peer chaincode commands* in PuTTY session 1, while you will enter *docker ps* and 
*docker images* commands in PuTTY session 2.
 
**Step 10.1:** Switch to peer0 of Org0 by entering ``scripts/setpeer 0 0`` in PuTTY session 1.

**Step 10.2:**	You will use the marbles chaincode to create a new Marbles owner named John.  If you would like to use a different name 
than John, that is fine but there will be other places later where you will need to use your “custom” name instead of John.  I will let 
you know when that is necessary.  Enter this command in PuTTY session 1::

 peer chaincode invoke -n marbles -c '{"Args":["init_owner", "o0000000000001","John","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME

You will see a lot of output that should end with the result of the invoke-  it is a little daunting but if you look carefully you should notice that much of what you 
input is shown in the results::

 2018-02-03 19:38:56.159 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 078 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \315\345J\275\313\251Fg\254\016\355\213\227\276\350,\341\022o\260F\032\001\264\r\246\3516\207\202\234\352\022\300\001\n\250\001\022\027\n\004lscc\022\017\n\r\n\007marbles\022\002\010\003\022\214\001\n\007marbles\022\200\001\n\020\n\016o0000000000001\032l\n\016o0000000000001\032Z{\"docType\":\"marble_owner\",\"id\":\"o0000000000001\",\"username\":\"john\",\"company\":\"Marbles Inc\"}\032\003\010\310\001\"\016\022\007marbles\032\0031.0" endorsement:<endorser:"\n\007Org0MSP\022\232\006-----BEGIN CERTIFICATE-----\nMIICHTCCAcOgAwIBAgIRAJWBfH/22tymH/PCqtwrwqAwCgYIKoZIzj0EAwIwdTEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGjAYBgNVBAoTEXVuaXRlZG1hcmJsZXMuY29tMR0wGwYDVQQDExRj\nYS51bml0ZWRtYXJibGVzLmNvbTAeFw0xODAyMDMxODUyMTJaFw0yODAyMDExODUy\nMTJaMFwxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH\nEw1TYW4gRnJhbmNpc2NvMSAwHgYDVQQDExdwZWVyMC51bml0ZWRtYXJibGVzLmNv\nbTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABA7vSTvBpqBaRoT8vYbamD/qJCcr\n2xbFhkZkI8StVmZRKrbKpzhOV/GkupXXmZbMJ88IhnAIqbCreSedmPdb8cujTTBL\nMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAILeQjr2Z\nbYgPwpVPFx/aRKKvey9W1nj2pOWlZU8tkWuqMAoGCCqGSM49BAMCA0gAMEUCIQDK\nQS0lbimtn4A3BAldqQ3B4w3pTNWrbCX6v33IeEx3VgIgRZkaxQk4UKpwe8/nGiio\nODD67Z2nLzEDiKtQY/ibEIw=\n-----END CERTIFICATE-----\n" signature:"0D\002 P\250L\t\335[_k\272GT\nt\364(\350\222\375g\310\253\362f\264\257`\3108\303\314T\203\002 {Q\202\274\177\302I?\035\264w\275\315\255\363Mz=*R\030y\207fN\237D\317\355\257O\317" > 
 2018-02-03 19:38:56.159 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 079 Chaincode invoke successful. result: status:200 
 2018-02-03 19:38:56.160 UTC [main] main -> INFO 07a Exiting.....

 
**Step 10.3:**	Let’s deconstruct the arguments to the chaincode::

 {“Args”:[“init_owner”, “o0000000000001”, “John”, “Marbles Inc”]}
 
This is in JSON format.  JSON stands for JavaScript Object Notation, and is a very popular format for transmitting data in many 
languages, not just with JavaScript.  What is shown above is a single name/value pair.  The name is *Args* and the value is an array of 
four arguments.  (The square brackets “[“ and “]” specify an array in JSON).

**Note:** In the formal JSON definition the term ‘*name/value*’ is used, but many programmers will also use the term ‘*key/value*’ 
instead.  You can consider these two terms as synonymous.  (Many people use the phrase “the same” instead of the word “synonymous”).

The *Args* name specifies the arguments passed to the chaincode invocation.  There is an interface layer, also called a “shim”, that 
gains control before passing it along to user-written chaincode functions-  it expects this *Args* name/value pair.

The shim also expects the first array value to be the name of the user-written chaincode function that it will pass control to, and 
then all remaining array values are the arguments to pass, in order, to that user-written chaincode function.

So, in the command you just entered, the *init_owner* function is called, and it is passed three arguments, *o0000000000001*, *John*, 
and *Marbles Inc*. 

It is logic within the *init_owner* function that cause updates to the channel’s ledger- subject to the transaction flow in Hyperledger 
Fabric v1.1.0-  that is, chaincode execution causes proposed updates to the ledger, which are only committed at the end of the 
transaction flow if everything is validated properly.  But it all starts with function calls inside the chaincode functions that ask 
for ledger state to be created or updated.

**Step 10.4:**	Go to PuTTY session 2, and enter these two Docker commands and you will observe that you only have a Docker image and a 
Docker container for peer0 of Org0::

 bcuser@ubuntu16043:~/zmarbles$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              67c5bde2de7a        9 minutes ago       218MB
  bcuser@ubuntu16043:~$ docker ps --no-trunc | grep dev-
 562b9a662bd867045590f4adc409af7c7bc4ab807eae422986bd384672bb0c11   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                10 minutes ago      Up 10 minutes                                                                                   dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16043:~$ 

The takeaway is that the chaincode execution has only run on peer0 of Org0 so far, and this is also the peer on which you instantiated 
the chaincode, so the Docker image for the chaincode, and the corresponding Docker container based on the image, have been created for 
only this peer.  You will see soon that other peers will have their own chaincode Docker image and Docker container built the first 
time they are needed.

**Step 10.5:**	You created a marble owner in the previous step, now create a marble belonging to this owner.   Perform this from peer0 of 
Org1, so from PuTTY session 1, enter ``source scripts/setpeer 1 0`` and then enter::

 peer chaincode invoke -n marbles -c '{"Args":["init_marble","m0000000000001","blue","35","o0000000000001","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME 

The end of the output should show a good result through all the confusion- again::

 2018-02-03 19:44:17.632 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 078 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \355J(\252F\320@6\362\354\323\035\301\322\302?n\346e\034\305b5g\307f\220\226n\340\300\335\022\270\002\n\240\002\022\027\n\004lscc\022\017\n\r\n\007marbles\022\002\010\003\022\204\002\n\007marbles\022\370\001\n\020\n\016m0000000000001\n\024\n\016o0000000000001\022\002\010\004\032\315\001\n\016m0000000000001\032\272\001{\n\t\t\"docType\":\"marble\", \n\t\t\"id\": \"m0000000000001\", \n\t\t\"color\": \"blue\", \n\t\t\"size\": 35, \n\t\t\"owner\": {\n\t\t\t\"id\": \"o0000000000001\", \n\t\t\t\"username\": \"john\", \n\t\t\t\"company\": \"Marbles Inc\"\n\t\t}\n\t}\032\003\010\310\001\"\016\022\007marbles\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\216\006-----BEGIN CERTIFICATE-----\nMIICEzCCAbqgAwIBAgIRAKwfx16zctV76tc9ktmIxoAwCgYIKoZIzj0EAwIwbzEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xFzAVBgNVBAoTDm1hcmJsZXNpbmMuY29tMRowGAYDVQQDExFjYS5t\nYXJibGVzaW5jLmNvbTAeFw0xODAyMDMxODUyMTJaFw0yODAyMDExODUyMTJaMFkx\nCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4g\nRnJhbmNpc2NvMR0wGwYDVQQDExRwZWVyMC5tYXJibGVzaW5jLmNvbTBZMBMGByqG\nSM49AgEGCCqGSM49AwEHA0IABG/SYfI7A2puIX2QP7uv1QWh4PsDpz/m9QNPXTTi\nw5wivftnoxdUCSGTkLA1rZJJNryEryv+RWVPjkHc3UoGenajTTBLMA4GA1UdDwEB\n/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIJ7EfvoO1EQigVs+v0/4\ngrUYlkkYCYkF6jFAGxxWOd4NMAoGCCqGSM49BAMCA0cAMEQCIHLSsJP6aUR7ol4b\nxnXLbAFecwV3Nl+b4SZCV9jyFsMhAiBfu/FLyYGKiBzjMNacb4PFGKNbtLuQeKJJ\nK3z2o0rZOg==\n-----END CERTIFICATE-----\n" signature:"0E\002!\000\206\3673\353\203\243X\032 {\037\t\247OE\\#+\225\235j\235\340\216\273G\340\214\355\206\014\245\002 !\333\265\223b\005|\327\260\2028\303\200\313A\037\234\021\3533\337\221(\026&\214+\336g@\226\253" > 
 2018-02-03 19:44:17.632 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 079 Chaincode invoke successful. result: status:200 
 2018-02-03 19:44:17.633 UTC [main] main -> INFO 07a Exiting.....


This time you called the *init_marble* function.  Now you have created one owner, and one marble.

The owner is *John* (or your custom name) and his id is *o0000000000001*, and his marble has an id of *m0000000000001*.  I cleverly 
decided that the letter ‘*o*’ stands for owner and the letter ‘*m*’ stands for marbles.  I put 12 leading zeros in front of the number 
1 in case you wanted to stay late and create trillions of marbles and owners.

**Step 10.6:**	In PuTTY session 2, repeat the Docker commands from *Step 10.4*.  Now you should see that you have two Docker images and two Docker containers::

 bcuser@ubuntu16043:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      latest              7628ef8c3e70        2 minutes ago       218MB
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              67c5bde2de7a        15 minutes ago      218MB
 bcuser@ubuntu16043:~$ docker ps --no-trunc | grep dev-
 bc45f38fd7e87f6c0033458a2631b801c9e8d24f1b18883bf469e51088047262   dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      "chaincode -peer.address=peer0.marblesinc.com:7052"                                                                                                                                                                                                                   3 minutes ago       Up 3 minutes                                                                                    dev-peer0.marblesinc.com-marbles-1.0
 562b9a662bd867045590f4adc409af7c7bc4ab807eae422986bd384672bb0c11   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                15 minutes ago      Up 15 minutes                                                                                   dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16043:~$ 


**Step 10.7:**	You will create a new owner now.  Try that on Peer 1 of Org0, so enter ``source scripts/setpeer 0 1`` in PuTTY session 1 
and then try the command::

 peer chaincode invoke -n marbles -c '{"Args":["init_owner","o0000000000002","Barry","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

What do you expect to happen when you enter this command?

Well, I don’t expect you to know for sure, but what I expect, if you have followed these instructions exactly, is that the *invoke* will 
fail.  It will fail because you have not yet installed the chaincode on Peer 1 of Org0.  Here is the relevant portion of the output 
describing the error::

 Error: Error endorsing invoke: rpc error: code = Unknown desc = Chaincode data for cc marbles/1.0 was not found, error cannot retrieve package for chaincode marbles/1.0, error open /var/hyperledger/production/chaincodes/marbles.1.0: no such file or directory - <nil>

You must first *install* chaincode on a peer not only before you can do an *instantiate* from that peer, but also before you can do 
an *invoke* or *query* from that peer.  If you want a peer to perform the endorsing function for a transaction, the chaincode for 
that transaction must be installed on that peer.  If that peer is a member of the channel on which the chaincode is instantiated, but 
has not had the chaincode installed on it, it will still perform the committer function and update its copy of the channel’s ledger 
when it receives valid transactions from the orderer, but it cannot endorse transaction proposals unless the chaincode has been 
installed on it.

**Step 10.8**:	Correct things by installing the chaincode on peer1 of Org0.  In PuTTY session 1, enter this command, which should 
look familiar to you::

 peer chaincode install -n marbles -v1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles

And since familiarity breeds contempt, I will not show the complete output but you should see a message near the bottom that reads
*Installed remotely response: <status:200 payload:”OK” >*

**Step 10.9:**	Now, in PuTTY session 1, repeat the *peer chaincode invoke* command from *Step 10.9*.  It should work this time::

 peer chaincode invoke -n marbles -c '{"Args":["init_owner","o0000000000002","Barry","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

The output format will be like what you have seen before, and you should be able to dig out some of the more human-readable pieces of 
it and assure yourself that this command succeeded.

**Step 10.10:**	Go back to PuTTY session 2 and enter the Docker commands that will show you that you now have your third pair in your 
set of chaincode-related Docker images and containers, the ones just built for peer1 of Org0::

 bcuser@ubuntu16043:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer1.unitedmarbles.com-marbles-1.0-dea1aa08dc7c6f282a31dd498670173c21d3e75ef0ef1d170b95e1212fbacb77   latest              49ebecd6c682        14 seconds ago      218MB
 dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      latest              7628ef8c3e70        6 minutes ago       218MB
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              67c5bde2de7a        18 minutes ago      218MB
 bcuser@ubuntu16043:~$ docker ps --no-trunc | grep dev-
 c776759144ed3d43062cc12507d6b7025c713448f5efeb42298c54b22569585f   dev-peer1.unitedmarbles.com-marbles-1.0-dea1aa08dc7c6f282a31dd498670173c21d3e75ef0ef1d170b95e1212fbacb77   "chaincode -peer.address=peer1.unitedmarbles.com:7052"                                                                                                                                                                                                                26 seconds ago      Up 25 seconds                                                                                   dev-peer1.unitedmarbles.com-marbles-1.0
 bc45f38fd7e87f6c0033458a2631b801c9e8d24f1b18883bf469e51088047262   dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      "chaincode -peer.address=peer0.marblesinc.com:7052"                                                                                                                                                                                                                   6 minutes ago       Up 6 minutes                                                                                    dev-peer0.marblesinc.com-marbles-1.0
 562b9a662bd867045590f4adc409af7c7bc4ab807eae422986bd384672bb0c11   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                19 minutes ago      Up 19 minutes                                                                                   dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16043:~$ 


**Step 10.11:**	Try some additional chaincode invocations. You have had enough experience switching between peers with  *source 
scripts/setpeer* and issuing the *peer chaincode invoke* command that I will not show the output, nor tell you from which peer you 
should enter your command.   I will just list several more commands you can run against the marbles chaincode. Feel free to switch 
amongst the four peers as you see fit before you enter each command.  Note however, that you have only installed the chaincode on 
three of the four peers, so if you choose that fourth peer, you will need to install the chaincode there first.   
I won’t tell you which peer does not currently have the chaincode installed, but if you need a hint, it is the one that does not 
have a Docker image built yet for its chaincode.  (Note that checking for the absence of a Docker image for a peer is not, by itself,
proof that you have not installed the chaincode on that peer- the Docker image is not built until you first invoke a function against 
the chaincode on that peer).

If you are ambitious and want to install the chaincode on that fourth peer, 
try the useful Docker commands I have shown you from PuTTY session 2 to see that the chaincode's Docker image and Docker container
are created when you invoke a transaction on that fourth peer.

Try some or all of these commands from PuTTY session 1:

Create a marble for Barry, i.e., owner o0000000000002::

 peer chaincode invoke -n marbles -c '{"Args":["init_marble","m0000000000002","green","50","o0000000000002","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

Obtain all marble information-  marbles and owners::

 peer chaincode invoke -n marbles -c '{"Args":["read_everything"]}' $FABRIC_TLS -C $CHANNEL_NAME

Change marble ownership-  ‘Barry’ is giving his marble to ‘John’::

 peer chaincode invoke -n marbles -c '{"Args":["set_owner","m0000000000002","o0000000000001","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

Get the history of marble ‘m0000000000002’::

 peer chaincode invoke -n marbles -c '{"Args":["getHistory","m0000000000002"]}' $FABRIC_TLS -C $CHANNEL_NAME

Delete marble ‘m0000000000002’::

 peer chaincode invoke -n marbles -c '{"Args":["delete_marble","m0000000000002","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME

Try again to get the history of marble ‘m0000000000002’ after you just deleted it::

 peer chaincode invoke -n marbles -c '{"Args":["getHistory","m0000000000002"]}' $FABRIC_TLS -C $CHANNEL_NAME

Obtain all marble information again.  See if it matches your expectations based on the commands you have entered::

 peer chaincode invoke -n marbles -c '{"Args":["read_everything"]}' $FABRIC_TLS -C $CHANNEL_NAME
 
**Step 10.12:** Exit the *cli* Docker container from PuTTY session 1.  Your command prompt should change to reflect that you are now 
back at your Linux on IBM Z host prompt and no longer in the Docker container::

 root@acd1f96d8807:/opt/gopath/src/github.com/hyperledger/fabric/peer# exit
 exit
 bcuser@ubuntu16043:~/zmarbles$ 


**Step 10.13:**	Congratulations!! Congratulations on your fortitude and perseverance.  Leave your Hyperledger Fabric network and all 
the chaincode Docker containers up and running-  you will use what you created here in the next lab where you will install a 
front-end Web application that will interact with the marbles chaincode that you have installed in this lab.


