---
layout: post
title: Bypassing Samesite Cookie Restrictions with Method Override
subtitle: Neat Samesite Bypass Trick
cover-img: https://i.imgur.com/S3dhyUi.jpg
thumbnail-img: https://i.imgur.com/G5v7saj.png
share-img: https://i.imgur.com/S3dhyUi.jpg
tags: [Samesite, Cookie, Bug Bounty, Bypass]
---

# Bypassing Samesite Cookie Restrictions with Method Override

Browsers, in an attempt to mitigate CSRF (Cross-Site Request Forgery) attacks, have introduced the [SameSite cookie attribute](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#samesite-cookie-attribute).

By default, cookies are set to `SameSite=Lax``, which stops the cookie from being sent during cross-site requests, unless they are initiated by top-level navigations via GET requests. This effectively protects against CSRF attacks launched through POST requests.

However, a loophole exists. A number of web development frameworks support the override of HTTP methods at the server-side. An attacker can exploit this by altering a POST request to a GET, thereby bypassing the SameSite cookie restrictions and successfully performing CSRF attacks.

## How Method Override Works

Method Override is a functionality that allows the HTTP method processed by the server to be different from the one specified in the initial request. Here are some typical ways to override methods:

- Hidden form fields, such as `_method=PUT``
- Custom HTTP headers, for instance `X-HTTP-Method-Override: DELETE``
- Server-side middleware capable of parsing these overrides

These techniques are frequently used to emulate `PUT``, `PATCH``, and `DELETE`` requests in web forms, which by design, only support `GET`` and `POST`` natively.

## Bypassing Lax Restrictions with GET Requests 

An attacker can leverage Lax restrictions by triggering a cross-site GET request from the victim's browser:


```
<script>
  document.location = 'https://example.com/transfer?recipient=attacker&amount=1000';
</script>
```

As a result of top-level navigation, this script includes cookies.

## Leveraging Method Override Example

Frameworks like Symfony allow the method to be overridden using `_method` parameters:

```html
<form action="https://example.com/transfer" method="POST">
  <input type="hidden" name="_method" value="GET">
  <input type="hidden" name="recipient" value="attacker">
  <input type="hidden" name="amount" value="1000">  
</form>
```

The server is deceived into treating the `POST` request as a `GET` request.

## Frameworks With Built-In Support

Here is a summary of some popular web frameworks and how they allow method override:

| Framework | Parameter | Method Override Header | Notes |  
|:------|:------|:------|:------|
| Symfony | `_method` | `X-HTTP-Method-Override` | |
| Rails | `_method` | `X-HTTP-Method-Override` | |
| Laravel | `_method` | `X-HTTP-Method-Override` | |    
| CodeIgniter | `_method` | `X-HTTP-Method-Override` | |   
| CakePHP | `_method` | `X-HTTP-Method-Override` | |  
| Yii | `_method` | `X-HTTP-Method-Override` | |  
| Flask | - | `X-HTTP-Method-Override` | |
| Django | - | `X-HTTP-Method-Override` | |
| Angular | - | - | Depends on the HTTP library used |
| React | - | - | Depends on the HTTP library used |   
| Vue.js | - | - | Depends on the HTTP library used |
| Ember.js | `_method` | `X-HTTP-Method-Override` | |  
| Meteor | `_method` | - | |
| Express.js | `_method` | `X-HTTP-Method-Override` | Via middleware |
| Koa.js | `_method` | `X-HTTP-Method-Override` | Via middleware |  
| Next.js | - | - | Depends on the HTTP library used |     
| Spring MVC | - | `X-HTTP-Method-Override` | |
| ASP.NET Core | - | `X-HTTP-Method-Override` | With middleware support |    
| Phoenix | `_method` | `X-HTTP-Method-Override` | |  
| Bottle | `_method` | `X-HTTP-Method-Override` | |

## Crafting the Exploit

The attacker can craft the exploit to trigger the malicious `GET` request:

```html
<script>  
  document.location = "https://example.com/change-email?email=pwned@example.com&_method=POST";
</script>
```

When loaded by the unsuspecting victim, this script bypasses the Lax restrictions and performs the intended action on behalf of the victim.

In conclusion, while the SameSite cookie attribute has added a layer of defense against CSRF attacks, it's essential to be aware of its limitations and potential workarounds. Method override, in certain circumstances, could be exploited by attackers to bypass these protections, happy hacking!