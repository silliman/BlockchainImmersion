Immunichain Blockchain Lab
==========================

**How did Immunichain Come About?**

Immunichain came from an IBM internal Blockchain hackathon in early May of 2017. The hackathon generated an use case from an IBMer. Around the same time, an Open Mainframe Project intern Kevin Lee from Illinois University was tasked to make a website demonstrating the benefits of Blockchain of this use case. In early August at SHARE, he showcased his end product - which is what you are going to work with today!

**What are you going to accomplish today?**

The Hyperledger Composer Playground is way to produce Blockchain applications quickly without the heavy technical knowledge. In this lab, we will only be using the web version of Hyperledger Composer. If you were to install Hyperledger Composer on your local machine, you would gain more benefits - like deploying to Hyperledger Fabric and opening up your network to a REST server. Once in the Composer Playground, you will create the Blockchain application with a banana file (.bna) that will make up Immunichain. You will then create assets and participants. After that, we will submit transactions and see how the participants look after those transactions. 

**Pre-Requisites for Mac**

*   Web Browser (Chrome preferably)
*   Command line
*   Internet connectively

**Pre-Requisites for Microsoft**

*   Web Browser (Chrome preferably)
*   Internet connectively
*   Command line


Part 1: Starting and Creating Your Hyperledger Composer Network
===============================================================

**1.** From your web browser, go to https://composer-playground.mybluemix.net/ -- This is the web version of Hyperledger Composer running in a Kubernetes environment.

**2.** You will get a "Welcome" pop-up box with a graphic and a few words. Click on Let's Blockchain. This will take us to the homepage of Hyperledger Composer.

.. image:: images/composer/1.1.png

**3.** You will see a sample business card. That is a great "Welcome to Composer, here is a sample application" meant for the very beginers. Since we are soon-to-be-experts, click on Deploy a New Business Network located to the right of the sample business card. The next screen will show you various samples and give you the availability deploy your own chaincode.

**4.** You have to give a name for your Blockchain Network. Give it a description as well. Then select **empty-business-network**. This will give us a bare bones business network.  

**5.** Then click on Deploy. This will deploy our chaincode/blockchain application onto a Hyperledger Fabric underneath that neither you nor I can see. Basically, it is being stored in your local cache.

**6.** Afterwards, you can come back to the Composer Playground play with some of the other sample business network applications, like animal tracking or vehicle lifecycle. These are found just below the option to deploy empty or sample blockchain applications.

**7.** You will then be taken to Your Wallet. Your wallet is basically a quick, seamless connection to multiple Blockchain connections you might have. You will see later how easy it is. Click on Connect now in order to get connected to your Immunichain network

.. image:: images/composer/1.3.png

**8.** Next, you will see a few files on the left. You can think of these files as the contents that make up your chaincode. Below there are definitions as to what each file is meant for:

*   README.md is meant to give people an overview of the purpose of the business network.
*   Model File contains all of participants, assets and transactions. It gets referred in the script.js file.
*   Access Control is how to grant access to certain participants, basically can they invoke certain transactions or not
*   We don't have a Script.js file in here yet, but the script.js houses all of the javascript that invokes the transactions

**9.** Now, we are going to add some substance to our model file. Begin by clicking on the model file on the left and then delete all of the contents within that file. Then add the below contents. Copy and paste is your very best friend here::

	/* Immunization definitions */

	namespace ibm.wsc.immunichain

	participant Guardian identified by gid {
      	o String gid
      	o String name
	}

	participant MedProvider identified by medid {
     	o String medid
      	o String name
	}

	abstract concept immunization {
      	o String name
      	o String provider
      	o String imdate
	}

	concept immunirecord extends immunization {}

	asset Childform identified by cid {
      	o String cid
      	o String name
      	o String address
      	--> Guardian guardian
      	o String dob
      	--> MedProvider [] medproviders optional
      	o immunirecord [] immunizations
	}

	transaction assignMedProvider {
      	--> Guardian guardian
      	--> MedProvider medprovider
      	--> Childform childform
	}

	transaction authMember {
      	--> Guardian guardian
      	--> Childform childform
	}

	transaction removeMemberAuth {
      	--> Guardian guardian
      	--> Childform childform
	}

	transaction addImmunizations {
      	o immunirecord [] vaccines
      	--> Childform childform
	}

	transaction updateChildForm {
      	o String name optional
      	o String address optional
      	--> Childform childform
	}

	transaction reassignGuardian {
      	--> Guardian oldguardian
      	--> Guardian newguardian
      	--> Childform childform
	}


