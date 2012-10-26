> 转自 <http://www.binarytides.com/python-program-to-fetch-domain-whois-data-using-sockets/>

# Python program to fetch domain whois data using sockets

### Whois
***

The whois information of a domain name provides various details like registrar, owner, registration date, expiry date etc. The whois information is provided by the corresponding whois servers of the registrars. The first step is to contact whois.iana.org which provides the actual whois server of a domain name. Next the particular whois server is contacted which provides the full whois data of the domain.

Implementation is quite simple and python makes it even simpler.

	#!/usr/bin/python

	'''
	Program to fetch whois information of a domain name
	Silver Moon
	m00n.silv3r@gmail.com
	'''
	import socket, sys

	#Perform a generic whois query to a server and get the reply
	def perform_whois(server , query) :
		#socket connection
		s = socket.socket(socket.AF_INET , socket.SOCK_STREAM)
		s.connect((server , 43))
		
		#send data
		s.send(query + '\r\n')
		
		#receive reply
		msg = ''
		while len(msg) < 10000:
			chunk = s.recv(100)
			if(chunk == ''):
				break
			msg = msg + chunk
		
		return msg
	#End

	#Function to perform the whois on a domain name
	def get_whois_data(domain):
		
		#remove http and www
		domain = domain.replace('http://','')
		domain = domain.replace('www.','')
		
		#get the extension , .com , .org , .edu
		ext = domain[-3:]
		
		#If top level domain .com .org .net
		if(ext == 'com' or ext == 'org' or ext == 'net'):
			whois = 'whois.internic.net'
			msg = perform_whois(whois , domain)
			
			#Now scan the reply for the whois server
			lines = msg.splitlines()
			for line in lines:
				if ':' in line:
					words = line.split(':')
					if  'Whois' in words[0] and 'whois.' in words[1]:
						whois = words[1].strip()
						break;
		
		#Or Country level - contact whois.iana.org to find the whois server of a particular TLD
		else:
			#Break again like , co.uk to uk
			ext = domain.split('.')[-1]
			
			#This will tell the whois server for the particular country
			whois = 'whois.iana.org'
			msg = perform_whois(whois , ext)
			
			#Now search the reply for a whois server
			lines = msg.splitlines()
			for line in lines:
				if ':' in line:
					words = line.split(':')
					if 'whois.' in words[1] and 'Whois Server (port 43)' in words[0]:
						whois = words[1].strip()
						break;
		
		#Now contact the final whois server
		msg = perform_whois(whois , domain)
		
		#Return the reply
		return msg
	# end
	        
	# get the domain name from command line argument
	domain_name = sys.argv[1]
	print get_whois_data(domain_name)

Run the program by issuing the following command at the terminal.

	$ python whois.py stackoverflow.com

The commandline argument should contain the domain name. The output would be the whois data. 



