## What is LFI
Local File Inclusion/ Path Traversal enables an attacker to read arbitrary files on the server that is running an application
It's something like requesting the server to get internal data which it's not configured to provide.

## Why does LFI works

Imagine a service is displaying images ```<img src="/loadImage?filename=218.png">```  
Here the loadImage is takeing filename parameter which is actually loading image from lets say (/var/www/html/images/218.png)  
Now we can actually try to travese to the previous directory using (..) or something like (/var/www/html/images/../../../../etc/hosts), We are going one step up each time so it should be `/etc/host`
However in case of windows we can do something like `..\..\..\..\windows\win.ini`

Lab 1: File path traversal, simple case

Traversing one directory at a time solved the lab
1. `../etc/shadow`
2. `../../etc/shadow`
3. `../../../etc/shadow`
And we solved the lab
You can bruteforce it as well


Lab 2: File path traversal, traversal sequences blocked with absolute path bypass
Bypass worked you can just directly request the internal file `/etc/passwd`

Lab 3: File path traversal, traversal sequences stripped non-recursively
Bypass worked as well we send something like `....//....//....//etc/passwd` and boom it worked.

Lab 4: File path traversal, traversal sequences stripped with superfluous URL-decode
Bypass worked you can send something like `..%252f..%252f..%252fetc/passwd`

Lab 5: Lab: File path traversal, validation of start of path
Bypass worked `/var/www/images/../../../etc/passwd`

Lab 6: File path traversal, validation of file extension with null byte bypass
Bypass worked `../../../etc/passwd%00.png`

### Some methods to bypass restrictions 
1. You can just directly fetch the file `/etc/shadow` sometimes it works obv.
2. You can use nested travel sequences, such as `....//` or `....\/`. These reverts into simple traversal squences when the inner sequence is stripped
How they work:

    Let’s say there's a filter that blocks ../ exactly.

    If an attacker sends ....//, and the filter removes or ignores extra . or /:

        ....// can become ../ after being sanitized or interpreted by the backend.

    while ....\/, which is a mix of forward and backslashes, useful against systems that use one over the other (Linux: /, Windows: \).
3. Most of the time the `multipart/form-data` requests, sent from the web server may strip the directory traversal payload in that case we could do url encoding or even double encoding.
  * ../ Becomes %2e%2e%2f and %252e%252e%252f respectively 
4. Providing base path as well for example
  * Lets say an image is hosted over /var/www/images the /var/ww/image/../../../etc/passwd should work 
5. Application may want extension to verify the integirty however you can something like, /etc/passwd%00.png `%00.png` which will be terminated as it's a null byte 
