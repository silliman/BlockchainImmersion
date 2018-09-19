Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.4 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.4 LTS* and the Linux kernel is at level *4.4.0-127*.  In addtion, several software prerequisites have already been installed on this instance for you, including:

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

 bcuser@ubuntu16044:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16044:~$
 
**Step 2.2:** Navigate to the directory you just created::

 bcuser@ubuntu16044:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$
 
**Step 2.3:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place where it lives.  The *-b v1.1.1* argument specifies that you want the v1.1.1 release level::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ git clone -b v1.1.1 https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 5640, done
 remote: Finding sources: 100% (45/45)
 remote: Total 71382 (delta 2), reused 71362 (delta 2)
 Receiving objects: 100% (71382/71382), 88.27 MiB | 30.15 MiB/s, done.
 Resolving deltas: 100% (32176/32176), done.
 Checking connectivity... done.
 Note: checking out 'df84b5b3c3014c3f6cf469df21766dcd645ecb7e'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

*Note:* The numbers in the various output messages may differ from what you see listed here, and this may be the case for any other times you do a *git clone* in the remainder of these labs.

**Step 2.4:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$

**Step 2.5:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 2.6:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real	2m21.217s
 user	0m3.129s
 sys	0m0.598s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ 

**Step 2.7:** Run *docker images* again and you will see several Docker images that were just created. You will notice that many of the Docker images at the top of the output were created in the last few minutes.  These were created by the *make docker* command.  The Docker images that are several months old were downloaded from the Hyperledger Fabric's public DockerHub repository.  Your output should look similar to that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-tools       latest              592d921603b8        34 seconds ago       1.37GB
 hyperledger/fabric-tools       s390x-1.1.1         592d921603b8        34 seconds ago       1.37GB
 hyperledger/fabric-orderer     latest              5491eacc2ca8        About a minute ago   203MB
 hyperledger/fabric-orderer     s390x-1.1.1         5491eacc2ca8        About a minute ago   203MB
 hyperledger/fabric-peer        latest              7a635dfa9f90        About a minute ago   210MB
 hyperledger/fabric-peer        s390x-1.1.1         7a635dfa9f90        About a minute ago   210MB
 hyperledger/fabric-javaenv     latest              9eb789f8a6fa        About a minute ago   1.38GB
 hyperledger/fabric-javaenv     s390x-1.1.1         9eb789f8a6fa        About a minute ago   1.38GB
 hyperledger/fabric-ccenv       latest              c794e7f5895f        About a minute ago   1.3GB
 hyperledger/fabric-ccenv       s390x-1.1.1         c794e7f5895f        About a minute ago   1.3GB
 hyperledger/fabric-baseimage   s390x-0.4.6         234d9beb079b        5 months ago         1.27GB
 hyperledger/fabric-baseos      s390x-0.4.6         0eaed2e8996f        5 months ago         173MB

**Step 2.8:** Navigate to the directory one level above where the “end-to-end” test lives::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ cd examples
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$

**Step 2.9:** Recent updates to Docker Compose have changed how the default Docker network is named. Previously, it created a network named *directory*_default where *directory* is the current directory with special characters like underscores removed.  For instance, the test we want to run lives in the *e2e_cli* directory and Docker would create an internal network named *e2ecli_default*.  The end-to-end test supplied by Hyperledger Fabric is configured to expect a network of this name.  Now, however, Docker Compose will leave the underscore in, and create an internal network named *e2e_cli_default*, which breaks the test we want to run unless we take corrective measures, which we're about to do.  The more elegant solution would be to change the value of the *CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE* environment variable set in *examples/e2e_cli/base/peer-base.yaml* from *e2ecli_default* to *e2e_cli_default*.  But a sneakier way to get around this for the purposes of this lab is to simply rename the directory from *e2e_cli*.  Let's do that::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$ mv -iv e2e_cli/ e2ecli
 'e2e_cli/' -> 'e2ecli'

**Step 2.10** Now navigate to your newly renamed *e2ecli* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$ cd e2ecli 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ 
 
