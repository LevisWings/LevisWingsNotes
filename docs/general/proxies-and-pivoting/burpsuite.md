# Burpsuite

## Features

{% tabs %}
{% tab title="Comparer" %}
### Features

* Compare whether an HTML page changed at all.
* Ideal for Python scripts where we have to assimilate a query or request to Python code.

### Requirements

* Have the file to upload
* Have a Python script that is transmitted by Burpsuite with the file.

### Usage

* Intercept On -> Upload the file manually -> Right click -> Send to repeater
* Intercept On -> Execute Python script -> Right click -> Send to repeater
* Go to the Repeater tab
* In tab 1 -> Right click -> Send to comparer
* In tab 2 -> Right click -> Send to comparer
* Go to the Comparer tab
* In the lower left corner, click on Words (to compare at word level)
* The highlighted ones are the ones that changed on both sides.
{% endtab %}

{% tab title="Repeater" %}
Repeater is used to repeat requests that we want it to do.

### Common uses

If you uploaded a reverse shell to the web server and after a while it gets deleted (or you delete it) do I have to do the whole procedure again? No, simply in HTTP History, you look for the request that you made to upload the reverse shell, and when you find it, press Ctrl + R to take it to the Repeater, and in that tab, if everything is in order, click on Go and that's it.
{% endtab %}

{% tab title="Intruder" %}
### Types of payloads

#### Simple List

We provide a dictionary.&#x20;

#### Recursive grep

We extract a text using regular expressions. This can be useful when processing a CSRF token, which is usually found in HTML code. For this case, what to do is:

* Grep - Extract → Add → Select the CSRF token to create the regular expression → Ok → Select Payload of type "Recursive grep" with the grep we created.
{% endtab %}
{% endtabs %}

## Tricks

### Intercept with Local Port Forwarding

If we are doing a Port Forwarding to, for example, a web service that is running on an internal machine, when we want to do an Intercept to our [localhost](http://localhost) through the port we have set, it will not work. For that we are going to have to create another proxy in Burpsuite.

#### Steps to configure

We are going to add a new proxy:

![](../../.gitbook/assets/proxy\_options1.png)

We add a port that we are not using (in this case we use 4646):

![](../../.gitbook/assets/proxy\_options2.png)

And then we specify where we want it to redirect traffic to. As in this case, we are doing a Port Forwarding, we have to point to the localhost (127.0.0.1) by the port that we have indicated at the time of doing the Port Forwarding.

![](../../.gitbook/assets/proxy\_options3.png)

Then we click on Ok and that's it. In this case, if we point to localhost:4646, we would be pointing to the Local Port Forwarding we did (localhost:1234).

### Intercept Metasploit

If we want to debug a metasploit web module, sometimes we will encounter modules that do not support proxies. However, if we configure the RHOST and RPORT to a Burpsuite proxy that redirects to the web service, it will work.

#### Steps to configure

We add a new proxy in Burp:

![](../../.gitbook/assets/intercept\_metasploit1.png)

We add a port that we are not using (in this case we use 4646):

![](../../.gitbook/assets/intercept\_metasploit2.png)

And then we specify where we want it to redirect traffic to. As in this case, if it is a web server, then we put its IP and port 80 (example with 10.10.10.10.10:80):

![](../../.gitbook/assets/intercept\_metasploit3.png)

And finally, we set the RHOST and RPORT values:

```bash
set RHOSTS 127.0.0.1
set RPORT 4646
```

### Ignore redirects

Sometimes, it can be that with Fuzzing we find a file but then it redirects us to the login of the page. What we can do in these cases is to try to bypass it by telling the Response Header to convert the 301 or 302 Found to 200 OK and see if it shows us a different content than the login.

![](../../.gitbook/assets/ignore\_redirects1.png)

Then we set the values:

![](../../.gitbook/assets/ignore\_redirects2.png)

### Paste a file (POST)

When there is a POST request, and it has a functionality to upload content to a file, we can try to replace that content with a RAW file. To do this we right click and use the `Paste from file option`:

![](../../.gitbook/assets/Paste\_from\_file\_burp.png)

### Match and replace

Sometimes, by altering a value of a header, body or parameter (either request or response), we can obtain new functionalities or visualize a resource that we are not allowed to access as a normal user. But, to achieve this, we must always alter a value. That is why the `Match and Replace` option will allow us to set the value we want for a certain part.

#### Example with a Cookie (Request Header)

Match and Replace → Add:

![](../../.gitbook/assets/match\_and\_replace.png)

We select the **Type** (where that part we want to replace is located) and do the **Match** and **Replace**. We can also use regular expressions. We give **Ok** and that's it.

### Proxy with Python

We may sometimes have problems tunneling Burpsuite traffic with Python (specifying the proxy 127.0.0.1:8080). In that case, we will have to set a different proxy.

#### Method

In the Python code, we put:

```bash
burp = { 'http' : 'http://127.0.0.1:4646' }
```

In Burpsuite, we add another listener:

* Binding tab → 4646

![](../../.gitbook/assets/proxy\_listener2.png)

Tab Request handgling → Web IP and Port.

![](../../.gitbook/assets/proxy\_listener1.png)

![](../../.gitbook/assets/proxy\_listerner3.png)

Now we go to the Match and Replace tab and put:

* Requests header
* Match: `Host: <IP>`
* Replace: `Host: <HOST>`

### Use SOCKS Proxy

Example with SOCKS proxy with **chisel, SSH Dynamic**, etc.:

![Tab: User options → Connections → SOCKS Proxy](../../.gitbook/assets/socks\_proxy.png)

#### Use multiple SOCKS proxies

Sometimes we can have multiple SOCKS proxies to reach a certain service/host. Burpsuite only allows the use of one SOCKS proxy. If we want to use several, we can run Burpsuite over proxychains:

```bash
# proxychains.conf:
socks5 127.0.0.1 1080
socks5 127.0.0.1 2080

# Burpsuite over proxychains:
proxychains -q burpsuite
```

### Add Custom Header

Proxy → Options → Match an Replace → Add:

![](../../.gitbook/assets/add\_custom\_header.png)

### Line breaks

It is possible that on Windows systems, web requests, especially in the headers, may need to have special characters at the end of a line, which is `\r\n`. Here is an example:

![](../../.gitbook/assets/lines\_break.png)

## Troubleshooting

### Problem 1

* Burpsuite needs us to run Java on the latest version (today, July 10, 2021, is the 16th). For that, we can change the java version in Arch Linux with the following command:

```bash
archlinux-java status
archlinux-java set <NAME>
```

### Problem 2

* Sometimes, Burpsuite will force us to use the `--illegal-access=permit` parameter in the Java command. For that, let's edit the `/usr/bin/burpsuite` file and add the indicated parameter to it:

```bash
#!/bin/sh
exec $JAVA_HOME/bin/java --illegal-access=permit -jar /usr/share/burpsuite/burpsuite.jar "$@"
```
