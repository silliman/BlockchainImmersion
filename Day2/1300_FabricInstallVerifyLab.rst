Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.5 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.5 LTS* and the Linux kernel is at level *4.4.0-137*.  In addtion, several software prerequisites have already been installed on this instance for you, including:

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

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 79472, done
 remote: Finding sources: 100% (79472/79472)
 remote: Total 79472 (delta 35136), reused 79434 (delta 35136)
 Receiving objects: 100% (79472/79472), 97.64 MiB | 26.29 MiB/s, done.
 Resolving deltas: 100% (35136/35136), done.
 Checking connectivity... done.

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
 hyperledger/fabric-tools       latest                         eb61a4372d2d        49 seconds ago      1.52GB
 hyperledger/fabric-tools       s390x-1.4.0-snapshot-5caab9b   eb61a4372d2d        49 seconds ago      1.52GB
 hyperledger/fabric-tools       s390x-latest                   eb61a4372d2d        49 seconds ago      1.52GB
 <none>                         <none>                         78ffc26ff38e        58 seconds ago      1.65GB
 hyperledger/fabric-testenv     latest                         8bb2f2157a7f        2 minutes ago       1.57GB
 hyperledger/fabric-testenv     s390x-1.4.0-snapshot-5caab9b   8bb2f2157a7f        2 minutes ago       1.57GB
 hyperledger/fabric-testenv     s390x-latest                   8bb2f2157a7f        2 minutes ago       1.57GB
 hyperledger/fabric-buildenv    latest                         d7ac7af63798        2 minutes ago       1.47GB
 hyperledger/fabric-buildenv    s390x-1.4.0-snapshot-5caab9b   d7ac7af63798        2 minutes ago       1.47GB
 hyperledger/fabric-buildenv    s390x-latest                   d7ac7af63798        2 minutes ago       1.47GB
 hyperledger/fabric-ccenv       latest                         1fd333963a9c        3 minutes ago       1.41GB
 hyperledger/fabric-ccenv       s390x-1.4.0-snapshot-5caab9b   1fd333963a9c        3 minutes ago       1.41GB
 hyperledger/fabric-ccenv       s390x-latest                   1fd333963a9c        3 minutes ago       1.41GB
 hyperledger/fabric-orderer     latest                         7269c1176d63        3 minutes ago       145MB
 hyperledger/fabric-orderer     s390x-1.4.0-snapshot-5caab9b   7269c1176d63        3 minutes ago       145MB
 hyperledger/fabric-orderer     s390x-latest                   7269c1176d63        3 minutes ago       145MB
 hyperledger/fabric-peer        latest                         63177913a293        4 minutes ago       151MB
 hyperledger/fabric-peer        s390x-1.4.0-snapshot-5caab9b   63177913a293        4 minutes ago       151MB
 hyperledger/fabric-peer        s390x-latest                   63177913a293        4 minutes ago       151MB
 hyperledger/fabric-baseimage   s390x-0.4.14                   6e4e09df1428        9 days ago          1.38GB
 hyperledger/fabric-baseos      s390x-0.4.14                   4834a1e3ce1c        9 days ago          120MB

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
 09b4a2c28e87        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer1.org2.example.com-mycc-1.0
 6a8f1936acf3        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer0.org1.example.com-mycc-1.0
 5532a4d94a57        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer0.org2.example.com-mycc-1.0
 a504007136dc        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   2 minutes ago        Exited (0) 48 seconds ago                                                                               cli
 ef1e3f73e632        hyperledger/fabric-orderer                                                                             "orderer"                2 minutes ago        Up 2 minutes                0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 0a65a1fae055        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka0
 cfd398ab4b8c        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka1
 71b76b641f25        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka3
 6963bd5d1ab9        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka2
 429776cada25        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 434ac0f8aa04        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 cdb9e8b9c885        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 ed8f768148c7        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 a3e9401ac3ef        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32775->2181/tcp, 0.0.0.0:32773->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper0
 3e549248300d        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32776->2181/tcp, 0.0.0.0:32774->2888/tcp, 0.0.0.0:32772->3888/tcp   zookeeper1
 ebbff1a3aa12        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 653584c0d8f3        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 82a9dba290e8        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper2
 fb0749622771        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 ac659714affb        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0

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

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 1697, done
 remote: Finding sources: 100% (61/61)
 remote: Total 11956 (delta 1), reused 11953 (delta 1)
 Receiving objects: 100% (11956/11956), 26.97 MiB | 20.56 MiB/s, done.
 Resolving deltas: 100% (4189/4189), done.
 Checking connectivity... done.

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
 hyperledger/fabric-ca          latest                         7a3fa3cd6f4c        2 minutes ago       317MB
 hyperledger/fabric-ca          s390x-1.4.0-snapshot-bd7f997   7a3fa3cd6f4c        2 minutes ago       317MB
 hyperledger/fabric-tools       latest                         eb61a4372d2d        38 minutes ago      1.52GB
 hyperledger/fabric-tools       s390x-1.4.0-snapshot-5caab9b   eb61a4372d2d        38 minutes ago      1.52GB
 hyperledger/fabric-tools       s390x-latest                   eb61a4372d2d        38 minutes ago      1.52GB
 hyperledger/fabric-testenv     latest                         8bb2f2157a7f        40 minutes ago      1.57GB
 hyperledger/fabric-testenv     s390x-1.4.0-snapshot-5caab9b   8bb2f2157a7f        40 minutes ago      1.57GB
 hyperledger/fabric-testenv     s390x-latest                   8bb2f2157a7f        40 minutes ago      1.57GB
 hyperledger/fabric-buildenv    latest                         d7ac7af63798        40 minutes ago      1.47GB
 hyperledger/fabric-buildenv    s390x-1.4.0-snapshot-5caab9b   d7ac7af63798        40 minutes ago      1.47GB
 hyperledger/fabric-buildenv    s390x-latest                   d7ac7af63798        40 minutes ago      1.47GB
 hyperledger/fabric-ccenv       latest                         1fd333963a9c        41 minutes ago      1.41GB
 hyperledger/fabric-ccenv       s390x-1.4.0-snapshot-5caab9b   1fd333963a9c        41 minutes ago      1.41GB
 hyperledger/fabric-ccenv       s390x-latest                   1fd333963a9c        41 minutes ago      1.41GB
 hyperledger/fabric-orderer     latest                         7269c1176d63        41 minutes ago      145MB
 hyperledger/fabric-orderer     s390x-1.4.0-snapshot-5caab9b   7269c1176d63        41 minutes ago      145MB
 hyperledger/fabric-orderer     s390x-latest                   7269c1176d63        41 minutes ago      145MB
 hyperledger/fabric-peer        latest                         63177913a293        42 minutes ago      151MB
 hyperledger/fabric-peer        s390x-1.4.0-snapshot-5caab9b   63177913a293        42 minutes ago      151MB
 hyperledger/fabric-peer        s390x-latest                   63177913a293        42 minutes ago      151MB
 hyperledger/fabric-zookeeper   latest                         5db059b03239        9 days ago          1.42GB
 hyperledger/fabric-kafka       latest                         3bbd80f55946        9 days ago          1.43GB
 hyperledger/fabric-couchdb     latest                         7afa6ce179e6        9 days ago          1.55GB
 hyperledger/fabric-baseimage   s390x-0.4.14                   6e4e09df1428        9 days ago          1.38GB
 hyperledger/fabric-baseos      s390x-0.4.14                   4834a1e3ce1c        9 days ago          120MB

