Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.5 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.5 LTS* and the Linux kernel is at level *4.4.0-139*.  In addtion, several software prerequisites have already been installed on this instance for you, including:

*	Docker and Docker Compose
*	Node.js and npm
*	Golang compiler
*	other necessary packages

During the lab, you will download three Hyperledger Fabric source code repositories, build the necessary artifacts from the source code, and run two comprehensive end-to-end tests to verify that your Hyperledger Fabric installation is in working order.

You will first download the *fabric* repository, which contains the source code for the core functionality of Hyperledger Fabric.  Using this source code, you will build the Docker images that comprise the core Hyperledger Fabric services. Then you will run an end-to-end test that utilizes the Hyperledger Fabric Command Line Interface (CLI) to verify that everything is working correctly.

Then you will download two more Hyperledger Fabric source code repositories, *fabric-ca* and *fabric-sdk-node*, and build from this source the artifacts necessary to run a second, more comprehensive end-to-end test of the Fabric Node.js SDK.
 
Section 2: Download, build and test the Hyperledger Fabric CLI
==============================================================

In this section, you will:

*	Download the source code repository containing the core Hyperledger Fabric functionality
*	Use the source code to build Docker images that contain the core Hyperledger Fabric functionality
*	Test for success by running the comprehensive end-to-end CLI test.

**Step 2.1:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it. If you are following these steps exactly, you already are.  If you strayed away from your home directory, I'm assuming you're smart enough to get back there or at least smart enough to ask for help::

 bcuser@ubuntu16045:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16045:~$
 
**Step 2.2:** Navigate to the directory you just created::

 bcuser@ubuntu16045:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$
 
**Step 2.3:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place where it lives::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.3.0 https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 5663, done
 remote: Finding sources: 100% (44/44)
 remote: Total 83861 (delta 0), reused 83830 (delta 0)
 Receiving objects: 100% (83861/83861), 103.24 MiB | 27.24 MiB/s, done.
 Resolving deltas: 100% (36805/36805), done.
 Checking connectivity... done.
 Note: checking out 'ab0a67ad7102f1281736e2c309e9a0ef91b08fd9'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>


*Note:* The numbers in the various output messages may differ from what you see listed here, and this may be the case for any other times you do a *git clone* in the remainder of these labs.

**Step 2.4:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$

**Step 2.5:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 2.6:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real	4m10.454s
 user	0m15.755s
 sys	0m1.330s
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ 

**Step 2.7:** Run *docker images* again and you will see several Docker images that were just created. You will notice that many of the Docker images at the top of the output were created in the last few minutes.  These were created by the *make docker* command.  The Docker images that are several months old were downloaded from Hyperledger Fabric's public DockerHub repository.  Your output should look similar to that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-tools       latest                         bdceed408af3        17 seconds ago      1.48GB
 hyperledger/fabric-tools       s390x-1.3.0-snapshot-ab0a67a   bdceed408af3        17 seconds ago      1.48GB
 hyperledger/fabric-tools       s390x-latest                   bdceed408af3        17 seconds ago      1.48GB
 <none>                         <none>                         a1914bb59c76        24 seconds ago      1.62GB
 hyperledger/fabric-testenv     latest                         3a5d26529b4b        2 minutes ago       1.54GB
 hyperledger/fabric-testenv     s390x-1.3.0-snapshot-ab0a67a   3a5d26529b4b        2 minutes ago       1.54GB
 hyperledger/fabric-testenv     s390x-latest                   3a5d26529b4b        2 minutes ago       1.54GB
 hyperledger/fabric-buildenv    latest                         97da0e9277ef        2 minutes ago       1.45GB
 hyperledger/fabric-buildenv    s390x-1.3.0-snapshot-ab0a67a   97da0e9277ef        2 minutes ago       1.45GB
 hyperledger/fabric-buildenv    s390x-latest                   97da0e9277ef        2 minutes ago       1.45GB
 hyperledger/fabric-ccenv       latest                         662ea5e33ace        2 minutes ago       1.39GB
 hyperledger/fabric-ccenv       s390x-1.3.0-snapshot-ab0a67a   662ea5e33ace        2 minutes ago       1.39GB
 hyperledger/fabric-ccenv       s390x-latest                   662ea5e33ace        2 minutes ago       1.39GB
 hyperledger/fabric-orderer     latest                         0958803f45ff        3 minutes ago       142MB
 hyperledger/fabric-orderer     s390x-1.3.0-snapshot-ab0a67a   0958803f45ff        3 minutes ago       142MB
 hyperledger/fabric-orderer     s390x-latest                   0958803f45ff        3 minutes ago       142MB
 hyperledger/fabric-peer        latest                         06a39ec85563        3 minutes ago       149MB
 hyperledger/fabric-peer        s390x-1.3.0-snapshot-ab0a67a   06a39ec85563        3 minutes ago       149MB
 hyperledger/fabric-peer        s390x-latest                   06a39ec85563        3 minutes ago       149MB
 hyperledger/fabric-baseimage   s390x-0.4.13                   f93a607516a7        8 weeks ago         1.35GB
 hyperledger/fabric-baseos      s390x-0.4.13                   8ed79b01636d        8 weeks ago         120MB

