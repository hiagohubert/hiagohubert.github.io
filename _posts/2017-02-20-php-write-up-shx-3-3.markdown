---
layout: post
title:  "Write up SHX 3 (PHP Security) - Hello Photo"
date:   2017-02-20
lang: pt
ref: php-write-up-shx-3-3
comments: true
---

Este é o último Write Up sobre o SHX 3 do Shellter Labs.

Descrição do problema:

> Seu amigo te pediu para verificar se o novo site que ele está desenvolvendo está seguro. Você pode ajudá-lo? 

Também é oferecido o link para o site a ser explorado, que tem essa cara:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Pagina inicial</figcaption>
</figure>

Logo ao abrir o site, é possível ver que se trata de um site de upload de imagens, então, vamos entender como está sendo feito o processo de upload, subindo uma imagem qualquer:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig2. - Upload de imagem</figcaption>
</figure>

Após fazer o upload da imagem é exibida a seguinte página:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig3. - Upload de imagem bem sucedido</figcaption>
</figure>

Conforme é exibido na Fig3, as imagens são salvas no diretório *uploads*, e tendo essa informação, podemos tentar subir um arquivo PHP para obter uma shell.

Ao tentar subir um arquivo em PHP com um simples "Hello World", a aplicação retorna um erro:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot4.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig4. - Upload de arquivo mal sucedido</figcaption>
</figure>

Minha primeira tentativa de burlar esta validação, foi utilizando o [Burp Suite](https://portswigger.net/burp/) para [alterar o Mime Type do arquivo na requisição](https://null-byte.wonderhowto.com/how-to/bypass-file-upload-restrictions-using-burp-suite-0164148/) , mas não funcionou, o erro persistiu, e percebi que a validação verificava parte do conteúdo do arquivo para saber se era uma imagem, então a solução foi injetar no conteúdo da imagem um código PHP. Eu utilizei o GIMP para fazer isto, mas você pode fazer isso com o Photoshop ou até mesmo editando o arquivo com o VIM.

Foi feito da seguinte forma:


<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot5.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig5. - Injetando código PHP na imagem através dos comentários desta.</figcaption>
</figure>

Após isto, basta salvar e modificar a extensão da imagem para *.php* e fazer o upload novamente no sistema. E dessa vez funciona:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot6.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig6. - Arquivo PHP validado como jpg/gif</figcaption>
</figure>


Feito isso, temos uma shell pronta para uso, agora é só procurar a flag pelo sistema de arquivo do servidor.

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot7.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig7. - Flag: shellter{b3_c4r3ful_w1th_upl04d5}</figcaption>
</figure>

Para quem tiver curiosidade, baixei o arquivo *uploader.php* e este é o código inseguro:

```php

<?php
$imageinfo = getimagesize($_FILES['uploadedfile']['tmp_name']);
if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg') {
    echo "Error: Please provide a valid image (GIF or JPEG).<br><br>Click <a href=\"./\">here</a> to go back.";
    exit;
}
$uploaddir = 'uploads/';
$temp = explode(".", basename($_FILES['uploadedfile']['name']));
$newfilename = round(microtime(true)) . '.' . end($temp);
$uploadfile = $uploaddir . basename($_FILES['uploadedfile']['name']);
//if (move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $uploadfile)) {
if (move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $uploaddir . $newfilename)) {
    echo "File <b>" . $newfilename . "</b> is valid, and was successfully uploaded to the uploads directory.<br><br>Click <a href=\"./\">here</a> to go back.";
} else {
    echo "Oops! Something went wrong.";
}
?>

```

Neste caso específico, uma solução possível para evitar que arquivos PHPs fossem executados na pasta de uploads seria criar um .htaccess e adicionar o código:

`
php_flag engine off
`

Lógico que isto não garante 100%, mas resolveria este caso.

Se você encontrou algum erro, tem algo a acrescentar ou alguma dúvida, comente abaixo!