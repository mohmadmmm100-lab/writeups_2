#xss #file_upload #svg
# summary
- try to bypass file type
- understand app logic and url
### **Goal before approaching the program**

To find a one-click exploit (XSS or SSRF)

### **Approach**

Found a target that has many features which included Discussion, Discovery, Mixtapes, Shorts, Activity and what not. I went ahead with looking at user dashboard.

> Why would I look for xss at a user dashboard where only I am the visitor?

Nice Question! If I found XSS there then it would be considered self XSS. which has no impact. It would be a challenge to convert self XSS into a valid one.

One parameter that could have been shared outside the dashboard was the `profile picture url`. So I need to find a way to upload malicious file instead of a jpeg.

After doing some research, I found out that svg is considered as an image and it also allows javascript to execute. [_click here for svg_xss demo_](https://thin-little-clerk.glitch.me/)


If you check the source code of this page you will find that there is a script tag within this svg dom

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*-jqe0PFh_Vq0nyv9uuATZw.png)

Ok so we know now that we have to upload svg file instead of valid jpeg.

### **Bypassing Filter**

Only valid file that could have been uploaded was either jpeg or png file.

> How was the file being verified?

- They were creating an api POST request with only the image header being sent. If the header is valid then there was another POST request that was uploading the actual file. No validation on this second POST request.
- Here we can just send a valid png and in the second request we can replace the png contents with the svg payload.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*SwMkIVmn6Hcd8IOctJ9uaQ.png)

After successfully bypassing checks and uploading the image, there was no alert box waiting for me to close it 🙁. Later I found out that they were using ImageMagick to compress the image size.

`https://media.redacted.com/img/<image_name>?size=medium` Just had to remove the `size` parameter. which loaded the original svg image.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*yn6gSnsaOYWgQQlWJa5qbw.png)

> How is this impactful?

Instead of calling alert we can write XHR request that sends the cookie data to our server. Because it has very critical `ACCESS_TOKEN` we could possibly take over someones account. Just send the user the link and wait for them to click
# link
https://prashantbhatkal2000.medium.com/svg-based-stored-xss-ee6e9b240dee