**Step 2.8:** Navigate to the directory where the “end-to-end” test lives::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ cd examples/e2e_cli
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$

**Step 2.9:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a process, and it is based on a Docker image.  Run this command, which shows all Docker containers. Right now there will be no output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps --all
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 2.10:** Run the end-to-end test with this command::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 2.11:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps --all
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                     PORTS                                                                       NAMES
 983cc0fce2e8        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   18 seconds ago       Up 17 seconds                                                                                          dev-peer1.org2.example.com-mycc-1.0
 19997254a1d4        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   37 seconds ago       Up 35 seconds                                                                                          dev-peer0.org1.example.com-mycc-1.0
 080fc8a865ce        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   55 seconds ago       Up 54 seconds                                                                                          dev-peer0.org2.example.com-mycc-1.0
 e6f9a25aed07        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   About a minute ago   Exited (0) 3 seconds ago                                                                               cli
 7a3eb3056daf        hyperledger/fabric-orderer                                                                             "orderer"                About a minute ago   Up About a minute          0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 7a27c6d12dce        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka3
 a725fbd039a2        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka2
 690cfb96f8aa        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka1
 253ec4228247        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka0
 4c010968e221        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 90a84d2599bb        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 1fd9d9272298        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 abeb31d61054        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 982b65d9eba8        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 721a24e18f49        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 a95de76b94f1        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32773->3888/tcp   zookeeper1
 b5f4136194be        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32774->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper2
 ad1f5388032f        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper0
 bf377461afee        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 c89871fcdef5        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3

The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one orderer container. The orderer service uses *Kafka* for consensus, and so is supported by four Kafka containers and three Zookeeper containers. There was a container created to run the CLI itself, and that container stopped running ten seconds after the test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 2.12:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh down

**Step 2.15:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps --all
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Recap:** In this section, you:

*	Downloaded the main Hyperledger Fabric source code repository
*	Ran *make* to build the project’s Docker images
*	Ran the Hyperledger Fabric command line interface (CLI) end-to-end test
*	Cleaned up afterwards
 
Section 3: Install the Hyperledger Fabric Certificate Authority
===============================================================

In the prior section, the end-to-end test that you ran supplied its own security-related material such as keys and certificates- everything it needed to perform its test.  Therefore it did not need the services of a Certificate Authority.

Almost all "real world" Hyperledger Fabric networks will not be this static-  new users, peers and organizations will probably join the network.  They will need Public Key Infrastructure (PKI) x.509 certificates in order to participate.  The Hyperledger Fabric Certificate Authority (CA) is provided by the Hyperledger Fabric project in order to issue these certificates.

The next major goal in this lab is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  This test makes calls to the Hyperledger Fabric Certificate Authority (CA). Therefore, before we can run that test, you will get started by downloading and building the Hyperledger Fabric CA.

