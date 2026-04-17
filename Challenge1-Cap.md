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

With these credentials I was able to confirm the ability to access FTP and more importantly, I was able to use them to successfully SSH into the machine with the following command:

~~~
ssh nathan@10.129.38.183

Response:
nathan@10.129.38.183's password:

Buck3tH4TF0RM3!

Response:
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Apr 17 18:11:10 UTC 2026

  System load:           0.08
  Usage of /:            36.7% of 8.73GB
  Memory usage:          20%
  Swap usage:            0%
  Processes:             222
  Users logged in:       0
  IPv4 address for eth0: 10.129.38.183
  IPv6 address for eth0: dead:beef::250:56ff:feb0:4f1

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

63 updates can be applied immediately.
42 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu May 27 11:21:27 2021 from 10.10.14.7
~~~

At this point I had access to nathan's home directory and the first flag for this machine.

~~~
788f2ff82fedabf0806514a4e5cf3627
~~~

The final step was to collect the flag in the root directory. My first attempt at this was to simply navigate to the root directory with cd, but was met with a permissions error. My next thought was to use sudo cd to use elevated privileges to access the root directory. Unfortunately, this did not work as I was met with the following message:

~~~
nathan is not in the sudoers file.  This incident will be reported.
~~~

Not good! With this action I've left a serious trace of my presence in this machine and if it were connected to an enterprise network with an IR team they would surely investigate this incident very quickly. 
