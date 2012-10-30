> http://www.binarytides.com/port-scanning-and-network-discovery-with-nmap/

# Port scanning and network discovery with nmap

### Nmap

Nmap (Network Mapper) is the most popular port scanner and network discovery tool used. It is available for all major platforms. In this article we are going to learn the basics about nmap and see how it can be used to scan the network and ports.

Project website  
<http://nmap.org/>

Install on Ubuntu

	$ sudo apt-get install nmap

The nmap manual is available at  
<http://nmap.org/book/man.html>

Some nmap commands need to create raw sockets. This needs root privileges on a linux system, for example ubuntu. On windows nmap uses the winpcap packet driver to send raw packets.

### Scan network for live hosts - Ping Probe/Ping Sweep

This is the first and most basic form of network scan that can be done with nmap, to detect hosts that are alive and responding on the network.

	$ nmap -sP 192.168.1.1-254
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-15 18:45 IST
	Nmap scan report for 192.168.1.1
	Host is up (0.0069s latency).
	Nmap scan report for 192.168.1.2
	Host is up (0.0012s latency).
	Nmap scan report for 192.168.1.101
	Host is up (0.000065s latency).
	Nmap done: 254 IP addresses (3 hosts up) scanned in 6.64 seconds

In the above command we scan all ip addresses from 192.168.1.1 to 192.168.1.254
Thats the range and can be specified by the short syntax of 192.168.1.1-254

The **CIDR** notation can also be used, for example like this 192.168.1.1/24
Note : In CIDR notation the number after the forward slash indicates the bits of the ip address that stay constant from left site. So 24 means that "192.168.1" stays constant (8 bits x 3)

**Avoid DNS resolution**

When doing ping sweeps, nmap tries reverse dns resolution of the target ip addresses. This is generally not needed and can be disabled with the -n option.

	$ nmap -sP -n 192.168.1.1-255

Ok so lets move on and do more scanning with the tool.

### Port scan a host

To port scan a particular host, the command would be

	$ nmap 192.168.1.1
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-15 19:01 IST
	Nmap scan report for 192.168.1.1
	Host is up (0.058s latency).
	Not shown: 998 closed ports
	PORT   STATE SERVICE
	23/tcp open  telnet
	80/tcp open  http
	
	Nmap done: 1 IP address (1 host up) scanned in 0.87 seconds

Thats the simplest command to issue with nmap. Nmap performs a scan to discover open ports on the target host. It can be an ip address or a host/domain name as well. Nmap provides the port number, state and the service that port number if associated with. For example port 80 is for http. If http port is open then the target system is serving web pages most probably.

If you wish to dig deeper and analyse what nmap is doing behind the scene, you can use a packet sniffer like wireshark to analyse the packets that nmap is generating and sending.

**Getting the daemon/service banner or version information**

Nmap can try to get the version number of the banner of each of the services that are running on the host. The -sV flag can be used for this

	$ nmap -sV localhost
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-15 19:15 IST
	Nmap scan report for localhost (127.0.0.1)
	Host is up (0.00041s latency).
	Not shown: 991 closed ports
	PORT     STATE SERVICE   VERSION
	21/tcp   open  ftp       vsftpd 2.3.5
	22/tcp   open  ssh       OpenSSH 5.9p1 Debian 5ubuntu1 (protocol 2.0)
	25/tcp   open  smtp      Postfix smtpd
	53/tcp   open  domain    dnsmasq 2.59
	80/tcp   open  http      Apache httpd 2.2.22 ((Ubuntu))
	631/tcp  open  ipp       CUPS 1.5
	3000/tcp open  ntop-http Ntop web interface 4.1.0
	3306/tcp open  mysql     MySQL 5.5.24-0ubuntu0.12.04.1
	9050/tcp open  tor-socks Tor SOCKS Proxy
	Service Info: Host:  enlightened-desktop; OSs: Unix, Linux
	
	Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 7.91 seconds

Thats lots of information!! Port number, service name, version/banner information etc.

**Types of port scan**

Nmap does port scanning in a number of ways like tcp connect, syn scan, fin scan etc. The most popular ones are tcp connect and syn scan. In tcp connect scan a full TCP connection is established and in syn scan only half connection is established. When running as non-root on linux, nmap does tcp connect by default

	$ nmap 192.168.1.1