**Step 3.1:** Use *cd* to navigate three directory levels up, to the *hyperledger* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ~/git/src/github.com/hyperledger
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 3.2:** Get the source code for the Fabric CA using *git*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.3.0 https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 2082, done
 remote: Finding sources: 100% (387/387)
 remote: Total 12505 (delta 22), reused 12501 (delta 22)
 Receiving objects: 100% (12505/12505), 28.35 MiB | 16.21 MiB/s, done.
 Resolving deltas: 100% (4265/4265), done.
 Checking connectivity... done.
 Note: checking out '8b93ae01aedda27162828ddeeeac9cbec27a5ad7'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 3.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.4:** Enter the following command, which will build the Hyperledger Fabric CA images.  Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images. You may see a couple of warnings near the top of the output about cache being disabled. You may ignore these warnings.::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca $ time FABRIC_CA_DYNAMIC_LINK=true make docker
   .
   .  (output not shown here)
   .
 real	1m29.510s
 user	0m0.313s
 sys	0m0.160s
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.5:** Enter the *docker images* command and you will see at the top of the output the Docker image that was just created for the Fabric Certificate Authority::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca          latest                         4e2616487cde        35 seconds ago      317MB
 hyperledger/fabric-ca          s390x-1.3.0                    4e2616487cde        35 seconds ago      317MB
 hyperledger/fabric-tools       latest                         bdceed408af3        10 minutes ago      1.48GB
 hyperledger/fabric-tools       s390x-1.3.0-snapshot-ab0a67a   bdceed408af3        10 minutes ago      1.48GB
 hyperledger/fabric-tools       s390x-latest                   bdceed408af3        10 minutes ago      1.48GB
 hyperledger/fabric-testenv     latest                         3a5d26529b4b        11 minutes ago      1.54GB
 hyperledger/fabric-testenv     s390x-1.3.0-snapshot-ab0a67a   3a5d26529b4b        11 minutes ago      1.54GB
 hyperledger/fabric-testenv     s390x-latest                   3a5d26529b4b        11 minutes ago      1.54GB
 hyperledger/fabric-buildenv    latest                         97da0e9277ef        12 minutes ago      1.45GB
 hyperledger/fabric-buildenv    s390x-1.3.0-snapshot-ab0a67a   97da0e9277ef        12 minutes ago      1.45GB
 hyperledger/fabric-buildenv    s390x-latest                   97da0e9277ef        12 minutes ago      1.45GB
 hyperledger/fabric-ccenv       latest                         662ea5e33ace        12 minutes ago      1.39GB
 hyperledger/fabric-ccenv       s390x-1.3.0-snapshot-ab0a67a   662ea5e33ace        12 minutes ago      1.39GB
 hyperledger/fabric-ccenv       s390x-latest                   662ea5e33ace        12 minutes ago      1.39GB
 hyperledger/fabric-orderer     latest                         0958803f45ff        12 minutes ago      142MB
 hyperledger/fabric-orderer     s390x-1.3.0-snapshot-ab0a67a   0958803f45ff        12 minutes ago      142MB
 hyperledger/fabric-orderer     s390x-latest                   0958803f45ff        12 minutes ago      142MB
 hyperledger/fabric-peer        latest                         06a39ec85563        13 minutes ago      149MB
 hyperledger/fabric-peer        s390x-1.3.0-snapshot-ab0a67a   06a39ec85563        13 minutes ago      149MB
 hyperledger/fabric-peer        s390x-latest                   06a39ec85563        13 minutes ago      149MB
 hyperledger/fabric-zookeeper   latest                         5db059b03239        7 weeks ago         1.42GB
 hyperledger/fabric-kafka       latest                         3bbd80f55946        7 weeks ago         1.43GB
 hyperledger/fabric-couchdb     latest                         7afa6ce179e6        7 weeks ago         1.55GB
 hyperledger/fabric-baseimage   s390x-0.4.13                   f93a607516a7        8 weeks ago         1.35GB
 hyperledger/fabric-baseos      s390x-0.4.13                   8ed79b01636d        8 weeks ago         120MB

You may have noticed that for many of the images, the *Image ID* appears more than once, e.g., once with a tag of *latest*,  once with a tag such as *s390x-1.3.0-snapshot-ab0a67a*, and once with a tag of *s390x-latest*. An image can actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive tag, *ss390x-1.3.0-snapshot-ab0a67a* in the case of the *fabric* repo, and *s390x-1.3.0* in the case of the *fabric-ca* repo, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.3.0*, and in all three cases you are asking for the same image, the image with ID *4e2616487cde*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 4: Install Hyperledger Fabric Node.js SDK
=================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 4.1:** Back up one directory level to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 4.2:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ git clone -b v1.3.0 https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 1054, done
 remote: Finding sources: 100% (34/34)
 remote: Total 12920 (delta 3), reused 12904 (delta 3)
 Receiving objects: 100% (12920/12920), 9.19 MiB | 9.68 MiB/s, done.
 Resolving deltas: 100% (6222/6222), done.
 Checking connectivity... done.
 Note: checking out '95b02d9b2ce508d8a3684792b6b040ae04a4067c'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 4.3:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.4:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.3.0 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)
 
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.5:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN gulp-debug@4.0.0 requires a peer of gulp@>=4 but none is installed. You must install peer dependencies yourself.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.4 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.4: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 1443 packages in 98.288s

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 4.6:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
 
