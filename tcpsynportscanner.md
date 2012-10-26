> http://www.binarytides.com/tcp-syn-port-scanning-with-metasploit/

# Tcp syn port scanning with metasploit

Metasploit can be used for tcp syn portscanning. The module auxiliary/scanner/portscan/syn can be used.
For tcp syn scanning metasploit must be run as root since tcp syn scanning requires raw socket, which require root privileges on a linux system. For example on ubuntu it should be run as "sudo msfconsole".

	msf > use auxiliary/scanner/portscan/syn 
	msf  auxiliary(syn) > show options
	
	Module options (auxiliary/scanner/portscan/syn):
	
	   Name       Current Setting  Required  Description
	   ----       ---------------  --------  -----------
	   BATCHSIZE  256              yes       The number of hosts to scan per set
	   INTERFACE                   no        The name of the interface
	   PORTS      1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
	   RHOSTS                      yes       The target address range or CIDR identifier
	   SNAPLEN    65535            yes       The number of bytes to capture
	   THREADS    1                yes       The number of concurrent threads
	   TIMEOUT    500              yes       The reply read timeout in milliseconds
	
	msf  auxiliary(syn) >

The options include specifying multiple remote hosts, port numbers and the number of threads. Lets take a quick example :

	msf  auxiliary(syn) > show options
	
	Module options (auxiliary/scanner/portscan/syn):
	
	   Name       Current Setting  Required  Description
	   ----       ---------------  --------  -----------
	   BATCHSIZE  256              yes       The number of hosts to scan per set
	   INTERFACE                   no        The name of the interface
	   PORTS      1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
	   RHOSTS                      yes       The target address range or CIDR identifier
	   SNAPLEN    65535            yes       The number of bytes to capture
	   THREADS    1                yes       The number of concurrent threads
	   TIMEOUT    500              yes       The reply read timeout in milliseconds
	
	msf  auxiliary(syn) > set RHOSTS 192.168.1.1
	RHOSTS => 192.168.1.1
	msf  auxiliary(syn) > set PORTS 1-1000
	PORTS => 1-1000
	msf  auxiliary(syn) > set THREADS 100
	THREADS => 100
	msf  auxiliary(syn) > set TIMEOUT 250
	TIMEOUT => 250
	msf  auxiliary(syn) > run
	
	[*]  TCP OPEN 192.168.1.1:21
	[*]  TCP OPEN 192.168.1.1:23
	[*]  TCP OPEN 192.168.1.1:80

So the output shows 3 ports open(21- ftp, 23 - telnet, 80 - http). The equivalent nmap command for this scan would be

	$ nmap -sS -T4 192.168.1.1 -p1-1000

The source code of the metasploit scanner can be viewed here. As can be seen its not very big in size and uses the pcap library.

### References

<http://www.metasploit.com/modules/auxiliary/scanner/portscan/syn>