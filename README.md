Download Link: https://assignmentchef.com/product/solved-cs6500-network-securityassignment-3
<br>
The objective of this assignment is to implement the KDC-based key establishment (exchange) for use with symmetric key encryption algorithms.

There will be two programs: (a) Key Distribution Center (KDC), and (b) Client (C). At start of execution, open at least 3 Terminal windows, one each for the KDC and two clients. In each terminal, invoke the corresponding program as described later. All communications between the KDC and the clients will be via TCP sockets.

The KDC’s activities include: (i) storing clients’ master keys in a file (that uses encryption/hash such that the master keys are not publicly readable); (ii) generating a symmetric key for use between two clients (A and B) for secure data communication; either A or B can request the secret key for their communication. The data encryption algorithm to be used for client-client and KDC-client communication is: AES-128CBC, i.e. 128-bit key with CBC mode.

<h1>1       KDC</h1>

The KDC is invoked as follows:

./kdc -p portid -o outfilename -f pwdfile

The kdc process will listen on port number (<em>portid</em>) specified in the command line; the diagnostic output messages (that describe kdc’s activities) will be stored in the file named, <em>outfilename</em>; the client’s details will be stored in the password file (<em>pwdfile</em>), specified in the command line.

For example:

./kdc -p 12345 -o out.txt -f passwd.txt

The KDC’s activities are:

<ol>

 <li>A message from a client is received with the following format:</li>

</ol>

| 301| IPAddress| ClientPortNum| ClientMasterKey| ClientName|

In the above message:

<ul>

 <li>the first field (“301”) is of type integer; Here, the code 301 indicates that this is a registration message from the client to the KDC.</li>

 <li>the second field is an IPv4 address, stored as a dotted string of length 16 bytes (e.g. “10.4.5.11”);</li>

 <li>the third field is a TCP port number, stored as a string of length 8 bytes (e.g. “35678”);</li>

 <li>the fourth field is the passphrase, a string of length 12 bytes; This string is converted into a 128-bit key and stored in an encrypted manner (with base64 encoding) in the password file maintained by the KDC.</li>

 <li>the last field is the client’s name, a string of length 12 bytes (only lowercase letters from ’a’ – ’z’ are permitted), e.g. “alice”.</li>

</ul>

The KDC will store the client’s details in the specified password file, as shown in the following example:

:alice:10.4.5.11:35678:ABCDEFabcdef123456789=: :babu:10.4.6.13:45561:abcdef12345ABCDEF6789=:

If the username specified above already exists, the KDC overwrites the password file with the latest information.

Upon successful retrieval of the message and storage in the password file, the KDC will send a message to the client in the following format:

| 302| ClientName|

When the client receives this message with code “302”, it processes the message but takes no further action.

<ol start="2">

 <li>A message from a client is received with the following format:</li>

</ol>

| 305| E_KA[ IDA|| IDB|| Nonce1] | IDA |

This message is sent from client (IDA) to KDC, requesting a secret key for communication with client (IDB). Assume that an entry for IDB is present in the password file. The KDC sends a response as follows:

| 306| E_KA[ Ks || IDA || IDB || Nonce1 || IPAddrB || PortNoB || E_KB[ Ks || IDA || IDB || Nonce1 || IPAddrA || PortNoA] ] |

The KDC generates <em>Ks </em>as a 8-byte random string from the character set,.

The message also includes Client B’s IP Address and Port Number that it is listening to.

The client receives this message and retrieves the key, <em>Ks</em>, for communication with client IDB, as explained in the class, by decrypting uses its shared master key with the KDC. It then sends a message to the specified IP address and Port number of Client B.

| 309| E_KB[ Ks || IDA || IDB || Nonce1 || IPAddrA || PortNoA] || IDA |

<h1>2       Client</h1>

A client will act either as a sender or a receiver. If it is a sender, the client program is invoked as follows:

./client -n myname -m S -o othername -i inputfile -a kdcip -p kdcport

Here, <em>myname </em>denotes the client’s (sender’s) name; <em>othername </em>denotes the receiver’s name; <em>inputfile </em>contains the contents that have to be encrypted and sent to the receiver; <em>kdcip </em>indicates the KDC’s IP address; <em>kdcport </em>indicates the KDC’s port number.

If it is a receiver, the client program is invoked as follows:

./client -n myname -m R -s outenc -o outfile -a kdcip                               -p kdcport

Here, <em>myname </em>denotes the client’s (receiver’s) name; <em>outenc </em>contains the contents received by the receiver; <em>outfile </em>contains the decrypted contents and received by the receiver; <em>kdcip </em>indicates the KDC’s IP address; <em>kdcport </em>indicates the KDC’s port number. The client’s activities are:

<ul>

 <li>Initiate the registration with the KDC, as explained above.</li>

 <li>After completion of initialization step, the client will sleep for 15 seconds, to ensure that all other clients are registered.</li>

 <li>If the client is a sender, it will initiate key request protocol with the server; once it obtains the secret key (Ks), it will follow the protocol described in class for client-to-client communication and encrypted data transfer.</li>

 <li>If the client is a receiver, it will wait to receive sender’s messages on its TCP port. The encrypted data file sent by the sender will be received and stored in the receiver’s output file (specified in the command line).</li>

 <li>The specific message formats are to be designed by you.</li>

</ul>

<h1>3       Sample Run</h1>

3.1      KDC Terminal Window

Assume that the KDC is running on a system with IP address of 10.21.22.23.

<table width="620">

 <tbody>

  <tr>

   <td width="620">Prompt&gt; script kdcscriptSystem outputs: Script started, output file is kdcscriptPrompt&gt; ./kdc -p 12345 -o out.txt -f pwd.txt …Starts TPC server to listen on port 12345 Wait for messages from clients..Type Ctrl-C to finally quitPrompt&gt; exitSystem outputs: Script done, output file is kdcscriptPrompt&gt; cat kdcscript</td>

  </tr>

 </tbody>

</table>

3.2      Sender Client Terminal Window

<table width="620">

 <tbody>

  <tr>

   <td width="620">P&gt; script sendscript …P&gt; ./client -n alice -m S -o bob -i in.txt -a 10.21.22.23 -p 12345…Contacts KDC and registers..Sleeps for 15 seconds ..Contacts KDC for secret key ….Quits after sending the file to bob.P&gt; exitSystem outputs: Script done, output file is sendscriptP&gt; cat sendscript</td>

  </tr>

 </tbody>

</table>

3.3      Receiver Client Terminal Window

<table width="620">

 <tbody>

  <tr>

   <td width="620">Prompt&gt; script recvscript …Prompt&gt; ./client -n bob -m R -s outenc.txt -o out.txt -a 10.21.22.23 -p 12345 …Contacts KDC and registers..Sleeps for 15 seconds ..Contacts KDC for secret key ….Quits after recving the file from alice.Prompt&gt; exitSystem outputs: Script done, output file is recvscriptPrompt&gt; cat recvscript</td>

  </tr>

 </tbody>

</table>

<h1>4       What to Submit on IITM Moodle</h1>

A single tar.gz file with name ROLLNO-Lab3.tgz

<ul>

 <li>Source files and Makefile</li>

 <li>A README File that explains how to compile and run the programs; whether your programs works correctly or whether there are any known bugs/errors in your program.</li>

 <li>Using the “script” Linux command, record the session from the Terminal window for KDC, sender and receiver; thus, there will be three typescript files, named as above.</li>

</ul>

Your system may be tested with the KDC, Sender and Receiver running on different machines (or) virtual machines on a given system.


