# Web Servers

### Python2/Python3

```bash
python -m SimpleHTTPServer 8080
python3 -m http.server 8080
```

### Ruby

```ruby
ruby -run -e httpd . -p 8080
```

### PHP

```bash
php -S 0.0.0.0:8080
```

### Socat

```bash
socat TCP-LISTEN:8080,reuseaddr,fork
```

### Busybox

```bash
busybox httpd -f -p 10000
```
