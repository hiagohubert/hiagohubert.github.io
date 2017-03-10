---
layout: post
title:  "Black Hat PHP"
date:   2017-03-07
lang: en
ref: blackhat-php-1
comments: true
---

You may have heard of the book [Black Hat Python](https://novatec.com/books/black-hat-python/), if you have not heard, it is a book where you are taught to develop In Python various tools that information security professionals (mainly pentesters) often use in their work routine. And because I really like PHP, I decided to try to reproduce all the book codes (which are in Python) in PHP, I'm not sure if it will be possible to develop all because of the differences between the two technologies, but all progress (or not) will be reproduced Here in the blog.

In this first post, I will describe the first 3 codes of the book, which are: TCP Client, UDP Client and TCP Server.

### TCP Client

The first example of the book is a simple TCP client, which uses sockets to connect to a remote host. Let's code it:

```php
<?php

$target_host = "www.google.com";
$target_port = 80;

#create the socket
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP); #1

#connect the client
socket_connect($socket, $target_host, $target_port); #2

#send some data
socket_write($socket, "GET / HTTP/1.1\r\nHost: google.com\r\n\r\n"); #3

#receive some data
$response = socket_read($socket, 4096); #4

echo $response;
```

To develop, I used the standard PHP socket library to create the socket, the [socket_create] (http://php.net/socket_create) function was used, where the * AF_INET * parameter specifies that the protocol to be used is IPV4, * SOCK_STREAM * indicates that the type of socket to be used, which is suitable for TCP, and * SOL_TCP * specifies that the protocol to be used for communication is TCP.

Then the socket connection (via the [socket_connect](http://php.net/socket_connect) function) is made with the remote host defined in the *$target_host* variable and the target port at *$target_port*. After connecting, a string (an HTTP header with the GET method) is sent by the socket with the function [socket_write](http://php.net/socket_write) to the host of google.com and soon after it is printed on the screen Response from the host via the [socket_read](http://php.net/socket_read) function.

When testing the client, this is the result:

<figure>
	<img src="{{ '/assets/img/blackhat-php-1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - TCP client execution and google.com response</figcaption>
</figure>

### UDP Client

In the UDP client, the code is very similar to TCP, changing only the parameters in the socket creation, and the messaging and response functions of the socket.

```php
<?php

$target_host = "127.0.0.1";
$target_port = 7718;

#create the socket
$socket = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);

#connect the client
socket_connect($socket, $target_host, $target_port);

#send some data
socket_sendto($socket, "AAABBBCCC" , 100 , 0 , $target_host , $target_port);

#receive some data
$response = socket_recvfrom($socket, $buf, 4096, 0, $client_host, $client_port);

echo $buf;

```

In the socket creation function [socket_create](http://php.net/socket_create) the *SOCK_STREAM* parameter has been replaced by *SOCK_DGRAM* which is most suitable for UDP that is not connection oriented and uses [datagrams](Https://en.wikipedia.org/wiki/Datagram), and logically, was replaced *SOL_TCP* by *SOL_UDP*.

The data send and receive functions have been replaced respectively by [socket_sendto](http://php.net/socket_sendto) and [socket_recvfrom](http://php.net/socket_recvfrom) which are better for dealing with the protocol than Is not connection oriented.

In order to test this script, I used [NetCat](http://netcat.sourceforge.net/), where I opened and I was "listening" to a UDP port with the following command: `nc -u -l 7718 < test_udp`, where the *test_udp* file contained a plain text, which is sent to connect to that udp port.

<figure>
	<img src="{{ '/assets/img/blackhat-php-2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig2. - UDP Client Execution and NetCat Response</figcaption>
</figure>

<figure>
	<img src="{{ '/assets/img/blackhat-php-3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig3. - Listen on UDP port with NetCat and message sent by UDP client</figcaption>
</figure>

### TCP Server

After the creation of the TCP and UDP client, the TCP Server was created, that is, instead of making the connection, the socket will be listening on a certain port and waiting for connections.

```php
<?php

$bind_ip  = "0.0.0.0";
$bind_port = 9999;

$server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);

socket_bind($server, $bind_ip , $bind_port);

socket_listen($server, 5);

printf("[*] Listening on %s:%d", $bind_ip, $bind_port);


while(1){
    $client = socket_accept($server);
    socket_getpeername($client, $address, $port);
    
    printf("\n[*] Accepted connection from: %s:%d", $address,$port);
     
    #print out what the client sends
    $request = socket_read($client, 1024, PHP_NORMAL_READ);
   	
    printf("\n[*] Received: %s \n",$request);

    #send back a packet
    socket_write($client, "ACK!", strlen("ACK!"));
	
    socket_close($client);
 
}

socket_close($server);

```

In this case, the socket creation will be the same as the TCP client, and then there are two functions that were not used in the last examples, the first one is [socket_bind](http://php.net/socket_bind) that is used to say The IP address and port that we want the server to listen to, then it has [socket_listen](http://php.net/socket_listen) that makes the server to start listening.

From the socket creation and initialization of the server listener, we start a loop to keep it running, and when the client connects, we put the socket in the *$client* variable through the [socket_accept](http://php.net/socket_accept) function and put the address and port in the $address and $port variables by passing them by reference to the [socket_getpeername](http://php.net/socket_getpeername) function, to read the data sent by the client we use the Function [socket_read](http://php.net/socket_read) and to send data to the client we use [socket_write](http://php.net/socket_write) function.

To test this script simply run it and on another terminal tab run the TCP client, but changing before the variables to `$target_host = "127.0.0.1"` and `$target_port = 9999`.

### Conclusion


The first 3 examples of the book were very easy to reproduce in PHP, the next example to be developed will be a PHP version of NetCat (the tool I used to test the UDP client), and it will be more complex.

The codes will all be available in this repository: [https://github.com/hiagohubert/blackhat-php](https://github.com/hiagohubert/blackhat-php).