Section 5: Run the Hyperledger Fabric Node.js SDK end-to-end test
=================================================================
In this section, you will run two tests provided by the Hyperledger Fabric Node.js SDK, verify their successful operation, and clean up afterwards.

The first test is a quick test that takes about a minute and does not bring up any chaincode containers.  The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take several minutes.

**Step 5.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
 1..890
 # tests 890
 # pass  890

 # ok

 [11:06:21] Finished 'run-headless' after 48 s
 [11:06:21] Starting 'run-test-headless'...
   .
   . (a bunch of output not shown here)
   .
   188 passing (491ms)

 [11:06:23] Finished 'run-test-headless' after 2.14 s
 -----------------------------------|----------|----------|----------|----------|-------------------|
 File                               |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 -----------------------------------|----------|----------|----------|----------|-------------------|
 All files                          |     82.9 |    77.59 |    89.83 |    82.96 |                   |
  fabric-ca-client/lib              |      100 |    99.02 |      100 |      100 |                   |
   AffiliationService.js            |      100 |      100 |      100 |      100 |                   |
   IdentityService.js               |      100 |      100 |      100 |      100 |                   |
   helper.js                        |      100 |       95 |      100 |      100 |                66 |
  fabric-client/lib                 |     84.6 |    83.17 |    94.27 |    84.62 |                   |
   BaseClient.js                    |      100 |      100 |      100 |      100 |                   |
   BlockDecoder.js                  |      100 |      100 |      100 |      100 |                   |
   CertificateAuthority.js          |      100 |      100 |      100 |      100 |                   |
   Channel.js                       |    53.58 |    49.51 |    74.58 |    53.57 |... 3895,3897,3899 |
   ChannelEventHub.js               |      100 |      100 |      100 |      100 |                   |
   Client.js                        |      100 |      100 |      100 |      100 |                   |
   Config.js                        |      100 |      100 |      100 |      100 |                   |
   Constants.js                     |      100 |      100 |      100 |      100 |                   |
   Orderer.js                       |      100 |      100 |      100 |      100 |                   |
   Organization.js                  |      100 |      100 |      100 |      100 |                   |
   Package.js                       |      100 |      100 |      100 |      100 |                   |
   Packager.js                      |      100 |      100 |      100 |      100 |                   |
   Peer.js                          |      100 |    94.44 |      100 |      100 |             62,66 |
   Policy.js                        |      100 |    93.88 |      100 |      100 |        81,171,191 |
   Remote.js                        |      100 |      100 |      100 |      100 |                   |
   SideDB.js                        |      100 |      100 |      100 |      100 |                   |
   TransactionID.js                 |      100 |      100 |      100 |      100 |                   |
   User.js                          |      100 |    98.33 |      100 |      100 |                61 |
   api.js                           |      100 |      100 |      100 |      100 |                   |
   client-utils.js                  |      100 |      100 |      100 |      100 |                   |
   hash.js                          |      100 |      100 |      100 |      100 |                   |
   utils.js                         |    91.95 |    90.32 |    97.14 |    91.95 |... 54,556,558,561 |
  fabric-client/lib/impl            |    66.85 |    60.18 |    69.84 |     66.7 |                   |
   BasicCommitHandler.js            |    73.33 |       70 |      100 |    73.33 |... 19,120,123,124 |
   CouchDBKeyValueStore.js          |    76.71 |       60 |    93.33 |    77.46 |... 46,147,160,161 |
   CryptoKeyStore.js                |      100 |     87.5 |      100 |      100 |             42,76 |
   CryptoSuite_ECDSA_AES.js         |     84.4 |    71.84 |    78.95 |       85 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js   |    73.41 |    59.72 |      100 |    73.41 |... 87,289,297,299 |
   FileKeyValueStore.js             |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js             |    97.89 |    84.78 |      100 |    97.85 |... 74,388,421,422 |
   bccsp_pkcs11.js                  |    25.58 |    30.97 |     8.33 |    24.02 |... 1051,1055,1056 |
  fabric-client/lib/impl/aes        |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                    |    11.11 |        0 |        0 |    11.11 |... 39,43,47,51,55 |
  fabric-client/lib/impl/ecdsa      |    49.29 |    31.25 |       45 |    51.11 |                   |
   key.js                           |    98.41 |    96.15 |      100 |    98.41 |               182 |
   pkcs11_key.js                    |     9.09 |        0 |        0 |     9.72 |... 55,159,160,162 |
  fabric-client/lib/msp             |    78.41 |    62.92 |    73.33 |    78.16 |                   |
   identity.js                      |    85.71 |    66.67 |    76.92 |    85.71 |... 06,107,201,228 |
   msp-manager.js                   |    86.54 |    77.27 |      100 |       86 |... 5,76,77,78,146 |
   msp.js                           |    66.18 |    46.43 |       50 |    66.18 |... 38,139,181,182 |
  fabric-client/lib/packager        |      100 |      100 |      100 |      100 |                   |
   BasePackager.js                  |      100 |      100 |      100 |      100 |                   |
   Car.js                           |      100 |      100 |      100 |      100 |                   |
   Golang.js                        |      100 |      100 |      100 |      100 |                   |
   Java.js                          |      100 |      100 |      100 |      100 |                   |
   Node.js                          |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib/utils           |      100 |      100 |      100 |      100 |                   |
   ChannelHelper.js                 |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib                |      100 |      100 |      100 |      100 |                   |
   contract.js                      |      100 |      100 |      100 |      100 |                   |
   gateway.js                       |      100 |      100 |      100 |      100 |                   |
   logger.js                        |      100 |      100 |      100 |      100 |                   |
   network.js                       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/api            |      100 |      100 |      100 |      100 |                   |
   queryhandler.js                  |      100 |      100 |      100 |      100 |                   |
   wallet.js                        |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/event     |      100 |      100 |      100 |      100 |                   |
   abstracteventstrategy.js         |      100 |      100 |      100 |      100 |                   |
   allfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   anyfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   defaulteventhandlerstrategies.js |      100 |      100 |      100 |      100 |                   |
   eventhubfactory.js               |      100 |      100 |      100 |      100 |                   |
   transactioneventhandler.js       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/query     |      100 |      100 |      100 |      100 |                   |
   defaultqueryhandler.js           |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/wallet    |      100 |      100 |      100 |      100 |                   |
   basewallet.js                    |      100 |      100 |      100 |      100 |                   |
   couchdbwallet.js                 |      100 |      100 |      100 |      100 |                   |
   filesystemwallet.js              |      100 |      100 |      100 |      100 |                   |
   inmemorywallet.js                |      100 |      100 |      100 |      100 |                   |
   x509walletmixin.js               |      100 |      100 |      100 |      100 |                   |
 -----------------------------------|----------|----------|----------|----------|-------------------|

 =============================== Coverage summary ===============================
 Statements   : 82.9% ( 5864/7074 )
 Branches     : 77.59% ( 2430/3132 )
 Functions    : 89.83% ( 830/924 )
 Lines        : 82.96% ( 5829/7026 )
 ================================================================================
 [11:06:34] Finished 'test-headless' after 1.2 min

