> http://www.binarytides.com/udp-socket-programming-in-winsock/

# UDP socket programming in winsock

UDP stands for User Datagram Protocol and is an alternative protocol to TCP the most common protocol used for data transfer over the internet. UDP is different from TCP in a number of ways. Most importantly UDP is a connectionless protocol.

In the TCP protocol first a connection is established by performing the 3 step handshake. This is done by calling the connect() socket function. However there is no such connection established in UDP. In simple terms when using the udp protocol, the client throws a packet at the server and its upto the server whether it catches it or not. If it fails then the udp protocol is not concerned.

This is opposite to what happens in the TCP protocol. If the receiver side fails to receive a packet then the sender side will find it out and resend the packet till it is properly received by the receiver. This is precisely the concept of connection.

In this article we are going to do some very simple udp socket programming by making a server and a client. We shall be doing this on the windows platform and on windows the socket api is called winsock. For coding its recommended to use VC++ 6.0 or 2010 express edition which is free and can be downloaded from microsoft.com

### Server

Here is the code for the server

	/*
		Simple UDP Server
		Silver Moon ( m00n.silv3r@gmail.com )
	*/
	
	#include<stdio.h>
	#include<winsock2.h>
	
	#pragma comment(lib,"ws2_32.lib") //Winsock Library
	
	#define BUFLEN 512	//Max length of buffer
	#define PORT 8888	//The port on which to listen for incoming data
	
	int main()
	{
		SOCKET s;
		struct sockaddr_in server, si_other;
		int slen , recv_len;
		char buf[BUFLEN];
		WSADATA wsa;
	
		slen = sizeof(si_other) ;
		
		//Initialise winsock
		printf("\nInitialising Winsock...");
		if (WSAStartup(MAKEWORD(2,2),&wsa) != 0)
		{
			printf("Failed. Error Code : %d",WSAGetLastError());
			exit(EXIT_FAILURE);
		}
		printf("Initialised.\n");
		
		//Create a socket
		if((s = socket(AF_INET , SOCK_DGRAM , 0 )) == INVALID_SOCKET)
		{
			printf("Could not create socket : %d" , WSAGetLastError());
		}
		printf("Socket created.\n");
		
		//Prepare the sockaddr_in structure
		server.sin_family = AF_INET;
		server.sin_addr.s_addr = INADDR_ANY;
		server.sin_port = htons( PORT );
		
		//Bind
		if( bind(s ,(struct sockaddr *)&server , sizeof(server)) == SOCKET_ERROR)
		{
			printf("Bind failed with error code : %d" , WSAGetLastError());
			exit(EXIT_FAILURE);
		}
		puts("Bind done");
	
		//keep listening for data
		while(1)
		{
			printf("Waiting for data...");
			fflush(stdout);
			
			//clear the buffer by filling null, it might have previously received data
			memset(buf,'\0', BUFLEN);
			
			//try to receive some data, this is a blocking call
			if ((recv_len = recvfrom(s, buf, BUFLEN, 0, (struct sockaddr *) &si_other, &slen)) == SOCKET_ERROR)
			{
				printf("recvfrom() failed with error code : %d" , WSAGetLastError());
				exit(EXIT_FAILURE);
			}
			
			//print details of the client/peer and the data received
			printf("Received packet from %s:%d\n", inet_ntoa(si_other.sin_addr), ntohs(si_other.sin_port));
			printf("Data: %s\n" , buf);
			
			//now reply the client with the same data
			if (sendto(s, buf, recv_len, 0, (struct sockaddr*) &si_other, slen) == SOCKET_ERROR)
			{
				printf("sendto() failed with error code : %d" , WSAGetLastError());
				exit(EXIT_FAILURE);
			}
		}
	
		closesocket(s);
		WSACleanup();
		
		return 0;
	}

To run the above program create a project in VC++ and compile and run. In Vc++ 2010 create an empty project and then add a c file.
The output should be something similar to this

	Initialising Winsock...Initialised.
	Socket created.
	Bind done
	Waiting for data...

