# Web Sensing: Data Diode Demonstrators

## Table of Contents:

- [Overview](#overview)
- [Access](#access)
- [Demo Instructions](#demo-instructions)
  - [ICMP Testing](#icmp-testing)
  - [UDP Testing](#udp-testing)
  - [TCP Testing](#tcp-testing)
  - [Advanced Options](#advanced-options)
  - [Command Summary](#command-summary)
- [Creating SSH Keys](#ssh-keys)

# Overview

A tutorial video is available [here](https://youtu.be/GOu_UgQR9Uw) -- be sure to use 1080p video quality via the YouTube gear icon.

We provide three online demonstrators: *demo02*, *demo03*, and *demo04*. Each system is configured as pictured below with two computers -- ***in*** and ***out*** -- connected via a Data Diode. The diode restricts traffic such that *only* UDP traffic can pass, one-way, from the protected *inside* asset (***in***) to the *outside* networked computer (***out***); all other traffic is dropped.

![demo_500x](https://user-images.githubusercontent.com/106708748/178362591-9d99c03e-83b1-40e2-971d-03a5b017388b.jpeg)


# Access

To access a demonstrator you will need to send us your *public-key*. We install it for you, allowing you to remotely access a designated demonstrator using a ***demo*** user account. If you do not have a public-key, please see the instructions at the end of this Wiki.

The ***in*** and ***out*** computers on each demonstrator are accessed using Secure Shell (SSH) to IP address **71.173.69.76** with the following ports: 

- demo02-in: **2222** 
- demo02-out: **2223** 

***
- demo03-in: **2224** 
- demo03-out: **2225** 

***
- demo04-in: **2226** 
- demo04-out: **2227** 

For example, to access **demo02-in** from a Linux or MacOS machine, use SSH in a Terminal window with the command:

`ssh demo@71.173.69.76 -p 2222`

On Windows, use the [Putty SSH client](https://devops.ionos.com/tutorials/use-ssh-keys-with-putty-on-windows/) and enter the ***demo*** user name, IP address, and port number in the connection dialog box.


# Demo Instructions

You may attempt to pass traffic across the diode in any manner you wish. We provide client/server programs (*wsc*/*wss*)to simplify this task. 

For reference, note that the diode is connected at IP address **172.16.253.1** on the ***in*** computers, and **172.16.253.2** on the ***out*** computers.


### ICMP Testing

On ***in***, use the *ping* command to attempt contact with ***out***:

`ping -w 3 172.16.253.2`

Conversely, on ***out*** use *ping* to attempt contact with ***in***:

`ping -w 3 172.16.253.1`

Both these commands will fail and timeout after 3 seconds: ICMP traffic is not allowed across the diode.


### UDP Testing

On ***out***, start a UDP *server* (*wss*) to receive any traffic that arrives over the diode:

`wss -v`

On ***in***, use the UDP *client* (*wsc*) to send a 64 Kilobyte file, containing random data, across the diode to ***out***:

`wsc -f testfiles/64kb.test -ip 172.16.253.2 -v`

This test will successfully transmit the file; the UDP server on ***out*** will print a summary of each message transmitted and recreate the file in *testifies/64kb.test*. Note that the *wsc* software adds extra messages for coordination when sending files. 

You may also verify that the file is correctly transmitted by computing a hash of the data on both the ***in*** and ***out*** computers using the command:

`sha256sum testfiles/64kb.test`

Conversely, run the *server* on ***in*** (`wss -v`) and the *client* on ***out*** using:

`wsc -f testfiles/64kb.test -ip 172.16.253.1 -v`

This attempts to send UDP data in the wrong direction over the diode; You will observe the client sending data, but no traffic will arrive at the server.

You may terminate the *server* using Control-c (hold down the control key, and press c). 

### TCP Testing

On ***out***, start a TCP *server* (*wss -m tcp*) to receive any traffic that arrives over the diode:

`wss -m tcp -v`

On ***in***, use the TCP *client* (*wsc -m tcp*) to send a 64 Kilobyte file, over the diode:

`wsc -m tcp -f testfiles/64kb.test -ip 172.16.253.2 -v`

Notice that no data is transferred: TCP is not allowed over the diode. TCP will continue to retrying the transfer and eventually timeout after approximately two minute; As before, you may terminate the server with Control-c.

Conversely, run the TCP *server* on ***in*** (`wss -m tcp -v`) and the *client* on ***out*** using:

`wsc -m tcp -f testfiles/64kb.test -ip 172.16.253.1 -v`

This command attempts to send TCP data in the wrong direction over the diode. You will again observe the *client* sending data, but no traffic arriving at the *server* : TCP is not allowed, in either direction, over the diode.



### Advanced Options

Each of the client/server utilities (*wsc*/*wss*) has a help menu with usage information. This can be accessed with the `-h` flag, for example:

`wsc -h`


### Command Summary

In the following summary, left-justified commands are executed on ***in*** and tabbed commands are executed on ***out***.

```
ssh demo@71.173.69.76 -p 2222     -- connect to in
ssh demo@71.173.69.76 -p 2223     -- connect to out

ping -w 3 172.16.253.2
    ping -w 3 172.16.253.1

    wss -v
wsc -f testfiles/64kb.test -ip 172.16.253.2 -v
sha256sum testfiles/64kb.test
    sha256sum testfiles/64kb.test

wss -v
    wsc -f testfiles/64kb.test -ip 172.16.253.1 -v

    wss -m tcp -v
wsc -m tcp -f testfiles/64kb.test -ip 172.16.253.2 -v

wss -m tcp -v
    wsc -m tcp -f testfiles/64kb.test -ip 172.16.253.1 -v
```




# Creating SSH Keys

If you already have an ssh key-pair you would like to use, send us your public key. If not, you will need to generate one. On Linux and MacOS this can most easily be performed by running:

`ssh-keygen`

in a Terminal window and typing *Enter* at each prompt. This command will print the key-pair's location, usually `~/.ssh`. The file named `id_rsa.pub` contains the *public* key to send to us. 

On Windows, follow the [Putty guide](https://devops.ionos.com/tutorials/use-ssh-keys-with-putty-on-windows/) to generate a key-pair and use it with the PuTTY tool. In the "Copy Public Key To Server" step, send us the *public* key which starts with *ssh-rsa*.


