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

*	Install some support packages using the Ubuntu package manager, *apt-get*
*	Download the source code repository containing the core Hyperledger Fabric functionality
*	Use the source code to build Docker images that contain the core Hyperledger Fabric functionality
*	Test for success by running the comprehensive end-to-end CLI test.

**Step 2.1:** There are some software packages necessary to be able to successfully build the Hyperledger Fabric source code.  Install them with 
this command. Observe the output, not shown here, to see the different packages 
installed::

 bcuser@ubuntu16044:~$ sudo apt-get install -y build-essential libltdl3-dev
 
**Step 2.2:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it. If you are following these steps exactly, you already are.  If you strayed away from your home directory, I'm assuming you're smart enough to get back there or at least smart enough to ask for help::

 bcuser@ubuntu16044:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16044:~$
 
**Step 2.3:** Navigate to the directory you just created::

 bcuser@ubuntu16044:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$
 
**Step 2.4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place where it lives.  The *-b v1.1.0* argument specifies that you want the v1.1.0 release level::

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

**Step 2.5:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$

**Step 2.6:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 2.7:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real	3m28.401s
 user	0m9.073s
 sys	0m0.818s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ 

**Step 2.8:** Run *docker images* again and you will see several Docker images that were just created. You will notice that many of the Docker images at the top of the output were created in the last few minutes.  These were created by the *make docker* command.  The Docker images that are several days or weeks old were downloaded from the Hyperledger Fabric's public DockerHub repository.  Your output should look similar to that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-tools       latest              391ad70c436f        2 minutes ago       1.37GB
 hyperledger/fabric-tools       s390x-1.1.0         391ad70c436f        2 minutes ago       1.37GB
 hyperledger/fabric-testenv     latest              6701f575c57b        2 minutes ago       1.45GB
 hyperledger/fabric-testenv     s390x-1.1.0         6701f575c57b        2 minutes ago       1.45GB
 hyperledger/fabric-buildenv    latest              371ac7debff2        2 minutes ago       1.36GB
 hyperledger/fabric-buildenv    s390x-1.1.0         371ac7debff2        2 minutes ago       1.36GB
 hyperledger/fabric-orderer     latest              19f475dc1361        3 minutes ago       203MB
 hyperledger/fabric-orderer     s390x-1.1.0         19f475dc1361        3 minutes ago       203MB
 hyperledger/fabric-peer        latest              e0252e5c78fe        3 minutes ago       210MB
 hyperledger/fabric-peer        s390x-1.1.0         e0252e5c78fe        3 minutes ago       210MB
 hyperledger/fabric-javaenv     latest              1eee38eda466        3 minutes ago       1.38GB
 hyperledger/fabric-javaenv     s390x-1.1.0         1eee38eda466        3 minutes ago       1.38GB
 hyperledger/fabric-ccenv       latest              662e666889a9        3 minutes ago       1.3GB
 hyperledger/fabric-ccenv       s390x-1.1.0         662e666889a9        3 minutes ago       1.3GB
 hyperledger/fabric-zookeeper   latest              103c1abf45ff        3 months ago        1.34GB
 hyperledger/fabric-zookeeper   s390x-0.4.6         103c1abf45ff        3 months ago        1.34GB
 hyperledger/fabric-kafka       latest              db99e941fe20        3 months ago        1.35GB
 hyperledger/fabric-kafka       s390x-0.4.6         db99e941fe20        3 months ago        1.35GB
 hyperledger/fabric-couchdb     latest              2aecbce9f786        3 months ago        1.56GB
 hyperledger/fabric-couchdb     s390x-0.4.6         2aecbce9f786        3 months ago        1.56GB
 hyperledger/fabric-baseimage   s390x-0.4.6         234d9beb079b        3 months ago        1.27GB
 hyperledger/fabric-baseos      s390x-0.4.6         0eaed2e8996f        3 months ago        173MB

**Step 2.9:** Navigate to the directory one level above where the “end-to-end” test lives::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric$ cd examples
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$

**Step 2.10:** Recent updates to Docker Compose have changed how the default Docker network is named. Previously, it created a network named *directory*_default where *directory* is the current directory with special characters like underscores removed.  For instance, the test we want to run lives in the *e2e_cli* directory and Docker would create an internal network named *e2ecli_default*.  The end-to-end test supplied by Hyperledger Fabric is configured to expect a network of this name.  Now, however, Docker Compose will leave the underscore in, and create an internal network named *e2e_cli_default*, which breaks the test we want to run unless we take corrective measures, which we're about to do.  The more elegant solution would be to change the value of the *CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE* environment variable set in *examples/e2e_cli/base/peer-base.yaml* from *e2ecli_default* to *e2e_cli_default*.  But a sneakier way to get around this for the purposes of this lab is to simply rename the directory from *e2e_cli*.  Let's do that::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$ mv -iv e2e_cli/ e2ecli
 'e2e_cli/' -> 'e2ecli'

