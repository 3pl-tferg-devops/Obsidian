Reduce EDNS reassembly buffer size. 

IP fragmentation is unreliable on the Internet today, and can cause
transmission failures when large DNS messages are sent via UDP. 

Even when fragmentation does work, it may not be secure; it is theoretically
possible to spoof parts of a fragmented DNS message, without easy
detection at the receiving end. 

Recently, there was an excellent study [Defragmenting DNS](https://indico.dns-oarc.net/event/36/contributions/776/) - Determining the optimal
maximum UDP response size for DNS by Axel Koolhaas, and Tjeerd Slokker in collaboration with NLnet Labs.

They explored DNS using real world data from the the RIPE Atlas probes and the researchers suggested different values for IPv4 and IPv6 and in different scenarios. 

They advise that servers should be configured to limit DNS messages sent over UDP to a size that will not trigger fragmentation on typical network links. DNS servers can switch from UDP to TCP when a DNS response is too big to fit in this limited buffer size. 

This value has also been suggested in DNS Flag Day 2020.