**Step 2.11:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a process, and it is based on a Docker image.  Run this command, which shows all Docker containers, however right now there will be no output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 2.12:** Run the end-to-end test with this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 2.13:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                     PORTS                                                                       NAMES
 42132dad36be        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   14 seconds ago       Up 13 seconds                                                                                          dev-peer1.org2.example.com-mycc-1.0
 81ff563d5c09        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   30 seconds ago       Up 29 seconds                                                                                          dev-peer0.org1.example.com-mycc-1.0
 eadb67f7477b        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   45 seconds ago       Up 45 seconds                                                                                          dev-peer0.org2.example.com-mycc-1.0
 37e6c4fa1003        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   About a minute ago   Exited (0) 3 seconds ago                                                                               cli
 c6b26ad1d4ab        hyperledger/fabric-orderer                                                                             "orderer"                About a minute ago   Up About a minute          0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 39e6f8698756        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka1
 cb000953d37e        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka3
 1f8135bfcb30        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka0
 86f0ab27c1c6        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute          9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka2
 0fcad568f5df        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 c59196866ae5        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 aed77f23d860        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 5e82d29722c6        hyperledger/fabric-peer                                                                                "peer node start"        About a minute ago   Up About a minute          0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 89ea97b667e5        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 8b4848033b49        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 ddfa455d2334        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 e658fffbbcb9        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32774->3888/tcp   zookeeper0
 d67009d7b074        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32773->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper1
 459aea2a7640        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   About a minute ago   Up About a minute          4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 d8c1ad496c5e        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   About a minute ago   Up About a minute          0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper2

The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one orderer container. The orderer service uses *Kafka* for consensus, and so is supported by four Kafka containers and three Zookeeper containers. There was a container created to run the CLI itself, and that container stopped running ten seconds after the test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 2.14:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ ./network_setup.sh down

**Step 2.15:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Recap:** In this section, you:

*	Downloaded the main Hyperledger Fabric source code repository
*	Installed prerequisite tools required to build the Hyperledger Fabric project
*	Ran *make* to build the project’s Docker images
*	Ran the Hyperledger Fabric command line interface (CLI) end-to-end test
*	Cleaned up afterwards
 
Section 3: Install the Hyperledger Fabric Certificate Authority
===============================================================

In the prior section, the end-to-end test that you ran supplied its own security-related material such as keys and certificates- everything it needed to perform its test.  Therefore it did not need the services of a Certificate Authority.

Almost all "real world" Hyperledger Fabric networks will not be this static-  new users, peers and organizations will probably join the network.  They will need PKI x.509 certificates in order to participate.  The Hyperledger Fabric Certificate Authority (CA) is provided by the Hyperledger Fabric project in order to issue these certificates.

The next major goal in this lab is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  This test makes calls to the Hyperledger Fabric Certificate Authority (CA). Therefore, before we can run that test, you will get started by downloading and building the Hyperledger Fabric CA.

**Step 3.1:** Use *cd* to navigate three directory levels up, to the *hyperledger* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ cd ~/git/src/github.com/hyperledger
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$

**Step 3.2:** Get the source code for the v1.2.0 release of the Fabric CA using *git*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ git clone -b v1.2.0 https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 17, done
 remote: Total 11356 (delta 0), reused 11356 (delta 0)
 Receiving objects: 100% (11356/11356), 26.27 MiB | 20.07 MiB/s, done.
 Resolving deltas: 100% (4001/4001), done.
 Checking connectivity... done.
 Note: checking out '3bcdbb2bb9f46c7eb705c9de8b9bb002c5c15fe3'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 3.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.4:** Enter the following command, which will build the Hyperledger Fabric CA images.  Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca $ time FABRIC_CA_DYNAMIC_LINK=true make docker
   .
   .  (output not shown here)
   .
 real	5m0.509s
 user	0m0.148s
 sys	0m0.195s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.5:** Enter the *docker images* command and you will see at the top of the output the Docker images that were just created for the Fabric Certificate Authority::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca-tools                               latest                         519ea75ef41e        2 minutes ago       1.54GB
 hyperledger/fabric-ca-tools                               s390x-1.2.1-snapshot-3bcdbb2   519ea75ef41e        2 minutes ago       1.54GB
 hyperledger/fabric-ca-peer                                latest                         e66fe1ffd608        3 minutes ago       298MB
 hyperledger/fabric-ca-peer                                s390x-1.2.1-snapshot-3bcdbb2   e66fe1ffd608        3 minutes ago       298MB
 hyperledger/fabric-ca-orderer                             latest                         91e4837359d1        4 minutes ago       292MB
 hyperledger/fabric-ca-orderer                             s390x-1.2.1-snapshot-3bcdbb2   91e4837359d1        4 minutes ago       292MB
 hyperledger/fabric-ca                                     latest                         9205d55c0dec        6 minutes ago       316MB
 hyperledger/fabric-ca                                     s390x-1.2.1-snapshot-3bcdbb2   9205d55c0dec        6 minutes ago       316MB

   .
   . (remaining output not shown here)
   .

