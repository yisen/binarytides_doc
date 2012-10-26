> http://www.binarytides.com/udp-socket-programming-in-php/

# UDP socket programming in php

In a previous [article](http://www.binarytides.com/php-socket-programming-tutorial-for-beginners/) we learnt about writing simple server and clients using TCP sockets in php. In this article we are going to use udp sockets for the same.

UDP sockets are much simpler to work with since they are connection-less. A udp server just has an socket that waits to receive some data and a socket client can send data on a socket without connection.

### Server

So lets code up the server.

	<?php
	
	//Reduce errors
	error_reporting(~E_WARNING);
	
	//Create a UDP socket
	if(!($sock = socket_create(AF_INET, SOCK_DGRAM, 0)))
	{
		$errorcode = socket_last_error();
	    $errormsg = socket_strerror($errorcode);
	    
	    die("Couldn't create socket: [$errorcode] $errormsg \n");
	}
	
	echo "Socket created \n";
	
	// Bind the source address
	if( !socket_bind($sock, "0.0.0.0" , 9999) )
	{
		$errorcode = socket_last_error();
	    $errormsg = socket_strerror($errorcode);
	    
	    die("Could not bind socket : [$errorcode] $errormsg \n");
	}
	
	echo "Socket bind OK \n";
	
	//Do some communication, this loop can handle multiple clients
	while(1)
	{
		echo "Waiting for data ... \n";
		
		//Receive some data
		$r = socket_recvfrom($sock, $buf, 512, 0, $remote_ip, $remote_port);
		echo "$remote_ip : $remote_port -- " . $buf;
		
		//Send back the data to the client
		socket_sendto($sock, "OK " . $buf , 100 , 0 , $remote_ip , $remote_port);
	}
	
	socket_close($sock);

The above program opens a udp server on port 9999 of localhost.

Run the above program from the terminal and it should show

	$ php server.php 
	Socket created 
	Socket bind OK 
	Waiting for data ...

This udp server can handle multiple clients since it does not use a hardbound connection and simply replies to whoever came in.

### Client

Now that our server is running fine, its time to code a client program that would connect to the server and communicate.

	<?php
	
	/*
		Simple php udp socket client
	*/
	
	//Reduce errors
	error_reporting(~E_WARNING);
	
	$server = '127.0.0.1';
	$port = 9999;
	
	if(!($sock = socket_create(AF_INET, SOCK_DGRAM, 0)))
	{
		$errorcode = socket_last_error();
	    $errormsg = socket_strerror($errorcode);
	    
	    die("Couldn't create socket: [$errorcode] $errormsg \n");
	}
	
	echo "Socket created \n";
	
	//Communication loop
	while(1)
	{
		//Take some input to send
		echo 'Enter a message to send : ';
		$input = fgets(STDIN);
		
		//Send the message to the server
		if( ! socket_sendto($sock, $input , strlen($input) , 0 , $server , $port))
		{
			$errorcode = socket_last_error();
			$errormsg = socket_strerror($errorcode);
			
			die("Could not send data: [$errorcode] $errormsg \n");
		}
			
		//Now receive reply from server and print it
		if(socket_recv ( $sock , $reply , 2045 , MSG_WAITALL ) === FALSE)
		{
			$errorcode = socket_last_error();
			$errormsg = socket_strerror($errorcode);
			
			die("Could not receive data: [$errorcode] $errormsg \n");
		}
		
		echo "Reply : $reply";
	}
	
The above client asks user to input some message which is send to the server. Then whatever reply the server sends back is printed.

The output shall be like this

	$ php client.php 
	Socket created 
	Enter a message to send : Hello
	Reply : OK Hello
	Enter a message to send : World
	Reply : OK World
	Enter a message to send :

### Conclusion

So in the above demonstration we saw how udp sockets can be used in php. UDP sockets are used in those type of network communication where reliability is not very important due to a number of reasons.

The above shown examples are very basic and would need a lot of modification to make ready for suitable use. So try them out.