You may have noticed that for many of the images, the *Image ID* appears more than once, e.g., once with a tag of *latest*,  once with a tag such as *s390x-1.4.0-snapshot-5caab9b*, and once with a tag of *s390x-latest*. An image can actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive tag, *s390x-1.4.0-snapshot-5caab9b* in the case of the *fabric* repo, and *s390x-1.4.0-snapshot-bd7f997* in the case of the *fabric-ca* repo, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.4.0-snapshot-bd7f997*, and in all three cases you are asking for the same image, the image with ID *7a3fa3cd6f4c*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 4: Install Hyperledger Fabric Node.js SDK
=================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 4.1:** Back up one directory level to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 4.2:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ git clone https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 643, done
 remote: Finding sources: 100% (6/6)
 remote: Total 11208 (delta 0), reused 11203 (delta 0)
 Receiving objects: 100% (11208/11208), 7.93 MiB | 6.39 MiB/s, done.
 Resolving deltas: 100% (5482/5482), done.
 Checking connectivity... done.

**Step 4.3:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.4:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.3.0-snapshot /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)



   ╭─────────────────────────────────────╮
   │                                     │
   │   Update available 5.6.0 → 6.4.1    │
   │     Run npm i -g npm to update      │
   │                                     │
   ╰─────────────────────────────────────╯

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

You may ignore the message concerning the available update to npm here and throughout the remainder of these labs.

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
 1..640
 # tests 640
 # pass  640

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
 
 1..1964
 # tests 1964
 # pass  1964

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

**Note:** The output of this command shows a few containers in the *Exited* state, but none in the *Up* state.  Over the course of the Hyperledger project, the cleanup command from *Step 5.5* tended to remove all containers, so that none were left behind even in the *Exited* state.  I suspect that this is just something that slipped through the cracks in a recent update and will probably be corrected in the future.

**Step 5.7:** And enter this comand and see that only a few chaincode images remain- those starting with *dev-* remain- again, related to the note at the end of the previous step, I suspect that a future fix will ensure that these images are deleted.

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