**10.** Click on **Add a File** towards the bottom of the files section. Now, select **Script File (.js)** and then Add in the highlighted blue. Obviously, this will add a Script file to our network. 

**11.** Just like step 9, delete all of the contents and fill in our file with the information below::

	'use strict';

	/**
      	* Add medical provider to child record
	* @param {ibm.wsc.immunichain.assignMedProvider} assignMedProvider - the assignMedProvider transaction
      	* @transaction
      	*/
	function assignMedProvider(assignMedProvider) {
      	var guardian = assignMedProvider.guardian;
      	var child = assignMedProvider.childform;
      	var medprovider = assignMedProvider.medprovider;
      	child.medproviders.push(medprovider);

      	return getAssetRegistry('ibm.wsc.immunichain.Childform')
      	.then(function(result) {
      	return result.update(child);
      	});
	}

	/**
      	* Add immunization(s) to child record
	* @param {ibm.wsc.immunichain.addImmunizations} addImmunizations - the addImmunizations transaction
      	* @transaction
      	*/
	function addImmunizations(addImmunizations){
      	var vaccines = addImmunizations.vaccines;
      	var child = addImmunizations.childform;
      	var immunizations = child.immunizations;
	/*    if (immunizations[0].name == 'default'){
      	immunizations.splice(0,1)
      	} */
      	immunizations.push.apply(immunizations,vaccines);

      	return getAssetRegistry('ibm.wsc.immunichain.Childform')
              .then(function(ChildRegistry){
                      //save the childform
                      return ChildRegistry.update(child);
              });
	}

	/**
      	* Update information on child record, can only be done by guardian
	* @param {ibm.wsc.immunichain.updateChildForm} updateChildForm - the updateChildForm transaction
      	* @transaction
      	*/
	function updateChildForm(updateChildForm){
      	var newaddress = null;
      	var newname = null;
      	var child = updateChildForm.childform;
      	newaddress = updateChildForm.address;
      	newname = updateChildForm.name;

      	if (newaddress != null && newname != null){
      	child.name = newname;
      	child.address = newaddress;
      	}
      	else if (newaddress != null){
      	child.address = newaddress;
      	}
      	else if (newname != null){
      	child.name = newname;
      	}
      	return getAssetRegistry('ibm.wsc.immunichain.Childform')
              .then(function(ChildRegistry){
                      //save the childform
                      return ChildRegistry.update(child);
              });
	}

	/**
      	* Assign child to his/herself when he/she is of legal age
	* @param {ibm.wsc.immunichain.reassignGuardian} reassignGuardian - the reassignGuardian transaction
	* @transaction
      	*/
	function reassignGuardian(reassignGuardian) {
      	var oldguardian = reassignGuardian.oldguardian;
      	var newguardian = reassignGuardian.newguardian;
      	var child = reassignGuardian.childform;
      	child.guardian = newguardian;

      	return getAssetRegistry('ibm.wsc.immunichain.Childform')
      	.then(function(result) {
      	return result.update(child);
      	});
	}

	/**
      	* Get the immunizations for a child
      	* @query
      	* @param {String} cid - the unique id assigned to the childform
      	* @returns {immunirecord[]} - the immunizations that the child has gotten
	*/
	function listImmunizations(cid) {
      	return query('select x.immunizations from Childform where x.cid ==: cid');
	}
	
**12.** Luckily, we don't have to make changes to our Access Control file right now. Now click on **Deploy Changes.** This will update our network with the specific modifications you just made. Basically, you just add participants, assets, and a script file, which houses all of our transactions in javascript. I highly suggest going through the code we pasted into the files to get a sense as to what is happening. 


Part 2: Creating Assets and Participants
========================================

**1.** Now that you have an Immunichain Business Network from scratch, jump over to the Test section of the Composer Playground. The test area allows you to create assets, participants and submit transactions against your assets and participants. Your screen should look like this: 

.. image:: images/composer/2.1.png

