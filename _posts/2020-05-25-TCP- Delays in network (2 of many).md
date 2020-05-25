---
layout: post
title:  "TCP: simulating packet delay and loss (2 of many)"
date:   2020-05-24 18:16:10 -0700
categories: stuff
---

# Set up

Setup is same as usual:
I am using RPi and my trusty Mac. RPi and Mac are both connected to Wifi to enable my interaction with RPi over SSH and RPi and Mac are connected via Ethernet for me to do my experiments on. 

In this case server is on RPi's eth0 at 192.168.3.3:234 and client is on Mac at 192.168.3.1

For this you can have 2 vms or do everything on same machine.

# Simulating network delays/packet drops/etc

`netem`[1] provides the functionality to simulate network delays and packet drops.

# TCP Options

Before we look at the (more) fun part. Let's quickly go over 2 of TCP options that we will need to know to understand following better. These options are TS and ECR. I think I can do better job of explaining them than RFC7323[2]. Key take aways are:
1. TS is just a monotonically increasing number and does not related to time as we know it.
2. ECR must be TS of previously ack'd packet
3. There is no need to sync clock b/n parties.

# TCP network delay

Assuming TCP handshake is done. Run following on RPi:
`sudo tc qdisc add dev eth0 root netem delay 100000ms`

If you are running this cmd again you'll have to replace `add` with `change`.
This essentially adds delay of 100000ms on any packet going via eth0.

If client does a push now, following happens:
```
16:38:12.694052 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [P.], seq 3:4, ack 1, win 2058, options [nop,nop,TS val 879994763 ecr 1882152379], length 1
16:38:12.845229 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [P.], seq 3:4, ack 1, win 2058, options [nop,nop,TS val 879994913 ecr 1882152379], length 1
16:38:13.146437 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [P.], seq 3:4, ack 1, win 2058, options [nop,nop,TS val 879995213 ecr 1882152379], length 1
...
...
...
16:38:53.367023 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [P.], seq 3:4, ack 1, win 2058, options [nop,nop,TS val 880035415 ecr 1882152379], length 1
16:38:59.967306 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [P.], seq 3:4, ack 1, win 2058, options [nop,nop,TS val 880042015 ecr 1882152379], length 1
```

Note how ECR are same in each Push. This signifies that it's same attempt and not repeated packet.


As client retries, it backs off. If delay is long enough, client gives up and sends reset to server closing connection:
```
16:39:06.568693 IP 192.168.3.1.57073 > 192.168.3.3.234: Flags [R.], seq 4, ack 1, win 2058, length 0
```

Although RFC7323[2] says that Reset should have TS but I guess my implementation hasn't caught up to it yet.


But server eventually comes gets the packet and starts sending ack:
```
16:39:52.694141 IP 192.168.3.3.234 > 192.168.3.1.57073: Flags [.], ack 4, win 510, options [nop,nop,TS val 1882162927 ecr 879994763], length 0
16:39:52.845292 IP 192.168.3.3.234 > 192.168.3.1.57073: Flags [.], ack 4, win 510, options [nop,nop,TS val 1882163078 ecr 879994913,nop,nop,sack 1 {3:4}], length 0
16:39:53.146509 IP 192.168.3.3.234 > 192.168.3.1.57073: Flags [.], ack 4, win 510, options [nop,nop,TS val 1882163379 ecr 879995213,nop,nop,sack 1 {3:4}], length 0
...
...
...
16:40:13.563038 IP 192.168.3.3.234 > 192.168.3.1.57073: Flags [.], ack 4, win 510, options [nop,nop,TS val 1882183795 ecr 880015615,nop,nop,sack 1 {3:4}], length 0
16:40:20.164345 IP 192.168.3.3.234 > 192.168.3.1.57073: Flags [.], ack 4, win 510, options [nop,nop,TS val 1882190396 ecr 880022215,nop,nop,sack 1 {3:4}], length 0
```

As you can see that ECR values of ACKs matches TS values from PUSHes which signifies that these ACKs are for those corresponding PUSHes.

Let's reset the delay to 1ms for next experiment
`sudo tc qdisc change dev eth0 root netem delay 1ms`


