# Box name - Cap
## Box IP - 10.129.36.179

To begin, I first needed to connect to the HTB lab machines which I did by installing OpenVPN and downloading the HTB OVPN profile. With this connection I could finally interact the target IP which I proved with a quick ping 

~~~
ping 10.129.36.179

Response:
PING 10.129.36.179 (10.129.36.179) 56(84) bytes of data.
64 bytes from 10.129.36.179: icmp_seq=1 ttl=62 time=92.3 ms
64 bytes from 10.129.36.179: icmp_seq=2 ttl=62 time=120 ms
64 bytes from 10.129.36.179: icmp_seq=3 ttl=62 time=59.4 ms
^C
--- 10.129.36.179 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 59.408/90.436/119.590/24.604 ms
~~~

At this point I ran three Nmap commands to learn a little more about the machine itself and its open services.

Service version scan:
~~~
nmap -sV 10.129.36.179

Response: 
~~~

TCP SYN port scan:
~~~
nmap -sS 10.129.36.179

Response: 
~~~

Operating system estimation:
~~~
nmap -O 10.129.36.179

Response: 
~~~

