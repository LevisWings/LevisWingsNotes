# SSRF - Source Code Review (Level I)

## PHP

### file\_get\_contents() <a href="#1-php-file-get-contents" id="1-php-file-get-contents"></a>

```php
<?php
    if (isset($_POST['url'])) 
    { 
        $content = file_get_contents($_POST['url']); 
        $filename = './uploads/'.rand().'image.jpg'; 
        file_put_contents($filename, $content); 
        echo $_POST['url'].""; 
        $img = "<img src=\"".$filename."\"/>"; 
    } 
    echo $img; 
?>
```

This PHP code obtains the data requested by a user (in this case, an **image**) by the POST parameter "**url**", using the PHP function `file_get_contents` and then stores it in a file with a randomly generated name (`rand()` function) on disk. Finally, the HTML **img** attribute is used to display the image to the user.

### fsockopen() <a href="#2-php-fsockopen-function" id="2-php-fsockopen-function"></a>

```php
<?php 
    function GetFile($host,$port,$link) 
    { 
        $fp = fsockopen($host, intval($port), $errno, $errstr, 30); 
        if (!$fp) { 
            echo "$errstr (error number $errno) \n"; 
        } else { 
            $out = "GET $link HTTP/1.1\r\n"; 
            $out .= "Host: $host\r\n"; 
            $out .= "Connection: Close\r\n\r\n"; 
            $out .= "\r\n"; 
            fwrite($fp, $out); 
            $contents=''; 
            while (!feof($fp)) { 
                $contents.= fgets($fp, 1024); 
        } 
        fclose($fp); 
        return $contents; 
        } 
    }
?>
```

In this case, the implementation obtains the data requested by a user (any file) using the PHP function fsockopen(). This function establishes a TCP connection to a socket on the server and performs a raw data transfer (also known as RAW).

### curl\_exec() <a href="#3-php-curl-exec-function" id="3-php-curl-exec-function"></a>

```php
<?php 
    if (isset($_POST['url']))
    {
        $link = $_POST['url'];
        $curlobj = curl_init();
        curl_setopt($curlobj, CURLOPT_POST, 0);
        curl_setopt($curlobj,CURLOPT_URL,$link);
        curl_setopt($curlobj, CURLOPT_RETURNTRANSFER, 1);
        $result=curl_exec($curlobj);
        curl_close($curlobj);

        $filename = './uploads/'.rand().'.pdf';
        file_put_contents($filename, $result); 
        echo $result;
    }
?>
```

This implementation fetches the data using cURL via PHP using the `curl_exec()` function. The data is downloaded and stored on disk under the **uploads directory**, where the file name will be a **random number** with the **.pdf** file extension.

## Javascript

### Node

```javascript
const express = require('express');
const axios = require('axios');
const app = express();
const port = 2000;

app.get('/api/upload', async function (req, res) {
  const url = req.body.imgUrl;
  const imgReq = await axios.get(url);
  avatar.update(imgReq.data);
  res.send(imgReq.data);
});
```

Explanation:

* `const url = req.body.imgUrl;`:  The user input in the req (imgUrl) parameters is stored in the url variable directly without validation.
* `const imgReq = await axios.get(url);`: The server sends a GET request to the URL entered by the user.
* `res.send(imgReq.data);`: The result of the GET request is sent directly to the user.
