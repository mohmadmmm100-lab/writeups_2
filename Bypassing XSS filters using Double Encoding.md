#xss #input
# summary
- try double and triple URL encoding .
- try encoding .
# explanation 
While going through some of the targets and testing input fields (like search boxes), I got an interesting input field, I just entered the usual input

![](https://hacklido.com/assets/files/2022-11-18/1668763154-226678-1.png)

![](https://hacklido.com/assets/files/2022-11-18/1668763154-238784-2.png)

and checked the source code.

![](https://hacklido.com/assets/files/2022-11-18/1668763154-271934-3.png)

Then I added a single quote but it filtered the input and replaced it with `hello1 &amp;` in some places and with ‘`&`’ in our target fields.

![](https://hacklido.com/assets/files/2022-11-18/1668763154-301874-4.png)

I tried URL encoding there, Then also got the same output which means it decodes the input.

So I used Double Encoding.  
By using double encoding it’s possible to bypass security filters that only decode user input once.  
The second decoding process is executed by the backend platform or modules that properly handle encoded data, but don’t have the corresponding security checks in place.

![](https://hacklido.com/assets/files/2022-11-18/1668763154-332051-5.png)

It works.  
Then our basic payload `'><script>alert(1)</script>` with double encoding tried.

`%2527%253E%253Cscript%253Ealert%25281%2529%253C%252Fscript%253E`

But it created an error

![](https://hacklido.com/assets/files/2022-11-18/1668763154-349072-6.png)

I searched for attributes of input tag to exploit using it.   
onfocus : The onfocus event occurs when an element gets focus.

`' onfocus='alert(1)'`

`%2527%2520onfocus%253D%2527alert%25281%2529%2527%2520`

![](https://hacklido.com/assets/files/2022-11-18/1668763154-367420-7.png)  
I clicked on the search bar, and the popup alert appeared.  
![](https://hacklido.com/assets/files/2022-11-18/1668763154-404929-8.png)  
But I thought of modifying it a little bit with autofocus which makes the text field automatically get focused upon page load and creates the popup alert while visiting the page itself.

`' onfocus='alert(1)' autofocus='`

`%2527%2520onfocus%253D%2527alert%25281%2529%2527%2520autofocus%253D%2527`

![](https://hacklido.com/assets/files/2022-11-18/1668763154-442961-9.png)

![](https://hacklido.com/assets/files/2022-11-18/1668763154-457905-10.png)

Yeah. It worked …

You can also use payloads like

`' onmouseover='alert(1)'`  
`%2527%2520onmouseover%253D%2527alert%25281%2529%2527%2520`
# link
https://hacklido.com/blog/87-reflected-xss-using-double-encoding