Syn scanning requires root privileges on linux systems. On ubuntu you have to do a sudo. To do a syn scan use the -sS option like this

	$ sudo nmap -sS 192.168.1.1

Syn scanning is faster since it does not establish a full TCP handshake. It is to some extent stealthier as well since old style firewalls may not be able to detect syn scans since full connection is not established. However modern firewalls can very well catch syn packets and detect port scanning attempts and stop the hacker right away.
However note that when nmap is run as root the default scanning technique used is syn scan. So the following are equivalent since in both cases nmap is running as root

	sudo nmap host
	sudo nmap -sS host

There are other types of port scanning techniques as well but we wont cover them in this article. So for more information check out the nmap manual at <http://nmap.org/book/man.html>  
Check out the -sF, -sX , -sA , -sN flags for more information on them

**Scanning specific ports only**

Nmap can be instructed to scan on specific ports or a range of port numbers by using the -p switch as follows :

	nmap -p1-1000 192.168.1.1/24

The above command would scan port numbers 1 to 1000 on all machines from 192.168.1.1 - 192.168.1.255

More examples :

	$ nmap -p22,23,100-150 192.168.10.0/24

The above will scan port numbers 22 , 23 and 100 to 150

	$ nmap -sU -pT:21,22,23,U:53,137 192.168.10.0/24

The above will scan TCP ports 21 22 and 23 and udp ports 53 and 137

**Skip online check**

Nmap by default first check if a host is online or not by doing a ping. If the host is not online then nmap would not port scan it. Many hosts now a days have firewalls installed that block ping requests. In such cases nmap can be instructed to not check if the host is online and that it should start port scan rightaway. This is done using the -PN option

	$ nmap 192.168.1.1 -PN

### Operating System detection

Nmap can try to find out the operating system on target system by doing some fingerprinting. This can be done by just using the -O switch. It also needs root privileges, since it uses raw sockets. Also note that if you are running some sort of firewall like firestart on linux or zonealarm on windows, then the firewalls may block raw sockets and as a result nmap would fail to show proper results.

	$ sudo nmap -O 192.168.1.1
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-16 12:17 IST
	Nmap scan report for 192.168.1.1
	Host is up (0.0067s latency).
	Not shown: 997 closed ports
	PORT     STATE SERVICE
	23/tcp   open  telnet
	80/tcp   open  http
	5431/tcp open  park-agent
	MAC Address: 00:21:2C:82:08:87 (SemIndia System Private Limited)
	Device type: general purpose
	Running: Linux 2.6.X
	OS details: Linux 2.6.13 - 2.6.28
	Network Distance: 1 hop
	
	OS detection performed. Please report any incorrect results at http://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 2.29 seconds

So nmap is able to detect that the operating system is Linux. It is important to note that OS fingerprint report by nmap may not be very accurate. It tries to discover the operating system by using some TCP header fields, but this technique cannot tell the exact linux distro for example. It can however in most cases give a correct indication as to whether the target is a linux or windows system.

Here is the scan result of a **windows** machine for example

	$ sudo nmap -O ############
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-16 14:20 IST
	Nmap scan report for ############ (###.###.###.###)
	Host is up (0.39s latency).
	Not shown: 987 filtered ports
	PORT      STATE SERVICE
	21/tcp    open  ftp
	25/tcp    open  smtp
	53/tcp    open  domain
	80/tcp    open  http
	110/tcp   open  pop3
	143/tcp   open  imap
	443/tcp   open  https
	1433/tcp  open  ms-sql-s
	2006/tcp  open  invokator
	3306/tcp  open  mysql
	3389/tcp  open  ms-term-serv
	8443/tcp  open  https-alt
	49158/tcp open  unknown
	Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
	Device type: general purpose
	Running: Microsoft Windows 2008
	OS details: Microsoft Windows Server 2008 Beta 3
	
	OS detection performed. Please report any incorrect results at http://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 60.22 seconds

### Aggressive scanning