# TCP packet loss

Assuming TCP handshake is done. Run following on RPi:
`sudo tc qdisc change dev eth0 root netem loss 50%`

As far as applications using this connection are concerned this appears as delay. Since the sending party retries and eventually data gets over to the other side.

Here is what it looks like:
```
16:50:50.715087 IP 192.168.3.1.57085 > 192.168.3.3.234: Flags [P.], seq 1415:1416, ack 1, win 2058, options [nop,nop,TS val 880752116 ecr 1882920725], length 1
16:50:50.866387 IP 192.168.3.1.57085 > 192.168.3.3.234: Flags [P.], seq 1415:1416, ack 1, win 2058, options [nop,nop,TS val 880752266 ecr 1882920725], length 1
16:50:51.168423 IP 192.168.3.1.57085 > 192.168.3.3.234: Flags [P.], seq 1415:1418, ack 1, win 2058, options [nop,nop,TS val 880752566 ecr 1882920725], length 3
...
...
16:51:04.977192 IP 192.168.3.1.57085 > 192.168.3.3.234: Flags [P.], seq 1415:1418, ack 1, win 2058, options [nop,nop,TS val 880766366 ecr 1882920725], length 3
16:51:04.977249 IP 192.168.3.3.234 > 192.168.3.1.57085: Flags [.], ack 1418, win 501, options [nop,nop,TS val 1882935205 ecr 880766366,nop,nop,sack 1 {1415:1418}], length 0
```

`192.168.3.1` repeatedly sends same data with backoff (ECR is constant) and eventually a packet reaches to `192.168.3.3` and `192.168.3.3` sends back an ACK (ECR is of TS of last PUSH). Keywork here is `an ACK`. This diffrentiates network delay with packet loss. In case of delay you'll see ACK for all packets with corresponding ECRs but in case of loss there is just 1 ACK with ECR equal to TS of packet received.

Let's reset the loss to 0% for next experiment:
`sudo tc qdisc change dev eth0 root netem loss 0%`

# TCP packet duplication

Let's set packet duplication to 100%
`sudo tc qdisc change dev eth0 root netem duplicate 100%`

Let's restart tcpdump with `-A` to see the payload and send data `All Opinions Rejected` from `192.168.3.3` to `192.168.3.1`
```
18:18:41.972164 IP 192.168.3.3.234 > 192.168.3.1.57453: Flags [P.], seq 4:26, ack 3, win 510, options [nop,nop,TS val 1888192257 ecr 885925249], length 22
E..J..@.@..............m.
.BL..=...........
p...4.%.All Opinions Rejected

18:18:41.972185 IP 192.168.3.3.234 > 192.168.3.1.57453: Flags [P.], seq 4:26, ack 3, win 510, options [nop,nop,TS val 1888192257 ecr 885925249], length 22
E..J..@.@..............m.
.BL..=...........
p...4.%.All Opinions Rejected

18:18:41.972702 IP 192.168.3.1.57453 > 192.168.3.3.234: Flags [.], ack 26, win 2058, options [nop,nop,TS val 886017581 ecr 1888192257], length 0
E..4....@..o.........m..L..=.
.X...
.......
4..-p...
18:18:41.972747 IP 192.168.3.1.57453 > 192.168.3.3.234: Flags [.], ack 26, win 2058, options [nop,nop,TS val 886017581 ecr 1888192257,nop,nop,sack 1 {4:26}], length 0
E..@....@..c.........m..L..=.
.X...
.	.....
4..-p......
.
.B.
.X
```


As you can see PUSH was duplicated since TS and ECR are same.
`192.168.3.1` sends back 2 ACKs but with same ECR which means that `192.168.3.1` knew that packets are duplicate. RFC7327[3] talks about it in `Protection Against Wrapped Sequences`. I'd like to capture packets from previous connection and replay them to see who TCP handles that. Looks like `tcpreplay`[4] is the tool for that.

[1] https://wiki.linuxfoundation.org/networking/netem
[2] https://www.rfcreader.com/#rfc7323_line435
[3] https://www.rfcreader.com/#rfc7323_line750
[4] https://tcpreplay.appneta.com/wiki/overview.html#overview