**Step 2.11** Now navigate to your newly renamed *e2ecli* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples$ cd e2ecli 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ 
 
**Step 2.12:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a process, and it is based on a Docker image.  Run this command, which shows all Docker containers, however right now there will be no output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 2.13:** Run the end-to-end test with this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 2.14:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                     PORTS                                                                       NAMES
 519d02f2b334        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer1.org2.example.com-mycc-1.0
 a4cc9a032dcc        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer0.org1.example.com-mycc-1.0
 322d8677d0be        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   About a minute ago   Up About a minute                                                                                       dev-peer0.org2.example.com-mycc-1.0
 57c698c165e8        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   2 minutes ago        Exited (0) 52 seconds ago                                                                               cli
 e11975d657e7        hyperledger/fabric-orderer                                                                             "orderer"                2 minutes ago        Up 2 minutes                0.0.0.0:7050->7050/tcp                                                      orderer.example.com
 142936ca4a36        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32806->9092/tcp                                           kafka1
 8eefa1eefeec        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32803->9092/tcp                                           kafka3
 4c9c1f7b0ab6        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32804->9092/tcp                                           kafka2
 acc84869e60d        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                9093/tcp, 0.0.0.0:32805->9092/tcp                                           kafka0
 fe9d198e2fd9        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 d26f4f029cbf        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 3d31d89bbfa7        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 11f66f4cdb1c        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 59a4840bc798        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 1999d2b73d01        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 7cecd19dce8a        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32802->2181/tcp, 0.0.0.0:32801->2888/tcp, 0.0.0.0:32800->3888/tcp   zookeeper0
 2b44d92cf8da        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 ca181a1184d7        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   2 minutes ago        Up 2 minutes                4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 c259f99f3428        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32799->2181/tcp, 0.0.0.0:32798->2888/tcp, 0.0.0.0:32797->3888/tcp   zookeeper1
 5a0f41510d1b        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   2 minutes ago        Up 2 minutes                0.0.0.0:32796->2181/tcp, 0.0.0.0:32795->2888/tcp, 0.0.0.0:32794->3888/tcp   zookeeper2


The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one orderer container. The orderer service uses *Kafka* for consensus, and so is supported by four Kafka containers and three Zookeeper containers. There was a container created to run the CLI itself, and that container stopped running ten seconds after the test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 2.15:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric/examples/e2ecli$ ./network_setup.sh down

**Step 2.16:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

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

**Step 3.2:** Get the source code for the v1.1.0 release of the Fabric CA using *git*::

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

**Step 3.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.4:** Enter the following command, which will build the Hyperledger Fabric CA images.  Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca $ time make docker
   .
   .  (output not shown here)
   .
 real	2m0.509s
 user	0m0.148s
 sys	0m0.195s
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 3.5:** Enter the *docker images* command and you will see at the top of the output the Docker images that were just created for the Fabric Certificate Authority::

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

You may have noticed that for many of the images, the *Image ID* appears twice, once with a tag of *latest*, and once with a tag such as *s390x-1.1.0*. An image can actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive tag, *s390x-1.1.0*, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.1.0*, and in all three cases you are asking for the same image, the image with ID *2ac752a91a56*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 4: Install Hyperledger Fabric Node.js SDK
=================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 4.1:** Back up one directory level to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$

**Step 4.2:** Now you will download the version 1.1.0 release of the Hyperledger Fabric Node SDK source code from its official repository::

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

**Step 4.3:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.4:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.1.0 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 4.5:** You must install a specific version of *grpc*, which is an npm package implementing the Google Remote Procedure Call protocol. This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install grpc@1.10.0

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 4.6:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 871 packages in 33.141s

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 4.7:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong with that. You can even steal this lab if you want to.
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


**Step 5.2:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

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

**Step 5.3:** (Optional) What I really like about the second end-to-end test is that it cleans itself up really well at the beginning- that is, it will remove any artifacts left running at the end of the prior test, so if you wanted to, you could simply enter *gulp test* again if you'd like to see this for yourself and have several minutes to spare.  If you're pressed for time, skip this step::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (output not shown here)
   . 

**Step 5.4:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a

**Step 5.5:** Enter this command to see that some Docker images for chaincode have been created as part of the test.  These are the images that start with *dev-*::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 
**Step 5.6:** You will now clean up. You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 
**Step 5.7:** Now observe that all Docker containers have been stopped and removed by entering this command::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 
**Step 5.8:** And enter this comand and see that all chaincode images (those starting with *dev-*) have been removed::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward. This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
