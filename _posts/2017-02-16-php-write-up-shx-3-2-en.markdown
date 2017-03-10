---
layout: post
title:  "Write up SHX 3 (PHP Security) - Quantum Photos"
date:   2017-02-16
lang: en
ref: php-write-up-shx-3-2
comments: true
---

This is another Write Up of SHX 3 from Shellter Labs, continuing with the theme Security with PHP

Problem description:

> "We know that flag is in the source code. Can you read it?"

It also offer a link to the site to be explored, and when you access the site, we see this:

<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Home Page</figcaption>
</figure>

### Solution
When I selected one of the menu pages, I saw the url is in this format: `http://lab.shellterlabs.com:32841/index.php?page=home.html` and I soon realized that this is a very common vulnerability in Web applications in PHP, which is [LFI (Local File Inclusion)](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion). To make sure it was an LFI flaw, I tried to include the Linux /etc/passwd file (and Unix in general), which is where the operating system stores the information about each of the users that can enter the system, to include I changed the url for `http://lab.shellterlabs.com:32841/index.php?page=../../../etc/passwd` and the entire contents of the *passwd* file was shown in screen.

Apesar de saber qual a vulnerabilidade da página, ainda não temos o que importa, que é flag, e como a descrição do problema diz que a flag está no código fonte, mas se incluirmos o arquivo index.php na página através de `http://lab.shellterlabs.com:32841/index.php?page=index.php` ainda não teremos o código php da página, pois este será incluído e interpretado (e provavelmente entrará em loop infinito de inclusão de arquivo).

So, to get access to the source code we can use a [php wrapper](http://php.net/manual/pt_BR/wrappers.php.php) that allows us to access the code. This way:

<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Source code converted to base64 via: http://lab.shellterlabs.com:32841/index.php?page=php://filter/convert.base64-encode/resource=index.php</figcaption>
</figure>

After that, when decoding the code in base64, it is possible to see and flag in the source code.

<figure>
	<img src="{{ '/assets/img/quantum_photos_screenshot3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Flag: shellter{oh_no_you_exploited_my_lfi!}</figcaption>
</figure>

Moral of the story: Never include files directly from user inputs.