> http://www.binarytides.com/udp-socket-programming-in-java/

# Udp socket programming in Java

### Datagram sockets

UDP sockets can be used in java with the DatagramSocket class.

### Server

Let code a simple udp server that listens on a certain port number.

	/**
		Java ECHO server with UDP sockets example
		Silver Moon (m00n.silv3r@gmail.com)
	*/
	
	import java.io.*;
	import java.net.*;
	
	public class udp_server
	{
		public static void main(String args[])
		{
			DatagramSocket sock = null;
			
			try
			{
				//1. creating a server socket, parameter is local port number
				sock = new DatagramSocket(7777);
				
				//buffer to receive incoming data
				byte[] buffer = new byte[65536];
				DatagramPacket incoming = new DatagramPacket(buffer, buffer.length);
				
				//2. Wait for an incoming data
				echo("Server socket created. Waiting for incoming data...");
				
				//communication loop
				while(true)
				{
					sock.receive(incoming);
					byte[] data = incoming.getData();
					String s = new String(data, 0, incoming.getLength());
					
					//echo the details of incoming data - client ip : client port - client message
					echo(incoming.getAddress().getHostAddress() + " : " + incoming.getPort() + " - " + s);
					
					s = "OK : " + s;
					DatagramPacket dp = new DatagramPacket(s.getBytes() , s.getBytes().length , incoming.getAddress() , incoming.getPort());
					sock.send(dp);
				}
			}
			
			catch(IOException e)
			{
				System.err.println("IOException " + e);
			}
		}
		
		//simple function to echo data to terminal
		public static void echo(String msg)
		{
			System.out.println(msg);
		}
	}

Run this program by typing the following at terminal/command line

	$ javac udp_server.java && java udp_server
	Server socket created. Waiting for incoming data...

Now the udp server is up and waiting for incoming data. To check that udp server is really up, the netstat command can be used.

**On Linux**

	$ netstat -u -ap
	(Not all processes could be identified, non-owned process info
	 will not be shown, you would have to be root to see it all.)
	Active Internet connections (servers and established)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	udp        0      0 localhost:11211         *:*                                 -               
	udp        0      0 localhost:48568         localhost:48568         ESTABLISHED -               
	udp        0      0 localhost:domain        *:*                                 -               
	udp        0      0 *:ipp                   *:*                                 -               
	udp        0      0 *:17500                 *:*                                 4103/dropbox    
	udp        0      0 *:mdns                  *:*                                 -               
	udp        0      0 *:34271                 *:*                                 -               
	udp6       0      0 [::]:43649              [::]:*                              -               
	udp6       0      0 [::]:7777               [::]:*                              6277/java       
	udp6       0      0 [::]:mdns               [::]:*                              -


Note the "[::]:7777".
It is our java udp server. The output also shows the pid (6277) and the command name (java).

Now that our server is up and running, its time to connect to it and verify its working. To connect to a simple server like this a program like telnet is needed. However the standard telnet utilities that ship with linux do not support udp. Hence we are going to use another utility called ncat. Ncat is a netcat implementation from nmap.
On ubuntu install nmap from synaptic shall also install ncat.

Quick example

	$ ncat -vv localhost 7777 -u
	Ncat: Version 5.21 ( http://nmap.org/ncat )
	Ncat: Connected to 127.0.0.1:7777.

The ncat utility is now connected to our udp server. The 'vv' switch is for very verbose. It shows detailed information of what is going on. The 'u' option is for udp protocol.

Now that we are connected, its time to send some message to the server. Just type something and hit enter.

	$ ncat -vv localhost 7777 -u
	Ncat: Version 5.21 ( http://nmap.org/ncat )
	Ncat: Connected to 127.0.0.1:7777.
	hello
	OK : hello
	how are you
	OK : how are you

Ok, so here we can see that the server replies with the same message with "OK : " prepended.
Multiple ncat terminals can connect to this udp server. Open another terminal and connect using the same command and try sending messages from both client terminals. The server would reply to each. Since the concept of "connection" does not exist in udp, the same loop can handle multiple connections much like the select function of c socket library.

Now that we have tested our server, its time to write a client which shall be used in place of ncat.

### Client

Here is the code

	/**
		Java ECHO client with UDP sockets example
		Silver Moon (m00n.silv3r@gmail.com)
	*/
	
	import java.io.*;
	import java.net.*;
	
	public class udp_client
	{
		public static void main(String args[])
		{
			DatagramSocket sock = null;
			int port = 7777;
			String s;
			
			BufferedReader cin = new BufferedReader(new InputStreamReader(System.in));
			
			try
			{
				sock = new DatagramSocket();
				
				InetAddress host = InetAddress.getByName("localhost");
				
				while(true)
				{
					//take input and send the packet
					echo("Enter message to send : ");
					s = (String)cin.readLine();
					byte[] b = s.getBytes();
					
					DatagramPacket  dp = new DatagramPacket(b , b.length , host , port);
					sock.send(dp);
					
					//now receive reply
					//buffer to receive incoming data
					byte[] buffer = new byte[65536];
					DatagramPacket reply = new DatagramPacket(buffer, buffer.length);
					sock.receive(reply);
					
					byte[] data = reply.getData();
					s = new String(data, 0, reply.getLength());
					
					//echo the details of incoming data - client ip : client port - client message
					echo(reply.getAddress().getHostAddress() + " : " + reply.getPort() + " - " + s);
				}
			}
			
			catch(IOException e)
			{
				System.err.println("IOException " + e);
			}
		}
		
		//simple function to echo data to terminal
		public static void echo(String msg)
		{
			System.out.println(msg);
		}
	}

Run the client

	$ javac udp_client.java && java udp_client
	Enter message to send : 
	Hello
	127.0.0.1 : 7777 - OK : Hello
	Enter message to send : 
	How are you
	127.0.0.1 : 7777 - OK : How are you

Before running the client, make sure that the server is running in another terminal/console.
Whatever message the client sends, is send back with "OK : " added to it. The client the echoes the message onto the terminal.

So this completes the server and client communication using udp sockets.
	