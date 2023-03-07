## JS

首先：JS并不能获取到MAC地址，曾经可以利用ie的activeX来实现，现在不行了

The MAC address, by TCP/IP standards, is never communicated outside of the local-area network to which it pertains — routers beyond that LAN don't even *get* the information you're trying to record.

There are many other ways to try and identify unique visitors, including matching the user-agent's details in addition to the IP, serving cookies as part of your response, etc… it is, after all, a core functionality in the field of "web analytics".

MAC addresses are simply not part of the gamut of techniques that it makes sense to utilize for it!



Sorry but sending MAC address isn't part of the HTTP. However, you can use cookie to identify different users. Any backend language will do (add cookie in the server side). You can set the cookie in the client side using JavaScript too.

[How to get client MAC address by a access on a website? - Stack Overflow](https://stackoverflow.com/questions/3454858/how-to-get-client-mac-address-by-a-access-on-a-website)



但是抛开JS的话，绝大部分后端语言都可以实现对MAC地址的获取

## JAVA