# cURL CheatSheet

### Basic

```bash
curl -v "<URL>" # Verbose (Request and response headers, etc.)
curl -i "<URL>" # Show response headers
curl -k "<URL>" # Ignore certificate SSL.
curl -L "<URL>" # Follow redirect.
curl -x [protocol://]host[:port] "<URL>" # Use proxy (HTTP, SOCKS, etc.)
curl "<URL>" -o /dev/null -w "%{http_code}" # Suppres body response and show status code.
```

### Authentication

```bash
curl --user admin:password123 http://10.10.10.10/admin
curl http://admin:password123@10.10.10.10/admin
curl http://10.10.10.10/admin -H '<COOKIE>'
```

### URL encode

```bash
curl -s --get "<URL>" --data-urlencode "<DATA WITH PARAMETERS>" # GET
curl -s "<URL>" --data-urlencode "<DATA>" # POST
```

### POST (form-data)

```bash
curl -s -X POST "<URL>" -F 'add=1' -F 'image=@shell.php' -x http://127.0.0.1:8080
```

![](../../.gitbook/assets/post\_form\_data.png)

### cURL command to code

{% embed url="https://curlconverter.com" %}
