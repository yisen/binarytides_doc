> http://www.binarytides.com/receive-full-data-with-recv-socket-function-in-c/

# Receive full data with recv socket function in C

### recv

The recv function is used to receive data on a socket. For example here is the code to fetch the home page of www.msn.com

	/**
		Simple TCP client to fetch a web page
		Silver Moon (m00n.silv3r@gmail.com)
	*/
	
	#include<stdio.h>
	#include<string.h>	//strlen
	#include<sys/socket.h>
	#include<arpa/inet.h>	//inet_addr
	
	int main(int argc , char *argv[])
	{
		int socket_desc;
		struct sockaddr_in server;
		char *message , server_reply[6000];
		
		//Create socket
		socket_desc = socket(AF_INET , SOCK_STREAM , 0);
		if (socket_desc == -1)
		{
			printf("Could not create socket");
		}
		
		//ip address of www.msn.com (get by doing a ping www.msn.com at terminal)
		server.sin_addr.s_addr = inet_addr("131.253.13.140");
		server.sin_family = AF_INET;
		server.sin_port = htons( 80 );
	
	
	
	
		//Connect to remote server
		if (connect(socket_desc , (struct sockaddr *)&server , sizeof(server)) < 0)
		{
			puts("connect error");
			return 1;
		}
		
		puts("Connected\n");
		
		//Send some data
		message = "GET /?st=1 HTTP/1.1\r\nHost: www.msn.com\r\n\r\n";
		if( send(socket_desc , message , strlen(message) , 0) < 0)
		{
			puts("Send failed");
			return 1;
		}
		puts("Data Send\n");
		
		//Receive a reply from the server
		if( recv(socket_desc, server_reply , 6000 , 0) < 0)
		{
			puts("recv failed");
		}
		puts("Reply received\n");
		puts(server_reply);
		
		return 0;
	}

The output might be something like this

	$ gcc simple_client.c && ./a.out
	Connected
	
	Data Send
	
	Reply received
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache, no-store
	Pragma: no-cache
	Content-Type: text/html; charset=utf-8
	Vary: Accept-Encoding
	P3P: CP="NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND"
	Set-Cookie: MC1=V=3&GUID=fd76ac7b61c9436f9a414441d186becd; domain=.msn.com; expires=Mon, 08-Sep-2014 09:58:09 GMT; path=/
	Set-Cookie: MC1=V=3&GUID=fd76ac7b61c9436f9a414441d186becd; domain=.msn.com; expires=Mon, 08-Sep-2014 09:58:09 GMT; path=/
	Set-Cookie: mh=MSFT; domain=.msn.com; expires=Mon, 08-Sep-2014 09:58:09 GMT; path=/
	Set-Cookie: CULTURE=EN-US; domain=.msn.com; expires=Sat, 15-Sep-2012 09:58:09 GMT; path=/
	Set-Cookie: CC=IN; domain=.msn.com; expires=Mon, 08-Sep-2014 09:58:09 GMT; path=/
	Set-Cookie: _FS=NU=1; domain=.msn.com; path=/
	Set-Cookie: _SS=SID=842BD72E52E84B36A38CD2350EB80138; domain=.msn.com; path=/
	Set-Cookie: SRCHD=D=2465878&MS=2465878&AF=NOFORM; expires=Mon, 08-Sep-2014 09:58:09 GMT; domain=.msn.com; path=/
	Set-Cookie: SRCHUID=V=2&GUID=9ACA924189EA4BF28C133336D34D51EF; expires=Mon, 08-Sep-2014 09:58:09 GMT; path=/
	Set-Cookie: SRCHUSR=AUTOREDIR=0&GEOVAR=&DOB=20120908; expires=Mon, 08-Sep-2014 09:58:09 GMT; domain=.msn.com; path=/
	errorCodeCount: [0:0]
	S: CO3SCH010130401
	Edge-control: no-store
	Date: Sat, 08 Sep 2012 09:58:09 GMT
	Content-Length: 111667
	
	<!DOCTYPE html><html xml:lang="en-us" lang="en-us" dir="ltr" xmlns="http://www.w3.org/1999/xhtml"><head><meta http-equiv=

The problem with the output is that it is incomplete. We already used a large sized buffer of 6000 characters to receive the reply. There cause of this problem is that we do not know the exact size of the response beforehand.

We might try to do any of these :

1. Use a very large buffer - But all data does not come in at once. The response comes as several TCP packets. So the recv function would return after receiving even partial data.

2. Making recv function wait till full size is received - But we do not know the full size.

A simple solution might look like this

	int receive_basic(int s)
	{
		int size_recv , total_size= 0;
		char chunk[CHUNK_SIZE];
		
		//loop
		while(1)
		{
			memset(chunk ,0 , CHUNK_SIZE);	//clear the variable
			if((size_recv =  recv(s , chunk , CHUNK_SIZE , 0) ) < 0)
			{
				break;
			}
			else
			{
				total_size += size_recv;
				printf("%s" , chunk);
			}
		}
		
		return total_size;
	}