You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this all of the tests pass, similar to what you see in the sample output. **Note:** The number of tests run for you may differ from the number shown here. 

**Step 5.2:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..2179
 # tests 2179
 # pass  2179

 # ok

 [11:31:25] Finished 'run-full' after 18 min
 [11:31:25] Starting 'run-test'...
   .
   . (a bunch of output not shown here)
   .
    188 passing (862ms)

 [11:31:29] Finished 'run-test' after 3.74 s
 -----------------------------------|----------|----------|----------|----------|-------------------|
 File                               |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 -----------------------------------|----------|----------|----------|----------|-------------------|
 All files                          |    91.69 |    84.55 |    93.07 |    91.79 |                   |
  fabric-ca-client/lib              |      100 |    99.02 |      100 |      100 |                   |
   AffiliationService.js            |      100 |      100 |      100 |      100 |                   |
   IdentityService.js               |      100 |      100 |      100 |      100 |                   |
   helper.js                        |      100 |       95 |      100 |      100 |                66 |
  fabric-client/lib                 |    97.25 |    93.15 |    99.63 |    97.27 |                   |
   BaseClient.js                    |      100 |      100 |      100 |      100 |                   |
   BlockDecoder.js                  |      100 |      100 |      100 |      100 |                   |
   CertificateAuthority.js          |      100 |      100 |      100 |      100 |                   |
   Channel.js                       |    92.03 |    80.84 |    98.31 |    92.06 |... 3852,3853,3895 |
   ChannelEventHub.js               |      100 |      100 |      100 |      100 |                   |
   Client.js                        |      100 |      100 |      100 |      100 |                   |
   Config.js                        |      100 |      100 |      100 |      100 |                   |
   Constants.js                     |      100 |      100 |      100 |      100 |                   |
   Orderer.js                       |      100 |      100 |      100 |      100 |                   |
   Organization.js                  |      100 |      100 |      100 |      100 |                   |
   Package.js                       |      100 |      100 |      100 |      100 |                   |
   Packager.js                      |      100 |      100 |      100 |      100 |                   |
   Peer.js                          |      100 |    94.44 |      100 |      100 |             62,66 |
   Policy.js                        |      100 |    93.88 |      100 |      100 |        81,171,191 |
   Remote.js                        |      100 |      100 |      100 |      100 |                   |
   SideDB.js                        |      100 |      100 |      100 |      100 |                   |
   TransactionID.js                 |      100 |      100 |      100 |      100 |                   |
   User.js                          |      100 |    98.33 |      100 |      100 |                61 |
   api.js                           |      100 |      100 |      100 |      100 |                   |
   client-utils.js                  |      100 |      100 |      100 |      100 |                   |
   hash.js                          |      100 |      100 |      100 |      100 |                   |
   utils.js                         |    96.61 |    91.94 |      100 |    96.61 |... 17,218,219,558 |
  fabric-client/lib/impl            |    70.03 |    63.35 |    69.84 |    69.94 |                   |
   BasicCommitHandler.js            |    88.33 |       85 |      100 |    88.33 |... 19,120,123,124 |
   CouchDBKeyValueStore.js          |    80.82 |    63.33 |    93.33 |    81.69 |... 46,147,160,161 |
   CryptoKeyStore.js                |      100 |     87.5 |      100 |      100 |             42,76 |
   CryptoSuite_ECDSA_AES.js         |     84.4 |    71.84 |    78.95 |       85 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js   |    86.71 |    76.39 |      100 |    86.71 |... 87,289,297,299 |
   FileKeyValueStore.js             |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js             |    97.89 |     87.5 |      100 |    97.85 |... 74,388,421,422 |
   bccsp_pkcs11.js                  |    25.58 |    30.97 |     8.33 |    24.02 |... 1051,1055,1056 |
  fabric-client/lib/impl/aes        |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                    |    11.11 |        0 |        0 |    11.11 |... 39,43,47,51,55 |
  fabric-client/lib/impl/ecdsa      |    49.29 |    31.25 |       45 |    51.11 |                   |
   key.js                           |    98.41 |    96.15 |      100 |    98.41 |               182 |
   pkcs11_key.js                    |     9.09 |        0 |        0 |     9.72 |... 55,159,160,162 |
  fabric-client/lib/msp             |    79.55 |    65.17 |    76.67 |    79.31 |                   |
   identity.js                      |    89.29 |    69.23 |    84.62 |    89.29 |... 96,106,107,201 |
   msp-manager.js                   |    86.54 |    81.82 |      100 |       86 |... 5,76,77,78,146 |
   msp.js                           |    66.18 |    46.43 |       50 |    66.18 |... 38,139,181,182 |
  fabric-client/lib/packager        |      100 |      100 |      100 |      100 |                   |
   BasePackager.js                  |      100 |      100 |      100 |      100 |                   |
   Car.js                           |      100 |      100 |      100 |      100 |                   |
   Golang.js                        |      100 |      100 |      100 |      100 |                   |
   Java.js                          |      100 |      100 |      100 |      100 |                   |
   Node.js                          |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib/utils           |      100 |      100 |      100 |      100 |                   |
   ChannelHelper.js                 |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib                |      100 |      100 |      100 |      100 |                   |
   contract.js                      |      100 |      100 |      100 |      100 |                   |
   gateway.js                       |      100 |      100 |      100 |      100 |                   |
   logger.js                        |      100 |      100 |      100 |      100 |                   |
   network.js                       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/api            |      100 |      100 |      100 |      100 |                   |
   queryhandler.js                  |      100 |      100 |      100 |      100 |                   |
   wallet.js                        |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/event     |      100 |      100 |      100 |      100 |                   |
   abstracteventstrategy.js         |      100 |      100 |      100 |      100 |                   |
   allfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   anyfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   defaulteventhandlerstrategies.js |      100 |      100 |      100 |      100 |                   |
   eventhubfactory.js               |      100 |      100 |      100 |      100 |                   |
   transactioneventhandler.js       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/query     |      100 |      100 |      100 |      100 |                   |
   defaultqueryhandler.js           |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/wallet    |      100 |      100 |      100 |      100 |                   |
   basewallet.js                    |      100 |      100 |      100 |      100 |                   |
   couchdbwallet.js                 |      100 |      100 |      100 |      100 |                   |
   filesystemwallet.js              |      100 |      100 |      100 |      100 |                   |
   inmemorywallet.js                |      100 |      100 |      100 |      100 |                   |
   x509walletmixin.js               |      100 |      100 |      100 |      100 |                   |
 -----------------------------------|----------|----------|----------|----------|-------------------|

 =============================== Coverage summary ===============================
 Statements   : 91.69% ( 6486/7074 )
 Branches     : 84.55% ( 2648/3132 )
 Functions    : 93.07% ( 860/924 )
 Lines        : 91.79% ( 6449/7026 )
 ================================================================================
 [11:31:37] Finished 'test' after 18 min
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.3:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all

