---
layout: post
title:  "Black Hat PHP"
date:   2017-03-07
comments: true
---

Provavelmente você já deve ter ouvido falar do livro [Black Hat Python](https://novatec.com.br/livros/black-hat-python/), se não ouviu, trata-se de uma obra onde se ensina a desenvolver em Python várias ferramentas que profissionais de segurança da informação (principalmente pentesters) utilizam frequentemente em sua rotina de trabalho. E como gosto muito do PHP, resolvi tentar reproduzir todos os códigos do livro (que estão em Python) em PHP, não tenho certeza se será possível desenvolver todos devido as diferenças entre as duas tecnologias, mas todo o progresso (ou não) será reproduzido aqui no blog.

Nesse primeiro post, vou descrever os 3 primeiros códigos do livro, que são: Cliente TCP, Cliente UDP e um Servidor TCP.

### Cliente TCP

O primeiro exemplo do livro, é um cliente TCP simples, que utiliza sockets para se conectar a um host remoto. Vamos ao código:

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

Para desenvolver, utilizei a biblioteca de socket padrão do PHP, para criar o socket, foi utilizada a função [socket_create](http://php.net/socket_create) , onde o parametro *AF_INET* especifica que o protocolo a ser utilizado é o IPV4, o *SOCK_STREAM* indica que o tipo de socket a ser utilizado, sendo este adequado ao TCP, e o *SOL_TCP* especifica que o protocolo a ser utilizado para comunição é o TCP.

Em seguida, é feita a conexão de socket (através da função [socket_connect](http://php.net/socket_connect)) com o host remoto definido na variável *$target_host* e a porta de destino em *$target_port*. Após conectar, é enviado uma string (um cabeçalho HTTP com o método GET) pelo socket com a função [socket_write](http://php.net/socket_write) para o host do google.com e logo após é impresso na tela a resposta do host através da função [socket_read](http://php.net/socket_read).
Ao testar o cliente, esse é o resultado:

<figure>
	<img src="{{ '/assets/img/blackhat-php-1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Execução de cliente TCP e resposta do google.com</figcaption>
</figure>

### Cliente UDP

No cliente UDP, o código é bem parecido com o do TCP, mudando apenas os parametros na criação do socket, e as funções de envio de mensagem e resposta do socket.

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

Na função de criação de socket [socket_create](http://php.net/socket_create) o parametro *SOCK_STREAM* foi substituido pelo *SOCK_DGRAM* que é o mais adequado para o UDP que não é orientado a conexão e utiliza [datagramas](https://pt.wikipedia.org/wiki/Datagrama), e logicamente, foi substituido *SOL_TCP* por *SOL_UDP*.

A funções de envio e recebimento de dados foram substituídas respectivamente por [socket_sendto](http://php.net/socket_sendto) e [socket_recvfrom](http://php.net/socket_recvfrom) que são melhores para lidar com o protocolo que não é orientado a conexão.

Para testar esse script, utilizei o [NetCat](https://www.vivaolinux.com.br/artigo/Netcat-O-canivete-suico-do-TCP-IP), onde abri e fiquei "escutando" uma porta UDP com o seguinte comando: `nc -u -l 7718 < teste_udp` , onde o arquivo *teste_udp* continha um texto simples, que é enviado para que se conecta nessa porta udp.

<figure>
	<img src="{{ '/assets/img/blackhat-php-2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig2. - Execução de cliente UDP e resposta do NetCat</figcaption>
</figure>

<figure>
	<img src="{{ '/assets/img/blackhat-php-3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig3. - Escuta na porta UDP com o NetCat e mensagem enviada pelo cliente UDP</figcaption>
</figure>

### Servidor TCP

Após a criação do cliente TCP e UDP, foi criado o Servidor TCP, ou seja, ao invés de efetuar a conexão, o socket ficará escutando em determinada porta e aguardando por conexões.


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

Neste caso, a criação do socket será igual ao do cliente TCP, e a seguir existem duas funções que não foram utilizadas nos últimos exemplos, a primeira é [socket_bind](http://php.net/socket_bind) que é utilizada para dizer o endereço de IP e a porta que queremos que o servidor escute, em seguida tem a [socket_listen](http://php.net/socket_listen) que faz o servidor iniciar a escuta.

A partir da criação do socket e inicialização da escuta do servidor, iniciamos um loop para mantê-lo em execução, e quando o cliente se conecta, colocamos o socket na variável *$client* através da função [socket_accept](http://php.net/socket_accept) e colocamos o endereço e a porta nas variáveis $address e $port passando estas por referência na função [socket_getpeername](http://php.net/socket_getpeername), para ler os dados enviados pelo cliente utilizamos a função [socket_read](http://php.net/socket_read) e para enviar dados para o cliente utilizamos.

Para testar este script basta executá-lo e em outra aba do terminal executar o cliente TCP, mas alterando antes as variáveis para `$target_host = "127.0.0.1"` e `$target_port = 9999` .


### Conclusão


Os primeiros 3 exemplos do livro foram bem fáceis de reproduzir em PHP, o próximo exemplo a ser desenvolvido será uma versão em PHP do NetCat (a ferramenta que utilizei para testar o cliente UDP), e será mais complexo.

Os códigos serão todos disponibilizados nesse repositório: [https://github.com/hiagohubert/blackhat-php](https://github.com/hiagohubert/blackhat-php).