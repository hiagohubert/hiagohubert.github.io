---
layout: post
title:  "Write up SHX 3 (PHP Security) - Hello Photo"
date:   2017-02-20
lang: en
ref: php-write-up-shx-3-3
comments: true
---

This is the last Write Up about SHX 3 from Shellter Labs.

Problem description:

> Your friend asked you to verify that the new website that he is developing is secure. Can you help him?

It is also provided the link to the site to be explored, that has this appearance:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot1.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig1. - Home page</figcaption>
</figure>

Soon after opening the site, you can see that it is an image upload site, so let's understand how the upload process is being done, uploading any image:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot2.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig2. - Image upload</figcaption>
</figure>

After uploading the image the following page is displayed:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot3.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig3. - Upload de imagem bem sucedido</figcaption>
</figure>

As shown in Fig3, the images are saved in the *uploads* directory, and having this information, we can try uploading a PHP file to get a shell.

When trying to upload a file in PHP with a simple "Hello World", the application returns an error:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot4.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig4. - Unsuccessful file upload</figcaption>
</figure>

My first attempt to bypass this validation was to use the [Burp Suite](https://portswigger.net/burp/) to [change the Mime Type of the file at the request](https://null-byte.wonderhowto.com / How-to / bypass-file-upload-restrictions-using-burp-suite-0164148 /), but did not work, error persisted, and I realized that validation checked part of the file contents to see if it was an image, the solution was to inject a PHP code into the image content. I used GIMP to do this, but you can do this with Photoshop or even editing the file with VIM.

It was done as follows:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot5.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig5. - Injecting PHP code into the image through comments from this.</figcaption>
</figure>

After that, just save and modify the extension of the image to *.php* and upload it back into the system. And this time it works:

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot6.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig6. - PHP file validated as jpg/gif</figcaption>
</figure>


After that, we have a shell ready to use, now just look for the flag by the server's file system.

<figure>
	<img src="{{ '/assets/img/hello_photo_screenshot7.png' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Fig7. - Flag: shellter{b3_c4r3ful_w1th_upl04d5}</figcaption>
</figure>

For those who are curious, I downloaded the file *uploader.php* and this is the unsafe code:

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

In this specific case, a possible solution to prevent PHP files from being run in the uploads folder would be to create an .htaccess and add the code:

`
php_flag engine off
`

Of course this does not guarantee 100%, but it would solve this case.

If you have found an error, have something to add or a question, comment below!