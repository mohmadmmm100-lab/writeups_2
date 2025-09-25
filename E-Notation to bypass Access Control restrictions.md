#idor #access_control
# summary
- use different id format and math function to exploit ( 123e0 , 123+1 and so on )
# write up
The company targeted in this bug bounty research is a large cap company with over 2 million customers worldwide. More specifically the host discussed in this article is a multi-user website with various functions, including discussions feature between the company employees and its users. These discussions are regarded as highly sensitive and include PII data which should be protected accordingly. During the investigation, a vulnerability was discovered that allowed unauthorized access to other users’ discussions through an access control bypass method and IDOR.

Sensitive details about the company and requests are redacted and the requests shown do not represent any real user.

The website’s discussion function utilized a user ID in the API request “_GET /api/v1/user/123/discussions_” Initially, the ID was protected against IDOR, returning a 403 error if the user lacked access to the entered ID. However, it was noted that entering IDs in unconventional formats, such as “**123a**” resulted in a 400 error with the message “error from discussions API.” Interestingly, querying a different API request with unconventional user ID format on the same host, such as “_GET /api/v1/information/users/_**_123a_**” returned the same result as the standard request “_GET /api/v1/information/users/_**_123_**” highlighting abnormal behavior in the firstly introduced **discussions** API.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*Kqi0BX7yG-aJ-XFN0U64_Q.png)

Normal request to fetch user’s discussions

Multiple attempts were made to exploit the discussions API user ID validation, revealing inconsistent responses to different ID formats.

- ”123aaa” —> 400 – ”Error from discussions api”
- ”123” —> 200 – ”Discussions for user 123: …”
- ”123.0” —> 200 – ”Discussions for user 123: …”
- ”123.2” —> 200 – ”Discussions for user 123: …”
- ”111.0” —> 403 – ”User has no access to this id”
- ”1234” —> 403 – ”User has no access to this id”
- ”123+1” —> 400 – ”Error from discussions api”
- ”123/1” —> 400 – ”Error from discussions api”
- ”123*1” —> 400 – ”Error from discussions api”
- ”123–0” —> 400 – ”Error from discussions api”

Despite attempts to manipulate IDs using arithmetic operations, they proved ineffective. However, it was observed that modifying the value so that it would result in correct numeric representation to the correct ID allowed successful requests.

As there are different ways of expressing numbers. This was the next point of focus. The expression could not be hex (123 = 0x7B) and it could not include any characters that are not “url safe” since it had to start with the correct user id. So… here comes the [scientific operation](https://en.wikipedia.org/wiki/Scientific_notation) called E-notation to the rescue. E-notation is normally used to represent really large or really small numbers that would be too long to conveniently represent in decimal form. For example 100 in E-notation would be represented as 10e1 or 1000e-1 or possibly as 10.0e1. There is a great [calculator](https://www.calculatorsoup.com/calculators/math/scientific-notation-converter.php) that was used during this case that helped a lot with finding the correct representation.

Since the correct expression that is valid to the access control validation, needed to start with the correct user ID. This needed to be taken into consideration. By incorporating the correct user ID with the E-notation into the request, the access control validation was successfully bypassed:

- “123e0” (123 in decimal) —> 200 – “Discussions for user 123: …”
- “123e1” (1230) —> 200 – “Discussions for user 1230: You should not see this…”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*5jmW9z1dWqqbtDEGYkqmIA.png)

Incorporating E-notation to the request allowed bypass of access control validation (user data in photo is fictive)

This method effectively circumvented access controls by including the necessary pieces for validation while “smuggling” additional values into the backend call. Validation only validated the first numeric characters inputted and the rest was disregarded if the representation was the correct numeric value (e.g. 123.0, 123.3, 123e3).

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*JtqinlVFVqjGgog2h6vu3w.png)

Discussions fetch flow that represents inconsistencies in the numeric representation

However, this approach has limitations, restricting traversal to only IDs higher than the user’s own ID by a factor of 10 (e.g., ID 123 could access discussions for users 1230 and above):

- “123e1” —> 1230
- “123.1e1” —> 1231
- “123.2e1” —> 1232

Furthermore, certain edge cases allowed access to users included in the requester’s ID:

- “123e-1” —> 12.3 —> user 12
- “123e-2” —> 1.23 —> user 1

This vulnerability was caused by the possibility to “smuggle” values into the backend calls to fetch user’s discussions. This was made possible by inconsistent handling of different numeric representations by the backends.


https://medium.com/@keizobugbounty/using-e-notation-to-bypass-access-control-restrictions-to-access-arbitrary-user-pii-discussions-1fa014b544d4