Before we create assets and participants, we need to know what each asset and participants represent. 

*   Guardian is the parent
*   MedProvider is simply a medical provider, like a doctor
*   Childform is simply the child or the asset in this business network

**2.** Now create a Guardian by **clicking Guardian on the left and then +Create New Participant in the top right.** Give the Guardian a number. I stick to 1, 2, 3 or low numbers that you can remember, but you can create any ID number you want. I suggest writing your ID numbers down as we move along. Once you have filled in the information click on Create

.. image:: images/composer/2.2.png

.. image:: images/composer/2.3.1.png

**3.** Once you have created a Guardian, your screen should look like this: 

.. image:: images/composer/2.4.png

**5.** Go ahead and make a Medical Provider. Same process as the guardian; **click on Medical Provider on the left and +Create New Participant in the top right.** Remember the Medical Provider number you create.

.. image:: images/composer/2.5.png

**6.** Now, let's make a child. **Click on optional properties at the bottom first.** Assign your new child to the guardian you just created two steps ago. We clicked optional properties because we can now pass information into those arrays.

.. image:: images/composer/2.6.png

**7.** Your screen should look like this when you are done:

.. image:: images/composer/2.7.png

**8.** Go ahead and create more medical providers, guardians and children. Just to remember to write down the ID numbers. This will make more sense when we submit transactions. 

Part 3: Adding Participants and Transactions
============================================

So far, everything has been a bit easy. Now, we are going to add a participant and some transaction code for that new participant. It is important to notice where I am adding in code in relation to the other lines of code.

**1.** Head into your model file by going to the Define section and clicking on the Model File, found on the left hand side. The model file defines all the participants, assets and transactions. It gives participants attributes like name, email address, id numbers, etc. 

.. image:: images/composer/3.1.1.png

**2.** On line 15, add in this participant::

	participant Member identified by memid {
	o String memid
	o String name
	}

.. image:: images/composer/3.2.png

**3.** On line 35, add in this line in the asset childform. The "-->" pulls in the participant member and calls it members and makes it optional::

	--> Member [] members optional

.. image:: images/composer/3.3.1.png

**4.** On line 47, add in this line in the transaction authMember::

	--> Member member

.. image:: images/composer/3.4.1.png

**5.** On line 54, add in this line in the transaction removeMemberAuth::

	--> Member member

.. image:: images/composer/3.5.1.png

**6.** Then click on Deploy Changes (the 0.19.0 release of Composer changes this button from Update to Deploy Changes. Both do the same thing, but the wording is different), if successful you will get a success message in the top right. You have now deployed a new version of the chaincode. If we were running this locally, you would see a new version of the chaincode represented with Docker Images and Containers.

.. image:: images/composer/3.6.1.png

What other participants or assets could you see being added the Immunichain Blockchain network? Collaborate with a few people around you to gather ideas. Later you can add these participants and assets to your network. 

Now, let's add some transactions.

**7.** Switch to the Script File in the Define Section

.. image:: images/composer/3.7.1.png

**8.** On line 20, add in this transaction. This is javascript, which is looking at participants, assets and transactions located in the model file::

	/**
	 * Authorize member to child record
	 * @param {ibm.wsc.immunichain.authMember} authMember - the authMember transaction
	 * @transaction
	*/
	function authMember(authMember) {
	  var guardian = authMember.guardian;
	  var child = authMember.childform;
	  var member = authMember.member;
	  child.members.push(member);
	return getAssetRegistry('ibm.wsc.immunichain.Childform')
	  .then(function(ChildRegistry) {
		return ChildRegistry.update(child);
	  });
	}

.. image:: images/composer/3.8.1.png

**9.** On line 36, add in this transaction as well::

	/**
	* Deauthorize member to child record, so remove from members list
	* @param {ibm.wsc.immunichain.removeMemberAuth} removeMemberAuth - the removeMemberAuth transaction
	* @transaction
	*/
	function removeMemberAuth(removeMemberAuth) {
	  var guardian = removeMemberAuth.guardian;
	  var child = removeMemberAuth.childform;
	  var member = removeMemberAuth.member;
	  var mem = child.members;
	  var idx = mem.indexOf(member);

	//if the member is in the array of Members, we can remove it
	  if (idx !== -1){
		mem.splice(idx,1);
	}

	return getAssetRegistry('ibm.wsc.immunichain.Childform')
	  .then(function(result) {
		return result.update(child);
        });
	}

