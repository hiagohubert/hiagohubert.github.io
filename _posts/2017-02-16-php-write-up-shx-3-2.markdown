---
layout: post
title:  "Write up SHX 3 (PHP Security) - Quantum Photos"
date:   2017-02-16
lang: pt
ref: php-write-up-shx-3-2
comments: true
---

Este é mais um Write Up do SHX 3 do Shellter Labs, continuando com o tema Segurança com PHP

Descrição do problema:

> "We know that flag is in the source code. Can you read it?" (Nós sabemos que a flag está no código-fonte. Você pode lê-lo?)

Também é oferecido um link para o site a ser explorado, e ao acessar o site, vemos isto:

<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Pagina inicial</figcaption>
</figure>

### Solução
Ao selecionar uma das páginas do menu, a url fica nesse formato: `http://lab.shellterlabs.com:32841/index.php?page=home.html` e logo percebe-se que trata-se de uma vulnerabilidade muito comum em aplicações web em PHP, que é o [LFI (Local File Inclusion)](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion). Para ter certeza que era uma falha de LFI, tentei incluir o arquivo /etc/passwd do Linux (e Unix em geral), que é onde o sistema operacional armazena as informações sobre cada um dos usuários que pode entrar no sistema, para incluir alterei a url para `http://lab.shellterlabs.com:32841/index.php?page=../../../../etc/passwd` e todo o conteúdo do arquivo *passwd* foi mostrado na tela. 

Apesar de saber qual a vulnerabilidade da página, ainda não temos o que importa, que é flag, e como a descrição do problema diz que a flag está no código fonte, mas se incluirmos o arquivo index.php na página através de `http://lab.shellterlabs.com:32841/index.php?page=index.php` ainda não teremos o código php da página, pois este será incluído e interpretado (e provavelmente entrará em loop infinito de inclusão de arquivo).

Então, para ter acesso ao código fonte podemos usar um [wrapper php](http://php.net/manual/pt_BR/wrappers.php.php) que nos possibilita ter acesso ao código. Desta forma:

<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Código fonte convertido para base64 atráves de: http://lab.shellterlabs.com:32841/index.php?page=php://filter/convert.base64-encode/resource=index.php</figcaption>
</figure>

Após isto, ao decodificar o código em base64, é possível ver e flag no código-fonte.


<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Flag: shellter{oh_no_you_exploited_my_lfi!}</figcaption>
</figure>

Moral da história: Nunca inclua arquivos diretamente de entradas de usuários.