#xss #ckeditor #cves
# summary
- check technologies .
- hunt on familiar technologies and make records for them .
- cves .
# explanation :
Let’s say the target is target.gov.in. I have already collected the domains during subdomain enumeration phase. While surfing different websites & seeing wappalyzer, one thing caught my eyes that the site it using rich text editor framework named “**CKEditor”. (Which is known to me before)**

![](https://miro.medium.com/v2/resize:fit:503/1*fsskTxh_tZnmJXumpeBkyg.png)

CKEditor

After seeing that, i remembered that i have already tested this “**CKEditor**” — web text editor before in another hunting. See about my writeup about it : [**XSS IN CKEditor — By Professor0xx01**](https://medium.com/@p.ra.dee.p_0xx01/found-multiple-bugs-xss-mitm-sec-misconf-in-an-educational-site-5a3804085da0)**.**

So, let’s make start a directory enumeration using **dirsearch** & see if my guesses are right or not ..??

dirsearch -u https://target.com -x 404,403

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*UfcNr99LSOMToO_nfpcmZw.png)

I got that endpoint but not the text editor accordingly………….!!



Then i moved to my another writeup ([**XSS IN CKEditor — By Professor0xx01**](https://medium.com/@p.ra.dee.p_0xx01/found-multiple-bugs-xss-mitm-sec-misconf-in-an-educational-site-5a3804085da0)**.)** & searched the endpoint as follows ,, to check wheather the CKEditor page actually exist or not…!!! And yayyyhhhhh………………… I got that juicy html page ………….!!!!!


The Juicy endpoint :

`https://<target>.gov.in/ckeditor/samples/plugins/htmlwriter/outputhtml.html` 

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*MLmxtCyhsUZFlo5pupW1Dg.png)

Now I quickly Inserted my XSS Payload to this page & Got the Alert () !!

`Xss<!--{cke_protected} --!><img src=x onerror=alert(`Professor0xx01`)> -->Attack`

**Steps To Reproduce :**

1. **First click on source ….**
2. **Give the malicious payload …..**
3. **Then click again on source !!**
4. **You will got that alert ()**.


![](https://miro.medium.com/v2/resize:fit:700/1*bXvtifipLmxB2A_y_UHdTQ.png)
# link
https://medium.com/@p.ra.dee.p_0xx01/how-i-found-xss-in-another-govt-site-nciipc-vdp-84d78c0319c2