See picture below to get a sense of what to do.

.. image:: images/composer/3.9.1.png

**10.** Again, click on Deploy Changes to update your Script File.

Part 4: Submitting Transactions
===============================

**1.** Now that we have a new participant, let's create a Member. Jump to the Test section and **click on Member on the left.** 

.. image:: images/composer/4.1.png

**2.** **Click on +Create New Participant**, found in the top right, and follow the steps below to add a Member. This shows you how easy it is to update your business network within Hyperledger Composer. Being able to add new participantand asset types are relatively easy within Composer.

.. image:: images/composer/4.2.png

**3.** Then click on the pencil in the top right of our child's box. The pencil means to update the information with our child or, really, any participant type in the network.

.. image:: images/composer/4.3.png

**4.** **Click on Optional Properties first.** You will notice the member section appearing now. Then click on Update. We had to do this step in order allow information to pass through, inbetween the array brackets.

.. image:: images/composer/4.4.png

**5.** Now, click on Submit Transaction and let's authorize a member to view the health record of our child. You can change the type of transaction you want by click on the middle grey box. I have it in a square below. All the transaction types are defined in our script.js file located in the define section.

.. image:: images/composer/4.5.png

**6.** Now, let's make an authorized member transaction. Here is my transaction. You can make any type of transaction you want here to accurately represent the correct children and member you desire.

.. image:: images/composer/4.6.png

My transaction says let member #1 (Fairmont High School) have Child #1's (Emily) health record. This would be extremely useful when every year thousands of kids get physicals in order to play a sport. Imagine having your medical provider authorize your child's health record to approve them playing a sport. I know my mom would've enjoyed not going up to the High School an additional time. 

**7.** You can view this transaction by clicking on Childform on the left and then Show All on Emily or whatever name you gave your child. Notice that member 1 is now in Emily's description.

.. image:: images/composer/4.7.png

**8.** Click on Submit Transaction in the bottom left.

**9.** A pop-up will appear. Change the transaction type to assignMedProvider to one of our medical providers to one of the children you've created.

**10.** Now, replace the ID Numbers to represent the guardian, medical provider and child you have within your network. Look at the below picture to get a sense of what to do

.. image:: images/composer/4.8.png

That basically says, assign medical provider #1 (Healthquest) to Child #1 (Emily).

**11.** Click Submit once you have the ID Numbers you want

**12.** Once you submit the transaction and it is good, click on All Transactions in the bottom left. This is what Composer likes to call the Historian. Now is a good time to tell you about this feature. The Historian is the sequence of transactions or addition or removal of participants or assets. I didn't tell you to look at the Historian when you were creating the Participants and Assets, but the Historian kept track of when and what type of participant or asset you created. You can scroll to the bottom to view the first transaction you created. You can see by clicking on view record. 

.. image:: images/composer/4.9.png

**13.** Back to our transaction, click on the Childform on the left. Find the child you assigned a Medical Provider to. Click on Show All to view the entire asset of your child. Notice the medical provider you assigned it to? 

.. image:: images/composer/4.10.png

**14.** Should we do another transaction? Of course! 

**15.** We have submitted some transactions, but now let's actually add some immunizations to a child

**16.** Click on Submit Transaction and then change the transaction type to addImmunizations. The format to add an immunization is a little different. In the Vaccine section put { "name" : "immunization", "provider" : "medical provider", "imdate" : "date" } inbetween the brackets. Replace the immunization, medical provider and date with whatever you would like. Here is what my transaction looks like::

	{ "name" : "immunization", "provider" : "medical provider", "imdate" : "date" }

.. image:: images/composer/4.11.png

**17.** To view your immunization, go your child in the Childform section

.. image:: images/composer/4.12.png

**18.** Continue to make various transactions that you want

Part 5: Modifying Permissions
=============================

If you were to go to the permissions.acl file in the Define section, you would notice how any participant can do anything that they want to the network. This doesn't actually replicate what would happen in a real Immunichain business network. In this section we are going to change the permissions to the business network. You will notice these permissions by submitting transactions with the various participant identities you are about to create. 

