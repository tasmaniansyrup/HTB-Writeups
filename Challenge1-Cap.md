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

At this point I ran two Nmap commands to learn a little more about the machine itself and its open services.

TCP port scan:
~~~
nmap -sT 10.129.36.179

Response:
PORT    STATE    SERVICE
21/tcp  open     ftp
22/tcp  open     ssh
80/tcp  open     http
667/tcp filtered disclose
~~~

Service version scan:
~~~
nmap -sV 10.129.36.179

Response:
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    gunicorn
~~~

Immediately I saw the ftp service open on port 21 and attempted to log into it in the event it was accessible with anonymous credentials. Unfortunately, it was not.

With the knowledge of the open http service running on port 80 I navigated to 10.129.36.179:80 to see what I could find. I found a simple security dashboard with information about security events, failed login attempts, and port scans. I was also immediately logged into an account with the name "Nathan". There was also the ability to begin a 5 second security snapshot and provided a resulting PCAP file.

At this point, the lab guide challenged me to gain access to other user's scans which it hinted may be possible by playing around with the URL. I wanted to try a fuzzing tool to potentially uncover other directories that may allow me to navigate to other users. 

To do that, I decided to explore the ffuf tool which is lightweight and used for quick enumeration using a wordlist. For wordlists I found a popular repo called SecLists created by Daniel Miessler which contains many differents lists for different purposes which I figured would be nice to have in the future.

~~~
wget -c https://github.com/danielmiessler/SecLists/archive/master.zip -O SecList.zip && unzip SecList.zip && rm -f SecList.zip
~~~

Once running the scan the page navigates to http://10.129.36.179:80/data/1. My goal is to fuzz the url where the data directory is. 

Fuzz attempt number 1:

~~~
ffuf -w ./common.txt -u http://10.129.36.179:80/FUZZ

Response:
data                    [Status: 302, Size: 208, Words: 21, Lines: 4]
ip                      [Status: 200, Size: 17454, Words: 7275, Lines: 355]
netstat                 [Status: 200, Size: 32634, Words: 15751, Lines: 488]
~~~

These three results are actually just the locations of the three different tabs that exist on the home page so nothing learned from this. 

Instead of targeting the 'data' portion of the url for fuzzing I instead tried changing the number after data since it denotes the current scan. I changed the url to http://10.129.36.179:80/data/0 which brought me to a complete scan with results. I downloaded the pcap and opened it up in wireshark.

Examining this packet capture revealed an interaction with the ftp service open on port 21 which I noticed originally after the nmap scan. Tracing the ftp packets reveals a request for a username and password followed responses containing both in the clear. We now have credentials that will allow us access into ftp.

~~~
User: nathan
Pass: Buck3tH4TF0RM3!
~~~
