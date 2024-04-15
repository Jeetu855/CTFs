On the web page it is written that

"We are so secure that we only allow requests from our own origin to access secret data. Talk about some serious security!"

![alt text](Attachments/1.png)

In burp, capture the request, send it to repeater and add the HTTP Origin header

```http
GET / HTTP/1.1

Host: 3.23.56.243:9003

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Connection: close

Upgrade-Insecure-Requests: 1

Origin: https://texsaw2024.com
```

And we get the flag

flag : texsaw{s7t_y0ur_or7g7n}
