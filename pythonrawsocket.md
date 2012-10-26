> 转自 <http://www.binarytides.com/raw-socket-programming-in-python-linux/>

Raw sockets allow a program or application to provide custom headers for the specific protocol(tcp ip) which are otherwise provided by the kernel/os network stack. In more simple terms its for adding custom headers instead of headers provided by the underlying operating system.

Raw socket support is available natively in the socket api in linux. This is different from windows where it is absent (it became available in windows 2000/xp/xp sp1 but was removed later). Although raw sockets dont find much use in common networking applications, they are used widely in applications related to network security.

In this article we are going to create raw tcp/ip packets. For this we need to know how to make proper ip header and tcp headers. A packet = Ip header + Tcp header + data.

So lets have a look at the structures.

**Ip header**

According to [RFC 791](http://www.ietf.org/rfc/rfc791.txt)

		0                   1                   2                   3   
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |Version|  IHL  |Type of Service|          Total Length         |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |         Identification        |Flags|      Fragment Offset    |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |  Time to Live |    Protocol   |         Header Checksum       |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                       Source Address                          |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                    Destination Address                        |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                    Options                    |    Padding    |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Every single number is 1 bit. So for example the Version field is 4 bit. The header must be constructed exactly like shown.

**TCP header**

Next comes the TCP header. According to [RFC 793](http://www.ietf.org/rfc/rfc793.txt)

		0                   1                   2                   3   
	    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |          Source Port          |       Destination Port        |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                        Sequence Number                        |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                    Acknowledgment Number                      |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |  Data |           |U|A|P|R|S|F|                               |
	   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
	   |       |           |G|K|H|T|N|N|                               |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |           Checksum            |         Urgent Pointer        |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                    Options                    |    Padding    |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
	   |                             data                              |
	   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

### Create a raw socket
***

Raw socket can be created in python like this

	#create a raw socket
	try:
		s = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_RAW)
	except socket.error , msg:
		print 'Socket could not be created. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
		sys.exit()												

To create raw socket, the program must have root privileges on the system. For example on ubuntu run the program with sudo. The above example creates a raw socket of type IPPROTO_RAW which is a raw IP packet. Means that we provide everything including the ip header.

Once the socket is created, next thing is to create and construct the packet that is to be send out. C like structures are not available in python, therefore the functions called pack and unpack have to be used to create the packet in the structure specified above.

So first, lets make the ip header

	source_ip = '192.168.1.101'
	dest_ip = '192.168.1.1'	# or socket.gethostbyname('www.google.com')

	# ip header fields
	ip_ihl = 5
	ip_ver = 4
	ip_tos = 0
	ip_tot_len = 0	# kernel will fill the correct total length
	ip_id = 54321	#Id of this packet
	ip_frag_off = 0
	ip_ttl = 255
	ip_proto = socket.IPPROTO_TCP
	ip_check = 0	# kernel will fill the correct checksum
	ip_saddr = socket.inet_aton ( source_ip )	#Spoof the source ip address if you want to
	ip_daddr = socket.inet_aton ( dest_ip )

	ip_ihl_ver = (version << 4) + ihl

	# the ! in the pack format string means network order
	ip_header = pack('!BBHHHBBH4s4s' , ip_ihl_ver, ip_tos, ip_tot_len, ip_id, ip_frag_off, ip_ttl, ip_proto, ip_check, ip_saddr, ip_daddr)

Now ip_header has the data for the ip header. Now the usage of pack function, it packs some values has bytes, some as 16bit fields and some as 32 bit fields.

Next comes the tcp header

	# tcp header fields
	tcp_source = 1234	# source port
	tcp_dest = 80	# destination port
	tcp_seq = 454
	tcp_ack_seq = 0
	tcp_doff = 5	#4 bit field, size of tcp header, 5 * 4 = 20 bytes
	#tcp flags
	tcp_fin = 0
	tcp_syn = 1
	tcp_rst = 0
	tcp_psh = 0
	tcp_ack = 0
	tcp_urg = 0
	tcp_window = socket.htons (5840)	#	maximum allowed window size
	tcp_check = 0
	tcp_urg_ptr = 0

	tcp_offset_res = (tcp_doff << 4) + 0
	tcp_flags = tcp_fin + (tcp_syn << 1) + (tcp_rst << 2) + (tcp_psh <<3) + (tcp_ack << 4) + (tcp_urg << 5)

	# the ! in the pack format string means network order
	tcp_header = pack('!HHLLBBHHH' , tcp_source, tcp_dest, tcp_seq, tcp_ack_seq, tcp_offset_res, tcp_flags,  tcp_window, tcp_check, tcp_urg_ptr)		

The construction of the tcp header is similar to the ip header. The tcp header has a field called checksum which needs to be filled in correctly. A pseudo header is constructed to compute the checksum. The checksum is calculated over the tcp header along with the data. Checksum is necessary to detect errors in the transmission on the receiver side.

### Code
***

Here is the full code to send a raw packet

	'''
		Raw sockets on Linux
		
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
			w = ord(msg[i]) + (ord(msg[i+1]) << 8 )
			s = s + w
		
		s = (s>>16) + (s & 0xffff);
		s = s + (s >> 16);
		
		#complement and mask to 4 byte short
		s = ~s & 0xffff
		
		return s

	#create a raw socket
	try:
		s = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_RAW)
	except socket.error , msg:
		print 'Socket could not be created. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
		sys.exit()

	# tell kernel not to put in headers, since we are providing it, when using IPPROTO_RAW this is not necessary
	# s.setsockopt(socket.IPPROTO_IP, socket.IP_HDRINCL, 1)
		
	# now start constructing the packet
	packet = '';

	source_ip = '192.168.1.101'
	dest_ip = '192.168.1.1'	# or socket.gethostbyname('www.google.com')

	# ip header fields
	ip_ihl = 5
	ip_ver = 4
	ip_tos = 0
	ip_tot_len = 0	# kernel will fill the correct total length
	ip_id = 54321	#Id of this packet
	ip_frag_off = 0
	ip_ttl = 255
	ip_proto = socket.IPPROTO_TCP
	ip_check = 0	# kernel will fill the correct checksum
	ip_saddr = socket.inet_aton ( source_ip )	#Spoof the source ip address if you want to
	ip_daddr = socket.inet_aton ( dest_ip )

	ip_ihl_ver = (ip_ver << 4) + ip_ihl

	# the ! in the pack format string means network order
	ip_header = pack('!BBHHHBBH4s4s' , ip_ihl_ver, ip_tos, ip_tot_len, ip_id, ip_frag_off, ip_ttl, ip_proto, ip_check, ip_saddr, ip_daddr)

	# tcp header fields
	tcp_source = 1234	# source port
	tcp_dest = 80	# destination port
	tcp_seq = 454
	tcp_ack_seq = 0
	tcp_doff = 5	#4 bit field, size of tcp header, 5 * 4 = 20 bytes
	#tcp flags
	tcp_fin = 0
	tcp_syn = 1
	tcp_rst = 0
	tcp_psh = 0
	tcp_ack = 0
	tcp_urg = 0
	tcp_window = socket.htons (5840)	#	maximum allowed window size
	tcp_check = 0
	tcp_urg_ptr = 0

	tcp_offset_res = (tcp_doff << 4) + 0
	tcp_flags = tcp_fin + (tcp_syn << 1) + (tcp_rst << 2) + (tcp_psh <<3) + (tcp_ack << 4) + (tcp_urg << 5)

	# the ! in the pack format string means network order
	tcp_header = pack('!HHLLBBHHH' , tcp_source, tcp_dest, tcp_seq, tcp_ack_seq, tcp_offset_res, tcp_flags,  tcp_window, tcp_check, tcp_urg_ptr)

	user_data = 'Hello, how are you'

	# pseudo header fields
	source_address = socket.inet_aton( source_ip )
	dest_address = socket.inet_aton(dest_ip)
	placeholder = 0
	protocol = socket.IPPROTO_TCP
	tcp_length = len(tcp_header) + len(user_data)

	psh = pack('!4s4sBBH' , source_address , dest_address , placeholder , protocol , tcp_length);
	psh = psh + tcp_header + user_data;

	tcp_check = checksum(psh)
	#print tcp_checksum

	# make the tcp header again and fill the correct checksum - remember checksum is NOT in network byte order
	tcp_header = pack('!HHLLBBH' , tcp_source, tcp_dest, tcp_seq, tcp_ack_seq, tcp_offset_res, tcp_flags,  tcp_window) + pack('H' , tcp_check) + pack('!H' , tcp_urg_ptr)

	# final full packet - syn packets dont have any data
	packet = ip_header + tcp_header + user_data

	#Send the packet finally - the port specified has no effect
	s.sendto(packet, (dest_ip , 0 ))	# put this in a loop if you want to flood the target

Run the above program from the terminal and check the network traffic using a packet sniffer like wireshark. It should show the packet.

Raw sockets find application in the field of network security. The above example can be used to code a tcp syn flood program. Syn flood programs are used in Dos attacks. Raw sockets are also used to code packet sniffers, port scanners etc. 

