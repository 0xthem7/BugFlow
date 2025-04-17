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


### Some methods to bypass restrictions 
1. You can just directly fetch the file `/etc/shadow` sometimes it works obv.
2. You can use nested travel sequences, such as `....//` or `....\/`. These reverts into simple traversal squences when the inner sequence is stripped
How they work:

    Let’s say there's a filter that blocks ../ exactly.

    If an attacker sends ....//, and the filter removes or ignores extra . or /:

        ....// can become ../ after being sanitized or interpreted by the backend.

    while ....\/, which is a mix of forward and backslashes, useful against systems that use one over the other (Linux: /, Windows: \).
