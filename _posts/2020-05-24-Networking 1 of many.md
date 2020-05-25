---
layout: post
title:  "Networking 1 of many"
date:   2020-05-24 18:16:10 -0700
categories: stuff
---

# Handshake, Push and Closing of TCP connection

I am using RPi and my trusty Mac. RPi and Mac are both connected to Wifi to enable my interaction with RPi over SSH and RPi and Mac are connected via Ethernet for me to do my experiments on.

For this you can have 2 vms or do everything on same machine.

### Start Server on RPi

`nc -kl $ETH0_IP$ 234`

This sets up a server listening on port 234 bound to ip on eth0 interface. You can figure value of $ETH0_IP$ from ifconfig and getting value of inet (ipv4) addr for eth0. In my case $ETH0_IP$ is 192.168.3.3. Mac IP is 192.168.3.1

### Start monitor on RPi

For this post I am only interested in TCP so lets start tcpdump on eth0 filtering only tcp traffic. I am running this on RPi.

`sudo tcpdump tcp -i eth0`


### TCP Handshake

From Mac try to do TCP handshake with process listening on ip $ETH0_IP$ and port 234:

`nc -v $ETH0_IP$ 234`

Tcpdump will show you then handshake of SYN, SYN-ACK, ACK:

```
21:18:40.964994 IP 192.168.3.1.56402 > 192.168.3.3.234: Flags [S], seq 2467894129, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 874609633 ecr 0,sackOK,eol], length 0
21:18:40.966085 IP 192.168.3.3.234 > 192.168.3.1.56402: Flags [S.], seq 3368342792, ack 2467894130, win 65160, options [mss 1460,sackOK,TS val 1812589640 ecr 874609633,nop,wscale 7], length 0
21:18:40.966245 IP 192.168.3.1.56402 > 192.168.3.3.234: Flags [.], ack 1, win 2058, options [nop,nop,TS val 874609634 ecr 1812589640], length 0
```

Output format of tcpdump is well described in: https://www.tcpdump.org/manpages/tcpdump.1.html

Anyway, let's go line by line:
`21:18:40.964994 IP 192.168.3.1.56402 > 192.168.3.3.234: Flags [S], seq 2467894129, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 874609633 ecr 0,sackOK,eol], length 0`

1st column time is time as seen by localhost, RPi in this case.
2nd column says the protocol, IP. For ARP, you'll see ARP etc.
3rd column is $PACKETS_SOURCE_IP$.$PACKETS_SOURCE_PORT$
5th column is $PACKETS_DESTINATION$.$PACKETS_DESTINATION_PORT$
6th and 7th column shows flags, in this case it's SYN, represented by S. Other flags can beS (SYN), F (FIN), P (PUSH), R (RST), U (URG), W (ECN CWR), E (ECN-Echo) or . (ACK), or none if no flags are set.
8th and 9th column is seq of packet in case of SYN and SYN-ACK. One handshake is done it appears that seq number resets to 1 (see the last ACK in handshake).
10th and 11th column show # of bytes left in buffer space in other direction of the connection.
12th is TCP options, go into details of that later.
Last and 2nd last column is length of payload. Which is 0 in this case since this is just a handshake.

### TCP connection failure since no one is listening on target port

```
15:43:42.097836 IP 192.168.3.1.56733 > 192.168.3.3.234: Flags [S], seq 1897475769, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 876730690 ecr 0,sackOK,eol], length 0
15:43:42.098903 IP 192.168.3.3.234 > 192.168.3.1.56733: Flags [R.], seq 0, ack 1897475770, win 0, length 0
```

Client sends SYN but TCP stack on other ends does not know of any procecss listeing at port 234 so it sends back RST-ACK flags

### TCP sending data

In TCP any side can choose to send data to other side.

After handshake the data push looks like so:
```
15:46:26.249518 IP 192.168.3.3.234 > 192.168.3.1.56743: Flags [P.], seq 1:2, ack 1, win 510, options [nop,nop,TS val 1879056293 ecr 876889517], length 1
15:46:26.249769 IP 192.168.3.1.56743 > 192.168.3.3.234: Flags [.], ack 2, win 2058, options [nop,nop,TS val 876894656 ecr 1879056293], length 0
```

Note the flag PUSH-ACK for push, things to note here are length is 1B and seq number is `1:2`. Also note that ACK is `1`.

Lets send a string this time `asdfasdfasdfasdfasdf`
```
15:51:31.305514 IP 192.168.3.1.56743 > 192.168.3.3.234: Flags [P.], seq 1:22, ack 2, win 2058, options [nop,nop,TS val 877199438 ecr 1879056293], length 21
15:51:31.306593 IP 192.168.3.3.234 > 192.168.3.1.56743: Flags [.], ack 22, win 510, options [nop,nop,TS val 1879361357 ecr 877199438], length 0
```

For PUSH-ACK, note the length is 21B and seq is 1:22. And for ACK that was sent back is 22.

### Server Ending connection

Either side may choose to close the connection

```
15:55:13.856162 IP 192.168.3.3.234 > 192.168.3.1.56743: Flags [F.], seq 2, ack 22, win 510, options [nop,nop,TS val 1879583911 ecr 877199438], length 0
15:55:13.856518 IP 192.168.3.1.56743 > 192.168.3.3.234: Flags [.], ack 3, win 2058, options [nop,nop,TS val 877421781 ecr 1879583911], length 0
15:55:13.856626 IP 192.168.3.1.56743 > 192.168.3.3.234: Flags [F.], seq 22, ack 3, win 2058, options [nop,nop,TS val 877421781 ecr 1879583911], length 0
15:55:13.857724 IP 192.168.3.3.234 > 192.168.3.1.56743: Flags [.], ack 23, win 510, options [nop,nop,TS val 1879583913 ecr 877421781], length 0
```

In this case server on RPi is closing it. Note that when server sends FIN-ACK, not only the client responds with ACK, it also follows up by another FIN-ACK. So first server tells client, `I am closing connection` and client say `ack, me too`. Lastly, server say `ack`.

Same thing happens when client initiates closing of connection
```
16:01:32.083471 IP 192.168.3.1.56757 > 192.168.3.3.234: Flags [F.], seq 1, ack 1, win 2058, options [nop,nop,TS val 877799491 ecr 1879957977], length 0
16:01:32.084722 IP 192.168.3.3.234 > 192.168.3.1.56757: Flags [F.], seq 1, ack 2, win 510, options [nop,nop,TS val 1879962147 ecr 877799491], length 0
16:01:32.084930 IP 192.168.3.1.56757 > 192.168.3.3.234: Flags [.], ack 2, win 2058, options [nop,nop,TS val 877799492 ecr 1879962147], length 0
```