**1.** Go to the Define section of Composer Playground. Then click on admin in the top right. Then click on ID Registry. The ID registry is a place to create new IDs associated with our business network. For example I can create an identity for Austin as the guardian in our network.

.. image:: images/composer/5.1.png

**2.** We are doing great if this is what your page looks like. Don't be alarm by the two different sections. The only difference between the two sections is the status column. If you hover your cursor on the far right you will see options. One of them is to revoke an identity. Revoke simply means that you will not be able to connect to that perspective. 

.. image:: images/composer/5.2.png

**3.** Click on Issue New ID

**4.** A pop-up will appear. Give your identity a name (disclaimer: the identity will be tied to a participant you created earlier in the lab; ie: Guardian: Austin, Medical Provider: HealthQuest). Then type in the number 1. You should now see the various participants that have an ID number of 1. If you gave your participants a different ID number, you won't see anything by typing in 1. Instead, type in the number you gave to your participants. Also, if you have multiple participants with the same ID number, there will be multiple options based on the ID number. Click on the participant that you trying to create. Here is what I did below:

.. image:: images/composer/5.3.png

**5.** If your screen looks like this, then we are in good shape

.. image:: images/composer/5.4.png

**6.** Go ahead and create other identities for your participants

**7.** I have a total of 4 identities in my business network, 3 different participants and then the admin card. Here is what my screen looks like. You could have more identities if you created more participants your created in Part 2

.. image:: images/composer/5.5.png

**8.** Since we are in the admin identity (make sure you see admin in the top right), lets change our permissions file. Click on Define and then Access Control in the bottom left.

.. image:: images/composer/5.6.1.png