You may have noticed that for many of the images, the *Image ID* appears twice, once with a tag of *latest*, and once with a tag such as *s390x-1.2.1-snapshot-3bcdbb2* or *s390x-1.1.1*. An image can actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive tag, *s390x-1.1.1* in the case of the *fabric* repo, and *s390x-1.2.1-snapshot-3bcdbb2* in the case of the *fabric-ca* repo, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.2.1-snapshot-3bcdbb2*, and in all three cases you are asking for the same image, the image with ID *9205d55c0dec*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 4: Install Hyperledger Fabric Node.js SDK
=================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 4.1:** Back up one directory level to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$

**Step 4.2:** Now you will download the version 1.1.1 release of the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger $ git clone -b v1.1.1 https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 438, done
 remote: Finding sources: 100% (12/12)
 remote: Total 9237 (delta 0), reused 9227 (delta 0)
 Receiving objects: 100% (9237/9237), 6.33 MiB | 10.68 MiB/s, done.
 Resolving deltas: 100% (4558/4558), done.
 Checking connectivity... done.
 Note: checking out '96c18d5b990983b377b0d4139f3ee112f9576f13'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 4.3:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.4:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.1.0 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.5:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 1026 packages in 73.652s

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 4.6:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Installed Node.js and npm
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
 
Section 5: Run the Hyperledger Fabric Node.js SDK end-to-end test
=================================================================
In this section, you will run two tests provided by the Hyperledger Fabric Node.js SDK, verify their successful operation, and clean up afterwards.

The first test is a quick test that takes a little over twenty seconds, and does not bring up any chaincode containers.  The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take several minutes.

**Step 5.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
 1..1093
 # tests 1093
 # pass  1093

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    65.29 |     61.4 |    55.26 |    65.29 |                |
   AffiliationService.js        |    66.67 |       70 |      100 |    66.67 |... 194,195,198 |
   FabricCAClientImpl.js        |    64.26 |    61.02 |    48.33 |    64.26 |... 924,926,929 |
   IdentityService.js           |    65.79 |       50 |    66.67 |    65.79 |... 254,255,258 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    68.43 |     63.1 |    71.96 |    68.57 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |     71.5 |       52 |    72.22 |    71.83 |... 4,1326,1327 |
   CertificateAuthority.js      |      100 |      100 |      100 |      100 |                |
   Channel.js                   |    49.58 |     45.4 |    57.89 |    49.47 |... 1,2343,2346 |
   ChannelEventHub.js           |    62.98 |    55.46 |    65.22 |    63.34 |... 3,1294,1296 |
   Client.js                    |    73.46 |    73.12 |    77.94 |    73.49 |... 7,1930,1933 |
   Config.js                    |    91.43 |       75 |      100 |    91.43 |      65,83,100 |
   Constants.js                 |      100 |      100 |      100 |      100 |                |
   EventHub.js                  |    69.91 |    65.85 |    67.74 |    70.37 |... 821,826,833 |
   Orderer.js                   |       50 |    35.71 |     62.5 |       50 |... 285,286,289 |
   Organization.js              |    84.78 |       80 |    93.33 |    86.05 |... 122,123,126 |
   Packager.js                  |    91.67 |    91.67 |      100 |    91.67 |          57,58 |
   Peer.js                      |    80.43 |     62.5 |    88.89 |    80.43 |... 140,142,143 |
   Policy.js                    |    99.07 |    92.16 |      100 |    99.07 |            169 |
   Remote.js                    |     97.8 |       90 |      100 |     97.8 |        102,114 |
   TransactionID.js             |       96 |     87.5 |      100 |       96 |             48 |
   User.js                      |    88.24 |    67.31 |       80 |    88.24 |... 226,246,253 |
   api.js                       |    33.33 |      100 |        0 |    33.33 |... 171,184,198 |
   client-utils.js              |    73.95 |    58.97 |    73.33 |    73.95 |... 223,225,227 |
   hash.js                      |      100 |      100 |    94.74 |      100 |                |
   utils.js                     |    79.24 |    72.88 |    77.14 |    79.24 |... 402,461,540 |
  fabric-client/lib/impl/       |    65.15 |    59.83 |    61.46 |     65.1 |                |
   CouchDBKeyValueStore.js      |    76.71 |       60 |    93.33 |    77.46 |... 156,169,170 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    86.08 |    71.84 |       80 |    86.08 |... 345,362,368 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |    98.74 |    85.63 |      100 |    98.71 |    146,411,412 |
   bccsp_pkcs11.js              |    24.79 |    32.92 |     8.33 |    24.79 |... 5,1089,1090 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    48.87 |    36.05 |       50 |    50.78 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |     9.33 |        0 |        0 |       10 |... 159,160,162 |
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
 All files                      |    67.67 |    61.19 |    67.43 |    67.82 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 67.67% ( 4057/5995 )
 Branches     : 61.19% ( 1701/2780 )
 Functions    : 67.43% ( 470/697 )
 Lines        : 67.82% ( 4036/5951 )
 ================================================================================
 [16:20:41] Finished 'test-headless' after 23 s