Now this server can be tested with an application called netcat. Over here we shall use the ncat implementation of netcat. It comes with nmap. Download and install nmap. Then do the following in the terminal.

	C:\>ncat -vv -u localhost 8888
	Ncat: Version 6.01 ( http://nmap.org/ncat )
	Ncat: Connected to 127.0.0.1:8888.

So ncat shows that it is connected to our udp server program. The -u option is for udp. Now we can send some data to the server from the ncat terminal which will be echoed back.

	C:\>ncat -vv -u localhost 8888
	Ncat: Version 6.01 ( http://nmap.org/ncat )
	Ncat: Connected to 127.0.0.1:8888.
	hello
	hello
	world
	world

This is simple. Try it.

The netstat command can be used to check the udp server's open port. Here is a quick example

	C:\>netstat -p UDP -a
	
	Active Connections
	
	  Proto  Local Address          Foreign Address        State
	  UDP    ----------:microsoft-ds  *:*
	  UDP    ----------:isakmp      *:*
	  UDP    ----------:1025        *:*
	  UDP    ----------:1037        *:*
	  UDP    ----------:1039        *:*
	  UDP    ----------:4500        *:*
	  UDP    ----------:8888        *:*
	  UDP    ----------:17500       *:*
	  UDP    ----------:ntp         *:*
	  UDP    ----------:netbios-ns  *:*
	  UDP    ----------:netbios-dgm  *:*
	  UDP    ----------:1900        *:*
	  UDP    ----------:ntp         *:*
	  UDP    ----------:1900        *:*
	
	C:\>

Now the line "----------:8888".  
It is our udp server that is listening on port 8888.
It is interesting to note that the netstat command will not show any connections for any client that is connected to the udp server, for example ncat.

### Client

Now that we have tested our server with ncat, its time to make a client program which shall connect to the server and do the same things that the ncat program did earlier.

	/*
		Simple udp client
		Silver Moon (m00n.silv3r@gmail.com)
	*/
	#include<stdio.h>
	#include<winsock2.h>
	
	#pragma comment(lib,"ws2_32.lib") //Winsock Library
	
	#define SERVER "127.0.0.1"	//ip address of udp server
	#define BUFLEN 512	//Max length of buffer
	#define PORT 8888	//The port on which to listen for incoming data
	
	int main(void)
	{
		struct sockaddr_in si_other;
		int s, slen=sizeof(si_other);
		char buf[BUFLEN];
		char message[BUFLEN];
		WSADATA wsa;
	
		//Initialise winsock
		printf("\nInitialising Winsock...");
		if (WSAStartup(MAKEWORD(2,2),&wsa) != 0)
		{
			printf("Failed. Error Code : %d",WSAGetLastError());
			exit(EXIT_FAILURE);
		}
		printf("Initialised.\n");
		
		//create socket
		if ( (s=socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP)) == SOCKET_ERROR)
		{
			printf("socket() failed with error code : %d" , WSAGetLastError());
			exit(EXIT_FAILURE);
		}
		
		//setup address structure
		memset((char *) &si_other, 0, sizeof(si_other));
		si_other.sin_family = AF_INET;
		si_other.sin_port = htons(PORT);
		si_other.sin_addr.S_un.S_addr = inet_addr(SERVER);
		
		//start communication
		while(1)
		{
			printf("Enter message : ");
			gets(message);
			
			//send the message
			if (sendto(s, message, strlen(message) , 0 , (struct sockaddr *) &si_other, slen) == SOCKET_ERROR)
			{
				printf("sendto() failed with error code : %d" , WSAGetLastError());
				exit(EXIT_FAILURE);
			}
			
			//receive a reply and print it
			//clear the buffer by filling null, it might have previously received data
			memset(buf,'\0', BUFLEN);
			//try to receive some data, this is a blocking call
			if (recvfrom(s, buf, BUFLEN, 0, (struct sockaddr *) &si_other, &slen) == SOCKET_ERROR)
			{
				printf("recvfrom() failed with error code : %d" , WSAGetLastError());
				exit(EXIT_FAILURE);
			}
			
			puts(buf);
		}
	
		closesocket(s);
		WSACleanup();
	
		return 0;
	}
	
The client will ask user to input a message which it will send to the udp server and the udp server shall reply back with the message

	Initialising Winsock...Initialised.
	Enter message : hello
	hello
	Enter message : how are you
	how are you
	Enter message :

### Conclusion

So communication with udp sockets is quite simple. And therefore udp sockets are used where the communication itself is very simple, for example dns requests/response etc. Or when doing some kind of multicast/broadcast. Where its not a big issue if data fails to transfer, or resending the packet is not expensive, udp can be used.

UDP has other benefits as well. Since there is not establishment of connection, no verification that the packet reached or not, it uses lesser bandwidth and is faster than TCP.