**Step 5.4:** Enter this command to see that quite a few Docker images for chaincode have been created as part of the test.  
These are the images that start with *dev-*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 
**Step 5.5:** You will now clean up. You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 
**Step 5.6:** Now observe that all Docker containers have been stopped and most have been removed by entering this command::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all
 CONTAINER ID        IMAGE                                                                                                    COMMAND                  CREATED             STATUS                          PORTS               NAMES
 1401b5aeeceb        dev-peer0.org2.example.com-second-v10-5714f9445c9ccd0fd2642a3a170d60848b55d4c0efff20d5b2edb9dedfd6f4d7   "/bin/sh -c 'cd /usr…"   18 minutes ago      Exited (0) 9 minutes ago                            dev-peer0.org2.example.com-second-v10
 45480b06c3fe        dev-peer0.org1.example.com-second-v10-7ac564a300ba156f1849b973e08e3fb8661959e16651ae0b3ca349c870799248   "/bin/sh -c 'cd /usr…"   18 minutes ago      Exited (0) About a minute ago                       dev-peer0.org1.example.com-second-v10
 16c07e6b8661        dev-peer0.org1.example.com-first-v10-6d77548f00e63ff9ef8c69c1684578b171e0eb81d0135182da6245e0b0e66124    "/bin/sh -c 'cd /usr…"   20 minutes ago      Exited (0) 52 seconds ago                           dev-peer0.org1.example.com-first-v10
 a153ed0d5010        dev-peer0.org2.example.com-first-v10-089fb6ff0168a2b4fe602be3a12d069e056c2e136bc6a5978716b2bb48848615    "/bin/sh -c 'cd /usr…"   20 minutes ago      Exited (0) 52 seconds ago                           dev-peer0.org2.example.com-first-v10

