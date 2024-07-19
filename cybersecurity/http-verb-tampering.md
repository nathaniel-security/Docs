# HTTP Verb Tampering

* &#x20;can be exploited by sending malicious requests using unexpected methods, which may lead to bypassing the web application's authorization mechanism or even bypassing its security controls against other web attacks

### Insecure Coding

```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

### Insecure Configurations

```
<Limit GET POST>
    Require valid-user
</Limit>
```

### Attack

* Crafting custom HTTP requests

```
[METHOD] /[index.htm] HTTP/1.1
host: [www.example.com]
```

```
OPTIONS /index.html HTTP/1.1
host: www.example.com
```

```
GET /index.html HTTP/1.1
host: www.example.com
```

```
HEAD /index.html HTTP/1.1
host: www.example.com
```

```
POST /index.html HTTP/1.1
host: www.example.com
```

```
PUT /index.html HTTP/1.1
host: www.example.com
```

```
DELETE /index.html HTTP/1.1
host: www.example.com
```

```
TRACE /index.html HTTP/1.1
host: www.example.com
```

```
CONNECT /index.html HTTP/1.1
host: www.example.com
```

### Automated HTTP Verb Tampering Testing

```
#!/bin/bash

for webservmethod in GET POST PUT TRACE CONNECT OPTIONS PROPFIND;

do
printf "$webservmethod " ;
printf "$webservmethod / HTTP/1.1\nHost: $1\n\n" | nc -q 1 $1 80 | grep "HTTP/1.1"

done
```

## Reference

* https://owasp.org/www-project-web-security-testing-guide/v41/4-Web\_Application\_Security\_Testing/07-Input\_Validation\_Testing/03-Testing\_for\_HTTP\_Verb\_Tampering