**9.** You will notice a few rules there already. These rules are required for the Admin identity to access the entire network. It is important that you leave those rules there. Now, you are going to add a few rules to our network. Copy these rules below::

	rule UpdatePersonal {
      	description: "Allow the guardian update the child's personal info"
        participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource(c): "ibm.wsc.immunichain.Childform"
      	transaction(tx): "ibm.wsc.immunichain.updateChildForm"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule txUpdatePersonal {
      	description: "Allow the guardian to update the child assets"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.updateChildForm"
      	action: ALLOW
	}

	rule AssignProvider {
      	description: "Allow the guardian to assign and update medical providers"
      	participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: UPDATE
      	resource(c): "ibm.wsc.immunichain.Childform"
      	transaction(tx): "ibm.wsc.immunichain.assignMedProvider"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule txAssignProvider {
      	description: "Allow the guardian to assign and update medical providers"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.assignMedProvider"
      	action: ALLOW
	}

	rule AuthMembers {
      	description: "Allow the guardian to authorize member organizations"
      	participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: UPDATE
      	resource(c): "ibm.wsc.immunichain.Childform"
      	transaction(tx): "ibm.wsc.immunichain.authMember"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule txGuardianAuthMembers {
      	description: "Allow the guardian to authorize member organizations"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.authMember"
      	action: ALLOW
	}

	rule txMedProviderAuthMembers {
      	description: "Allow the guardian to authorize member organizations"
      	participant: "ibm.wsc.immunichain.MedProvider"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.authMember"
  	transaction: "ibm.wsc.immunichain.authMember"
      	action: ALLOW
	}

	rule DeauthMembers {
      	description: "Allow the guardian to deauthorize member organizations"
      	participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: UPDATE
      	resource(c): "ibm.wsc.immunichain.Childform"
      	transaction(tx): "ibm.wsc.immunichain.removeMemberAuth"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule txDeauthMembers {
      	description: "Allow the guardian to deauthorize member organizations"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.removeMemberAuth"
      	action: ALLOW
	}

	rule Reassign {
      	description: "Allow the guardian to reassign their children (if of age)"
      	participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: UPDATE
      	resource(c): "ibm.wsc.immunichain.Childform"
      	transaction(tx): "ibm.wsc.immunichain.reassignGuardian"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule txReassign {
      	description: "Allow the guardian to reassign their children (if of age)"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: ALL
      	resource: "ibm.wsc.immunichain.reassignGuardian"
      	action: ALLOW
	}

	rule GuardianRead {
      	description: "Allow guardians to view their own child's health record"
      	participant(g): "ibm.wsc.immunichain.Guardian"
      	operation: READ, UPDATE
      	resource(c): "ibm.wsc.immunichain.Childform"
      	condition: (c.guardian.getIdentifier() == g.getIdentifier())
      	action: ALLOW
	}

	rule readMembers {
      	description: "Allow guardians to view their own child's health record"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: READ
      	resource: "ibm.wsc.immunichain.Member"
      	action: ALLOW
	}

	rule readMedicalProviders {
    	description: "Allow the guardian to create medical providers in the network"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: READ
      	resource: "ibm.wsc.immunichain.MedProvider"
      	action: ALLOW
	}

	rule addChild {
      	description: "Allow the Medical Provider to add a child in the network"
      	participant: "ibm.wsc.immunichain.MedProvider"
      	operation: CREATE
      	resource: "ibm.wsc.immunichain.Childform"
      	action: ALLOW
	}

	rule CreateChild {
      	description: "Allow the Guardian to add a child in the network"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: CREATE
      	resource: "ibm.wsc.immunichain.Childform"
      	action: ALLOW
	}

	rule MedicalProviderRead {
    	description: "Allow members to view children that have them as a member"
      	participant(g): "ibm.wsc.immunichain.MedProvider"
      	operation: UPDATE, READ
      	resource(c): "ibm.wsc.immunichain.Childform"
      	condition: (c.medproviders.some(function(MedProvider) {
      	return MedProvider.getIdentifier() == g.getIdentifier();
      	}))
      	action: ALLOW
	}

	rule medRead1 {
    	description: "Allow the Medical Providers to read all the members available in the network"
      	participant: "ibm.wsc.immunichain.MedProvider"
      	operation: READ
      	resource: "ibm.wsc.immunichain.Member"
      	action: ALLOW
	}

	rule medRead2 {
    	description: "Allow the Medical provider to view all the guardian's in the network"
      	participant: "ibm.wsc.immunichain.MedProvider"
      	operation: READ
      	resource: "ibm.wsc.immunichain.Guardian"
      	action: ALLOW
	}

	rule MemRead {
      	description: "Allow the Members to view all the Children in the network"
      	participant: "ibm.wsc.immunichain.Member"
      	operation: READ
      	resource: "ibm.wsc.immunichain.Childform"
      	action: ALLOW
	}

	rule medUser {
    	description: "Allow the Medical provider to view all the guardian's in the network"
      	participant: "ibm.wsc.immunichain.MedProvider"
      	operation: READ, CREATE
      	resource: "org.hyperledger.composer.system.*"
      	action: ALLOW
	}

	rule memberUser {
    	description: "Allow the Medical provider to view all the guardian's in the network"
      	participant: "ibm.wsc.immunichain.Member"
      	operation: READ
      	resource: "org.hyperledger.composer.system.*"
      	action: ALLOW
	}

	rule GuardanUser {
    	description: "Allow the Medical provider to view all the guardian's in the network"
      	participant: "ibm.wsc.immunichain.Guardian"
      	operation: READ, CREATE
      	resource: "org.hyperledger.composer.system.*"
      	action: ALLOW
	}


**10.** **Then paste these rules above the other rules** in the Access Control file. Here is what I my screen looks like now. The order of the ACL rules is important. The first rule determines if the participants can proceed to the next rule:

.. image:: images/composer/5.8.1.png

**11.** Once you are good to go, click on Deploy Changes in the bottom left and that will make changes across the entire business network. Read through some of the rules that we just implemented. What do you think will change as we go through the various identities?

.. image:: images/composer/5.9.1.png

**12.** Click on admin in the top right again. This time, click on My Business Networks. This will take us to the Composer Playground homepage

**13.** Now your screen should look like this:

.. image:: images/composer/5.10.png

When you created the identities, Composer was creating ID Cards for those identities. That is why I have 4 ID Cards. They are all tied to the Immunichain business network and to the participants you created in Part 2. You could think of this as a 4 peer Blockchain network, with 1 of the peers being an admin who oversees the entire network. 

**14.** Go ahead and click on Connect Now with your Guardian ID.

.. image:: images/composer/5.11.png

**15.** You are now in the Guardian's perspective in the Immunichain business network. Go ahead and click on the other participants in the Test section

Medical Providers:

.. image:: images/composer/5.12.png

Members: 

.. image:: images/composer/5.13.png

Child: 

.. image:: images/composer/5.14.png

What did you notice about the permissions here? From the Guardian perspective, you can view all the Medical Providers, Members and Children that the Guardian has ownership of. 

**16.** Go ahead and update your Child by clicking on the pencil in the top right. Delete the Medical Providers and Members. **When I say delete all the members, I mean to delete the contents within the brackets - []. So leave the [] in the member section.** We are doing this so that we can submit transactions from various perspectives. This will test our new access control rules.

.. image:: images/composer/5.15.png

.. image:: images/composer/5.16.png

**17.** Submit transaction from the Guardian perspective. Start with assigning a Medical Provider. 

.. image:: images/composer/5.17.png

**18.** Submit another transaction by assigning a Member

.. image:: images/composer/5.18.png

From the Guardian perspective, you are able to do a lot of different things. First, you can view the Children in the network that the Guardian has ownership of. Also, the guardian can create additional children with the way the permissions are set up. Do you think this is a viable option in a production environment? I would say no, but you can have the Medical Provider, who administered the birth of the Child, create the Child asset. In a production environment, this would be negotiated between all the participants in the business network. Also, as the Guardian you can also view all the Members and Medical Providers. Why do you think that is so? When you have a child as a guardian you want to be able to view all the options you have as possible Medical Providers and Members. In a real-world scenario, maybe the Guardian would only view and allow all the Medical Providers that are tied to their Health Insurance, but that would require an Insurer in this Immunichain business network. Maybe in the future :) 

**19.** I think you're getting the sense from the Guardian perspective. Before we jump to another perspective, **delete all Members. When I say delete all the members, I mean to delete the contents within the brackets - []. So leave the [] in the member section** You previously did this from step 16 in this part. Once you have successfully done that, go ahead and switch to the Medical Provider perspective. Click on My Business Networks in the top right. Then click on Connect Now on the Medical Provider

.. image:: images/composer/5.19.png

**20.** Click around on the other participants in the Immunichain Business Network

Guardian: 

.. image:: images/composer/5.20.png

Members:

.. image:: images/composer/5.21.png

Child: 

.. image:: images/composer/nomember.png

**21.** Click on Submit Transaction. Start with assigning a Member

.. image:: images/composer/5.23.png

**22.** Now, create another Child asset. Have the Child's guardian be the first Guardian. In my business network, this would be Guardian Austin. 

.. image:: images/composer/5.24.png

Now, you won't notice the kid show up from the medical perspective, but I now have TWINS! My life suddenly got crazy for a 23-year-old. I guess I need to continue work in order to support them. Or just become a crypto-currency millionaire (I don't know if that's possible these days). 

On a slightly more serious note, maybe having the Medical Provider create additional children isn't the best idea. It really depends on who the Medical Provider is. Is it the hospital? Or more specifically, is the Medical Provider the doctor who works in the baby delivery department of the hospital? Should the Medical Provider be able to create the child, or should we leave it up to the Guardians to create the children? These types of conversations have to occur between the peers in the business network if this was to be a production environment. 

**23.** Great, we just created another Child. Jump back over to the Guardian perspective. Did the new Child show up? 

.. image:: images/composer/5.26.png

**24.** Go ahead and only assign a Medical Provider to the new Child by submitting a transaction 

**25.** Should we jump to the Member perspective? Absolutely! 

.. image:: images/composer/5.27.png

**26.** Look around at the various participants in the Immunichain business network

Child: 

.. image:: images/composer/5.28.png

**27.** If you noticed, all the children showed up. Click on Show All on the Bobbie, you notice that this member isn't listed as one her authorized Members.

.. image:: images/composer/5.29.png

Is this a good thing - that Bobbie appeared to this member? Absolutely not. This would be a non-negotiable in the business network. You wouldn't want a Member to be able to see a Child, unless it has authorization. Could you imagine a Member being able to read all the Immunization records of every Child? We have to modify the permissions in our Access Control file. 

See if you can modify the rule in the Access Control file in the Define section. 

**End of Lab!**