**Note:** The output of this command shows a few containers in the *Exited* state, but none in the *Up* state.  In golden bygone days of lore (approximately summer of 2018, if you live in the northern hemisphere), the cleanup command from *Step 5.5* tended to remove all containers, so that none were left behind, even in the *Exited* state.  But then an evil dragon entered the land and caused the creation of new container names that the cleanup commands did not check for, allowing these containers to remain as a blight upon the land of Hyperledger Fabric Node.js SDK.  Until a future Prince Charming or Joan of Arc rises up to submit a change request to slay these wicked containers, we will have to take extraordinary steps (*5.8* and *5.10* to be precise) in order to rid our system of these containers and their associated images and free the Hyperledger Fabric peasantry of this pestilence.

**Step 5.7:** See the above note and then proceed, O' brave one!::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 REPOSITORY                                                                                               TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.org2.example.com-second-v10-5714f9445c9ccd0fd2642a3a170d60848b55d4c0efff20d5b2edb9dedfd6f4d7   latest              fbce3d7767e1        23 minutes ago      1.52GB
 dev-peer0.org1.example.com-second-v10-7ac564a300ba156f1849b973e08e3fb8661959e16651ae0b3ca349c870799248   latest              214cd785c0b8        23 minutes ago      1.52GB
 dev-peer0.org1.example.com-first-v10-6d77548f00e63ff9ef8c69c1684578b171e0eb81d0135182da6245e0b0e66124    latest              54de5cfcca0c        25 minutes ago      1.52GB
 dev-peer0.org2.example.com-first-v10-089fb6ff0168a2b4fe602be3a12d069e056c2e136bc6a5978716b2bb48848615    latest              da9c92f1be1f        25 minutes ago      1.52GB
 
