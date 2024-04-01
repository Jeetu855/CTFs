
In page source we can see that 

```js
document.addEventListener('DOMContentLoaded', function() {
            var count = parseInt(localStorage.getItem('count')) || 0;
            var cookieImage = document.getElementById('cookieImage');
            var display = document.getElementById('clickCount');
            display.textContent = count;
            cookieImage.addEventListener('click', function() {
                count++;
                display.textContent = count;
                localStorage.setItem('count', count);
                if (count >= 10000000) {
                    fetch('/click', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded'
                        },
                        body: 'count=' + count
                    })
                    .then(response => response.json())
                    .then(data => {
                        alert(data.flag);
                    });
                }
            });
        });
```

So we need to set the localStorage variable named `count` value to more than `10000000`

To do this, go into the console and run
```js
localStorage.setItem("count",10000000)
```

Reload the page and click again once more and we get the flag

flag : utflag{y0u_cl1ck_pr3tty_f4st}