If the CHUNK_SIZE is relatively small compared to the total size of the response then it will work but partially.
Lets say the CHUNK_SIZE = 512 and data is much larger. Then the recv function would receive data in a fashion similar to this

CHUNK_SIZE + CHUNK_SIZE + CHUNK_SIZE ..... + something less than CHUNK_SIZE.

The last call would be returning an amount of data that is smaller than the chunk size (unless the whole response size is a multiple of the CHUNK_SIZE).

Now in the last call to recv, recv would block till it gets CHUNK_SIZE amount of data or a default timeout occurs which can be from 30 seconds to 1 minute. This can be too much of waiting for an application.

An alternative strategy would be to receive without waiting and with a specific maximum timeout. Here are the steps ...

1. Try to receive some data, lets say 512 bytes. If nothing is received till a certain timeout then stop.
2. Go to step 1.

The code

	/**
		Simple TCP client to fetch a web page
		Silver Moon (m00n.silv3r@gmail.com)
	*/
	
	#include<stdio.h>	//printf
	#include<string.h>	//strlen
	#include<sys/socket.h>	//socket
	#include<arpa/inet.h>	//inet_addr
	#include<unistd.h>	//usleep
	#include<fcntl.h>	//fcntl
	
	//Size of each chunk of data received, try changing this
	#define CHUNK_SIZE 512
	
	//Receiving function
	int recv_timeout(int s, int timeout);
	
	int main(int argc , char *argv[])
	{
		int socket_desc;
		struct sockaddr_in server;
		char *message , server_reply[2000];
		
		//Create socket
		socket_desc = socket(AF_INET , SOCK_STREAM , 0);
		if (socket_desc == -1)
		{
			printf("Could not create socket");
		}
		
		//ip address of www.msn.com (get by doing a ping www.msn.com at terminal)
		server.sin_addr.s_addr = inet_addr("131.253.13.140");
		server.sin_family = AF_INET;
		server.sin_port = htons( 80 );
	
		//Connect to remote server
		if (connect(socket_desc , (struct sockaddr *)&server , sizeof(server)) < 0)
		{
			puts("connect error");
			return 1;
		}
		
		puts("Connected\n");
		
		//Send some data
		message = "GET /?st=1 HTTP/1.1\r\nHost: www.msn.com\r\n\r\n";
		if( send(socket_desc , message , strlen(message) , 0) < 0)
		{
			puts("Send failed");
			return 1;
		}
		puts("Data Send\n");
		
		//Now receive full data
		int total_recv = recv_timeout(socket_desc, 4);
		
		printf("\n\nDone. Received a total of %d bytes\n\n" , total_recv);
		return 0;
	}
	
	/*
		Receive data in multiple chunks by checking a non-blocking socket
		Timeout in seconds
	*/
	int recv_timeout(int s , int timeout)
	{
		int size_recv , total_size= 0;
		struct timeval begin , now;
		char chunk[CHUNK_SIZE];
		double timediff;
		
		//make socket non blocking
		fcntl(s, F_SETFL, O_NONBLOCK);
		
		//beginning time
		gettimeofday(&begin , NULL);
		
		while(1)
		{
			gettimeofday(&now , NULL);
			
			//time elapsed in seconds
			timediff = (now.tv_sec - begin.tv_sec) + 1e-6 * (now.tv_usec - begin.tv_usec);
			
			//if you got some data, then break after timeout
			if( total_size > 0 && timediff > timeout )
			{
				break;
			}
			
			//if you got no data at all, wait a little longer, twice the timeout
			else if( timediff > timeout*2)
			{
				break;
			}
			
			memset(chunk ,0 , CHUNK_SIZE);	//clear the variable
			if((size_recv =  recv(s , chunk , CHUNK_SIZE , 0) ) < 0)
			{
				//if nothing was received then we want to wait a little before trying again, 0.1 seconds
				usleep(100000);
			}
			else
			{
				total_size += size_recv;
				printf("%s" , chunk);
				//reset beginning time
				gettimeofday(&begin , NULL);
			}
		}
		
		return total_size;
	}
	
The recv_timeout function receives data on a socket with a given maximum timeout. The above program should fetch the full html content of www.msn.com till the closing html tag.

Data is received in chunks of size 512 bytes. This can be set to a different value like 1024 or 2000 or anything.
Another important change is that the socket is set in non blocking mode. When the socket is in non-blocking mode, then recv would not block and return immediately. If data is there it would return with the data otherwise with an error.

In the above example the socket can be kept blocking. Then the recv function must use the flag MSG_DONTWAIT

	if((size_recv = recv(s , chunk , CHUNK_SIZE , MSG_DONTWAIT) ) < 0)

Both techniques should produce identical results.

This technique of waiting for some timeout may not be the most reliable technique to ensure that full data has been received. The most reliable approach would be to have some sort of indicator that confirms completion of transfer. The reply can for example have and "end marker" or the total reponse size mentioned in it somewhere so that the receiving application can judge it better.

