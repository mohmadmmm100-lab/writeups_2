#external #xss 
# summary
- use intruder with payload list and check for length 
- external program hunt
- check new features
# report

First, I tried to find the program through Google Dork. The dork I used is: **intext:Cryptocurrency Exchange intext:Bug bounty**

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*ZPOw6GhEnjkkkQWywZXa9A.png)

Google dork

After finding the program, I checked the scope of the program, but unfortunately there was only one domain, and that was the main domain.

So I said to myself, “Let’s hunt on this and go deep into the target and try to find at least one valid bug.”

I gave up after about 2–3 days because I hadn’t found anything. After 2 weeks, I again visited the site, and I see that it has implemented some new features. New features mean new bugs. let’s hack


I went through all of the features and captured all of the requests with BurpSuite, and one feature caught my eye, where you can translate the word into different languages.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*Ypv_RFlHzzHlmuo0x1TScQ.png)

Translation helper

An input box, let’s try for XSS. After that, I type **“>img src=x onerror=alert(1)>** and then looking at the source code, the value is well sanitised.

![](https://miro.medium.com/v2/resize:fit:369/1*IJuhZtrSDjJV3tr3GHivow.png)

well senitize

Now I sent the request to the repeater tab and tried some XSS bypasses but didn’t find anything useful. And lastly, I sent the request to the intruder tab, and I fuzzed with some XSS payload lists, and after finishing the intruder attack, when clicking on the length tab, I got a few payloads of bigger length.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*MOSLaSc3Js8TVpVF3gVFnA.png)

Possible XSS

The XSS payload is successfully fired when you click on show response.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*6Cn_OzfaJZG2e77KNDCT2w.png)

xss payload fire

XSS polygot was the payload that was executed :

> ==**javascript:/* →</title></style></textarea></script></xmp><details/open/ontoggle=’+/`/+/”/+/onmouseover=1/+/[*/[]/+alert(/@PortSwiggerRes/)//’>**==

And after reporting the bug, they said that your bug is eligible for a $200 bounty, and I received the bounty in crypto.
# link
https://kingcoolvikas.medium.com/how-i-found-my-first-xss-on-a-bug-bounty-program-c41107617ce1