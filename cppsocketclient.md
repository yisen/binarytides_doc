> http://www.binarytides.com/code-a-simple-socket-client-class-in-c/

# Code a simple socket client class in c++

### Wrapper class for socket functions

The standard socket library in C comes with a lot of functions for every task like connecting, sending data and receiving data etc. However knowing the syntax of all the functions and calling them again and again and in the right sequence could be a bit intimidating.

Using a class can help in such a situation. It has fewer functions and a simpler syntax. The class however does call the actual socket functions from the library.

In this article we would try to code a wrapper class for "client" functionality. A socket client connects to a certain server on a certain port number and then sends some data and then waits for a reply. And it keeps doing this for as long as the application wishes to. Common examples of such clients are web browser and ftp clients. Web browsers connect to websites on port 80 and fetch the html content of the webpage and then render them on the screen for the user.

Before moving on, it is suggested that you get your concepts clear on the basic socket operations. This [socket programming tutorial](http://www.binarytides.com/beginners-guide-to-socket-programming-in-c-on-linux/) can help.

### Code

Now here we have coded a simple class called tcp_client, which can be used to perform basic clientside socket operations like sending and receiving data from a tcp server on a certain port number.

The following code works only on linux.

	/**
		C++ client example using sockets
	*/
	#include<iostream>	//cout
	#include<stdio.h>	//printf
	#include<string.h>	//strlen
	#include<string>	//string
	#include<sys/socket.h>	//socket
	#include<arpa/inet.h>	//inet_addr
	#include<netdb.h>	//hostent
	
	using namespace std;
	
	/**
		TCP Client class
	*/
	class tcp_client
	{
	private:
		int sock;
		std::string address;
		int port;
		struct sockaddr_in server;
		
	public:
		tcp_client();
		bool conn(string, int);
		bool send_data(string data);
		string receive(int);
	};
	
	tcp_client::tcp_client()
	{
		sock = -1;
		port = 0;
		address = "";
	}
	
	/**
		Connect to a host on a certain port number
	*/
	bool tcp_client::conn(string address , int port)
	{
		//create socket if it is not already created
		if(sock == -1)
		{
			//Create socket
			sock = socket(AF_INET , SOCK_STREAM , 0);
			if (sock == -1)
			{
				perror("Could not create socket");
			}
			
			cout<<"Socket created\n";
		}
		else	{	/* OK , nothing */	}
		
		//setup address structure
		if(inet_addr(address.c_str()) == -1)
		{
			struct hostent *he;
			struct in_addr **addr_list;
			
			//resolve the hostname, its not an ip address
			if ( (he = gethostbyname( address.c_str() ) ) == NULL)
			{
				//gethostbyname failed
				herror("gethostbyname");
				cout<<"Failed to resolve hostname\n";
				
				return false;
			}
			
			//Cast the h_addr_list to in_addr , since h_addr_list also has the ip address in long format only
			addr_list = (struct in_addr **) he->h_addr_list;
	
			for(int i = 0; addr_list[i] != NULL; i++)
			{
				//strcpy(ip , inet_ntoa(*addr_list[i]) );
				server.sin_addr = *addr_list[i];
				
				cout<<address<<" resolved to "<<inet_ntoa(*addr_list[i])<<endl;
				
				break;
			}
		}
		
		//plain ip address
		else
		{
			server.sin_addr.s_addr = inet_addr( address.c_str() );
		}
		
		server.sin_family = AF_INET;
		server.sin_port = htons( port );
		
		//Connect to remote server
		if (connect(sock , (struct sockaddr *)&server , sizeof(server)) < 0)
		{
			perror("connect failed. Error");
			return 1;
		}
		
		cout<<"Connected\n";
		return true;
	}
	
	/**
		Send data to the connected host
	*/
	bool tcp_client::send_data(string data)
	{
		//Send some data
		if( send(sock , data.c_str() , strlen( data.c_str() ) , 0) < 0)
		{
			perror("Send failed : ");
			return false;
		}
		cout<<"Data send\n";
		
		return true;
	}
	
	/**
		Receive data from the connected host
	*/
	string tcp_client::receive(int size=512)
	{
		char buffer[size];
		string reply;
		
		//Receive a reply from the server
		if( recv(sock , buffer , sizeof(buffer) , 0) < 0)
		{
			puts("recv failed");
		}
		
		reply = buffer;
		return reply;
	}
	
	int main(int argc , char *argv[])
	{
		tcp_client c;
		string host;
		
		cout<<"Enter hostname : ";
		cin>>host;
		
		//connect to host
		c.conn(host , 80);
		
		//send some data
		c.send_data("GET / HTTP/1.1\r\n\r\n");
		
		//receive and echo reply
		cout<<"----------------------------\n\n";
		cout<<c.receive(1024);
		cout<<"\n\n----------------------------\n\n";
		
		//done
		return 0;
	}
	 
The output looks like this

	$ g++ client.cpp && ./a.out 
	Enter hostname : google.com
	Socket created
	google.com resolved to 74.125.236.200
	Connected
	Data send
	----------------------------
	
	HTTP/1.1 302 Found
	Location: http://www.google.co.in/
	Cache-Control: private
	Content-Type: text/html; charset=UTF-8
	Set-Cookie: PREF=ID=1502961bcf9bdea5:FF=0:TM=1347354195:LM=1347354195:S=w3DK3EmasMONNCdC; expires=Thu, 11-Sep-2014 09:03:15 GMT; path=/; domain=.google.com
	Set-Cookie: NID=63=EaA_pl5fP_JeXSfaY9YZ1v8SYaDBFmpU5ha2e3FVd3cURjsg_yC22N2l2soBAq9dfuzyXFDafN1fAL2Zr4AYKmrTjRgPdV4hiO-hZDDyAlHIA6D5MUVtEQnj5HS6v4Qw; expires=Wed, 13-Mar-2013 09:03:15 GMT; path=/; domain=.google.com; HttpOnly
	P3P: CP="This is not a P3P policy! See http://www.google.com/support/accounts/bin/answer.py?hl=en&answer=151657 for more info."
	Date: Tue, 11 Sep 2012 09:03:15 GMT
	Server: gws
	Content-Length: 221
	X-XSS-Protection: 1; mode=block
	X-Frame-Options: SAMEORIGIN
	
	<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
	<TITLE>302 Moved</TITLE></HEAD><BODY>
	<H1>302 Moved</H1>
	The document has moved
	<A HREF="http://www.google.co.in/">here</A>.
	</BODY></HTML>
	N
	
	----------------------------
	
	$

In the above example we connected to google.com and fetched the index page.
The code for doing this is this much

	c.conn(host , 80);
	c.send_data("GET / HTTP/1.1\r\n\r\n");
	
	cout<<c.receive(1024);

So instead of using the socket functions like socket, connect, send, recv with their full syntax it becomes very easy and simple to code when using a simple wrapper class which provides such functions.

The conn function of the tcp_client class is quite flexible. It can take a hostname or an ip address to connect to. If it is given an ip address, it uses it rightaway, if not, then it tries to resolve it to an ip address using the gethostbyname function.

### Further ideas

The above is a very simple and limited tcp client. It has to be enhanced in many different ways before being used in a real life application. For example the receive function is not quite flexible. It will receive only the given number of bytes and then stop. It needs to be coded such that it can receive full reply without failure. The class can be modified to work with both tcp and udp protocols.

So lots of things can be done. Try it out.