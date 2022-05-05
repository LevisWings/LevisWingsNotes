# SSRF - Source Code Review

## PHP

### file\_get\_contents() <a href="#1-php-file-get-contents" id="1-php-file-get-contents"></a>

```php
<?php
    if (isset($_POST['url'])) 
    { 
        $content = file_get_contents($_POST['url']); 
        $filename = './images/'.rand().'img1.jpg'; 
        file_put_contents($filename, $content); 
        echo $_POST['url'].""; 
        $img = "<img src=\"".$filename."\"/>"; 
    } 
    echo $img; 
?>
```

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

        $filename = './curled/'.rand().'.txt';
        file_put_contents($filename, $result); 
        echo $result;
    }
?>
```

## Javascript

### Node (Level 1)

```javascript
const express = require('express');
const axios = require('axios');
const app = express();
const port = 3000;

app.get('/user/image', async function (req, res) {
  const imgUrl = req.body.imgUrl;
  const imageReq = await axios.get(imgUrl);
  user.updateProfileImage(imageReq.data);
  res.send(imageReq.data);
});
```

### Node (Level 2)

Work in Progresss