**Step 5.8:** Let's clean up the Docker containers and images that were left behind:: 

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker rm $(docker ps --all --quiet)
 1401b5aeeceb
 45480b06c3fe
 16c07e6b8661
 a153ed0d5010

**Step 5.9:** Now verify that those containers are gone::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 5.10:** Now remove the Docker chaincode images::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker rmi $(docker images --quiet dev-*)
 Untagged: dev-peer0.org2.example.com-second-v10-5714f9445c9ccd0fd2642a3a170d60848b55d4c0efff20d5b2edb9dedfd6f4d7:latest
 Deleted: sha256:fbce3d7767e1930da50b338d49775991aa15be18afa2e88eac18f726033f5a2f
 Deleted: sha256:777e3a6c96b5781545de94cff7848c9c30d0ce96c3cf64df58d4f9b26aa7ffff
 Deleted: sha256:62ba18e90a475193fe18ee52604fc2f38e21c34e557462f95375d82e7935fe4c
 Deleted: sha256:356a5ab2878cedcdbe50b2badf0bfcb1e95b02ca7d64ce6292fa793b8ac44964
 Untagged: dev-peer0.org1.example.com-second-v10-7ac564a300ba156f1849b973e08e3fb8661959e16651ae0b3ca349c870799248:latest
 Deleted: sha256:214cd785c0b8efae917a77d02c65e3005d132b74f5bbd3191dcc03fe85627a56
 Deleted: sha256:d5cd6079cb3477d9e1280550f31eb8b9f8b855a96e01098fcbd9309aafe60442
 Deleted: sha256:4df132c70a05f72198f937120789c6ae743a94f21d78dc5ae2456ff76ab11efd
 Deleted: sha256:ba8eec44118cc0fb559ccdd41d24824f178816a5922ac193f969b35c2553c38a
 Untagged: dev-peer0.org2.example.com-first-v10-089fb6ff0168a2b4fe602be3a12d069e056c2e136bc6a5978716b2bb48848615:latest
 Deleted: sha256:da9c92f1be1fcfae1380e7312e880b12108ad88b8c9ee8d4f52f374ea63894bb
 Deleted: sha256:2f76702f6fc66a0a1f95dc5b1a79f46f9924d752e256db8ec42a07a7d9031b72
 Deleted: sha256:82c003b01fc4adc48a82194650e37288eae4b9ff51da18ef6abfd4e6ef16cee1
 Deleted: sha256:61b1e05339e70d996e31d94d274d5e4059562bae76eefcb20ffe1a7f28fc4f09
 Untagged: dev-peer0.org1.example.com-first-v10-6d77548f00e63ff9ef8c69c1684578b171e0eb81d0135182da6245e0b0e66124:latest
 Deleted: sha256:54de5cfcca0c79969880cc1fdb03d0f95bd5c7bc2ea66afae2bfa0865df74def
 Deleted: sha256:cc579055dfb3a8f13dcb5875c5b291c1a9909b8d539ea94299d09b21110ed529
 Deleted: sha256:33a48adcfc981a5b7aac17f264b2fc3d28ef1cdcf4cc20bdc6904d8404daee77
 Deleted: sha256:203a87318e0edaad7fcec7a639a158be46bfad80b4b4aeffd996347f9fff99fc
 
**Step 5.11:** Verify that the Docker chaincode images are gone::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Recap:** In this section,you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward. This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