You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this is that of the 1093 tests, all of them passed.  


**Step 5.2:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..1817
 # tests 1817
 # pass  1817

 # ok

 -------------------------------|----------|----------|----------|----------|----------------|
 File                           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 -------------------------------|----------|----------|----------|----------|----------------|
  fabric-ca-client/lib/         |    94.36 |    89.47 |    90.79 |    94.36 |                |
   AffiliationService.js        |    98.33 |       96 |      100 |    98.33 |            195 |
   FabricCAClientImpl.js        |    94.04 |     88.7 |       90 |    94.04 |... 918,926,929 |
   IdentityService.js           |    92.11 |    84.62 |    88.89 |    92.11 |... 246,249,255 |
   helper.js                    |      100 |      100 |      100 |      100 |                |
  fabric-client/lib/            |    89.03 |    79.26 |    87.61 |    89.19 |                |
   BaseClient.js                |     96.3 |    91.67 |      100 |     96.3 |            119 |
   BlockDecoder.js              |    91.71 |       65 |    98.15 |    92.17 |... 8,1323,1324 |
   CertificateAuthority.js      |      100 |      100 |      100 |      100 |                |
   Channel.js                   |    85.23 |    72.42 |    85.53 |    85.35 |... 1,2343,2346 |
   ChannelEventHub.js           |    88.55 |    82.77 |    89.13 |    88.68 |... 2,1289,1296 |
   Client.js                    |    90.56 |    83.37 |    89.71 |    90.52 |... 7,1930,1933 |
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
   api.js                       |    33.33 |      100 |        0 |    33.33 |... 171,184,198 |
   client-utils.js              |    94.12 |    76.92 |      100 |    94.12 |... 210,223,225 |
   hash.js                      |      100 |      100 |    94.74 |      100 |                |
   utils.js                     |    79.66 |    74.58 |    77.14 |    79.66 |... 402,461,540 |
  fabric-client/lib/impl/       |    66.04 |     60.7 |     62.5 |    65.88 |                |
   CouchDBKeyValueStore.js      |    87.67 |       70 |      100 |    87.32 |... 156,169,170 |
   CryptoKeyStore.js            |      100 |     87.5 |      100 |      100 |                |
   CryptoSuite_ECDSA_AES.js     |    86.08 |    71.84 |       80 |    86.08 |... 345,362,368 |
   FileKeyValueStore.js         |    91.89 |    83.33 |      100 |    91.89 |       56,57,74 |
   NetworkConfig_1_0.js         |    98.74 |    86.78 |      100 |    98.71 |    146,411,412 |
   bccsp_pkcs11.js              |    24.79 |    32.92 |     8.33 |    24.79 |... 5,1089,1090 |
  fabric-client/lib/impl/aes/   |    11.11 |        0 |        0 |    11.11 |                |
   pkcs11_key.js                |    11.11 |        0 |        0 |    11.11 |... 52,56,60,64 |
  fabric-client/lib/impl/ecdsa/ |    48.87 |    36.05 |       50 |    50.78 |                |
   key.js                       |      100 |    96.88 |      100 |      100 |                |
   pkcs11_key.js                |     9.33 |        0 |        0 |       10 |... 159,160,162 |
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
 All files                      |    84.45 |    74.28 |    81.92 |    84.64 |                |
 -------------------------------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 84.45% ( 5063/5995 )
 Branches     : 74.28% ( 2065/2780 )
 Functions    : 81.92% ( 571/697 )
 Lines        : 84.64% ( 5037/5951 )
 ================================================================================
 [16:32:03] Finished 'test' after 8.1 min
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.3:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a

**Step 5.4:** Enter this command to see that some Docker images for chaincode have been created as part of the test.  These are the images that start with *dev-*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 
**Step 5.5:** You will now clean up. You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 
**Step 5.6:** Now observe that all Docker containers have been stopped and removed by entering this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 
**Step 5.7:** And enter this comand and see that all chaincode images (those starting with *dev-*) have been removed::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward. This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
