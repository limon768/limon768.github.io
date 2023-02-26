---
title: Portswigger Lab JWT
date: 2023-02-26 09:45:47 +07:00
modified: 2023-02-26 09:45:47 +07:00
tags: [Portswigger, JWT , Writeup]
description: Write Up for Portswigger Lab JWT
---


![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/logo.png)

Lab Name → **JWT**

Lab Link → [https://portswigger.net/web-security/all-labs](https://portswigger.net/web-security/all-labs)

Description → All Labs of JWT from Portswigger.

For all the labs I used Burpsuite with JWT Editor and JSON Web Token extensions.

# Apprentice

## JWT authentication bypass via unverified signature

Let's login with the given credentials and intercept the request. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/1.png)
![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/2.png)

Our Extension can automatically detect JSON token. Let's change the user to administrator. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/3.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/4.png)

After forwarding the request we can see the username is changed. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/5.png)

Now let's repeat the process and each time the username needs to be changed. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/6.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/7.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/8.png)

NOTE → If you don't want to change your username every time, after intercepting the request send it to repeater. 

##  JWT authentication bypass via flawed signature verification

We can bypass just by changing `alg` parameter to none

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/9.png)

# Practitioner

## JWT authentication bypass via weak signing key

After intercepting the request we can brute force a weak signing key using Hashcat.

`hashcat -a 0 -m 16500 <jasonTOKEN> <wordlist>`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/10.png)
![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/11.png)

After finding the signing key we can sign out payload.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/12.png)

NOTE → When forwarding the request delete the automated added line

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/13.png)

## JWT authentication bypass via jwk header injection

Let's intercept the request.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/14.png)

Now create an RSA Key.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/15.png)
![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/16.png)

The extension will automatically add the RSA Key to the cookie. We have Embed JWK. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/17.png)

Now after forwarding the request we have successfully bypassed to Admin.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/18.png)

## JWT authentication bypass via jwk header injection

This time after creating the RSA key we are going to copy Public Key As JWK.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/19.png)

Next, we are going to add our public key to the exploit server body. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/20.png)

Now in our payload header, we are going to add the JKU server link and our KEY ID. And then we are going to sign our token with RSA. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/21.png)


## JWT authentication bypass via kid header path traversal

We can create a Symmetric Key and In "k" parameter we can add null in base64. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/22.png)

Next in our payload header, we set the path to `/dev/null` and then sign the key to bypass. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/23.png)

# Expert

## JWT authentication bypass via algorithm confusion

We can get the server public key by going `/jwks.json`.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/24.png)

Now with that public key, we can make our own RSA key. After clicking `New RSA Key` we can add the public key. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/25.png)

Let's also create a new Symmetric key.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/26.png)

Now lets copy the RSA key as a public PEM key. And then encode it with base64

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/27.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/28.png)

Now let's edit out Symmetric Key and add the base64 string in "k" parameter. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/29.png)

Now change the "alg" to `HS256` and sign the token to bypass.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/30.png)


## JWT authentication bypass via algorithm confusion with no exposed key

First, we need to get [rsa_sig2n](https://github.com/silentsignal/rsa_sign2n).

Burpsuite also made the task easy for us. We can use the docker version.

We need to get 2 tokens. We can login with our default credentials twice to get the tokens.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/31.png)

Next, we can use these keys to make our forged key.

`docker run --rm -it portswigger/sig2n <Token 1> <Token 2>`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/32.png)

Now lets try one of the tampered JWT token. 

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/33.png)

We get 200 OK. Now generate a new symmetric key and add the base64 string in "k" parameter.
![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/34.png)

Lastly set "alg" parameter to HS256 and sign the token to bypass.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/JWT/35.png)