The -A option can be used to perform an aggressive scan which is equal to - "enable OS detection and Version detection, Script scanning and Traceroute". Here is a quick example

	$ sudo nmap -A -T4 ##########
	[sudo] password for enlightened: 
	
	Starting Nmap 5.21 ( http://nmap.org ) at 2012-08-16 15:02 IST
	Nmap scan report for ########## (###.###.###.###)
	Host is up (0.38s latency).
	Not shown: 989 filtered ports
	PORT      STATE SERVICE       VERSION
	21/tcp    open  ftp           Microsoft ftpd
	25/tcp    open  smtp          MailEnable smptd 4.26--
	53/tcp    open  domain        ISC BIND hostmaster
	80/tcp    open  http          Microsoft IIS webserver 7.0
	|_html-title: Welcome to Homepage
	110/tcp   open  pop3          MailEnable POP3 Server
	|_pop3-capabilities: OK(K Capability list follows) USER TOP UIDL
	143/tcp   open  imap          MailEnable imapd
	|_imap-capabilities: IMAP4rev1 IMAP4 CHILDREN IDLE AUTH=LOGIN AUTH=CRAM-MD5
	2006/tcp  open  http          Microsoft IIS httpd 7.0
	| html-title: Document Moved
	|_Requested resource was http://##########/ABC
	3306/tcp  open  mysql         MySQL (unauthorized)
	3389/tcp  open  microsoft-rdp Microsoft Terminal Service
	8443/tcp  open  ssl/http      Microsoft IIS webserver 7.0
	|_sslv2: server still supports SSLv2
	|_html-title: Site doesn't have a title (text/html).
	49158/tcp open  msrpc         Microsoft Windows RPC
	Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
	Device type: general purpose
	Running: Microsoft Windows 2008
	OS details: Microsoft Windows Server 2008 Beta 3
	Network Distance: 16 hops
	Service Info: Host: CL-T192-200CN.home; OS: Windows
	
	TRACEROUTE (using port 21/tcp)
	HOP RTT       ADDRESS
	1   8.13 ms   192.168.1.1
	2   44.42 ms  117.194.224.1
	3   40.74 ms  218.248.162.230
	4   70.79 ms  218.248.255.82
	5   124.74 ms 115.114.130.33.STATIC-Chennai.vsnl.net.in (115.114.130.33)
	6   148.41 ms 172.31.19.146
	7   145.28 ms ix-0-100.tcore1.MLV-Mumbai.as6453.net (180.87.38.5)
	8   366.30 ms if-2-2.tcore2.MLV-Mumbai.as6453.net (180.87.38.2)
	9   375.30 ms if-6-2.tcore1.L78-London.as6453.net (80.231.130.5)
	10  372.00 ms if-2-2.tcore2.L78-London.as6453.net (80.231.131.1)
	11  428.80 ms if-20-2.tcore2.NYY-NewYork.as6453.net (216.6.99.13)
	12  442.52 ms if-1-0-0.mcore3.MTT-Montreal.as6453.net (216.6.99.10)
	13  382.34 ms if-0-3-1-0.tcore1.MTT-Montreal.as6453.net (64.86.31.53)
	14  364.63 ms 64.86.31.42
	15  ...
	16  369.24 ms ###.###.###.###
	
	OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 107.29 seconds

For privacy the actual host name its ip address have been hidden.
A new parameter -T has been used in the above example. The T parameter can be used to adjust the speed of the scan. It takes values from 0-5. 0 being the slowest and 5 being the fastest. Over here we used 4.

Apart from open ports, and operating system details, we also have the traceroute output.

### Saving output to file

Nmap can save the scan results to various kinds of file formats like normal text, xml etc. The options to use are -oN -oX -oS -oG and -oA. The oA option = oN + oX + oG.

Quick example

	$ nmap -sP -n 192.168.1.1-255 -oA lan_scan.txt

The above will create lan_scan.txt.gnmap ,lan_scan.txt.nmap and lan_scan.txt.xml files

lan_scan.txt.nmap file looks like this

	# Nmap 5.21 scan initiated Thu Aug 16 15:33:45 2012 as: nmap -sP -n -oA lan_scan.txt 192.168.1.1-255
	Nmap scan report for 192.168.1.1
	Host is up (0.0073s latency).
	Nmap scan report for 192.168.1.2
	Host is up (0.0010s latency).
	Nmap scan report for 192.168.1.101
	Host is up (0.00021s latency).
	# Nmap done at Thu Aug 16 15:33:48 2012 -- 255 IP addresses (3 hosts up) scanned in 2.51 seconds

Resources

1.	<http://nmap.org/book/man.html>