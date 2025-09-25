#xss #file_upload #svg 
# summary
- xss using xml in svg file upload or any uploaded xml
- check patched vulnerabilities and compare changes
# explanation

![](https://miro.medium.com/v2/resize:fit:700/1*WlJFPYym3kVg7sI6u9Y-jA.png)

I found xss via file upload here i uploaded svg file which stored in google cloud and reflected with xss

1) login at app.xyz.com  
2) go to online store >> settings >> Email Notification >> Email design  
3) click edit  
4) Upload the file with svg payload  
5) save the file and open image in new tab

But it’s marked as duplicate

Next day I observed that bug got patched and we can’t do xss anymore.

## Get Jay Sharma’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

But after I saw URL changed

> https://images.xyz.com/s/cdn/v1.0/i/m?url=https%3A%2F%2Fstorage.googleapis.com%2Fproduction-xyz-v1-3%2F823%2F1134823%2FUc1iyTw%2Fe843c14e4a5f46c6898aeb3133ea1a30&methods=convert%2Cpng

After URL decoding

https://images.xyz.com/s/cdn/v1.0/i/m?url=https://storage.googleapis.com/production-xyz-v1-3/823/1134823/Uc1iyTw/e843c14e4a5f42c6898aeb3133ea1a30&methods=convert,png

What if I remove the convert part from URL.

Oh we get pop up again

![](https://miro.medium.com/v2/resize:fit:700/1*vXQOSeXU4F-Nrs5d-0RKAA.png)
# extra
## 🖼️ What’s special about SVG?

- **SVG (Scalable Vector Graphics)** is an image format, but unlike PNG or JPG, it’s actually **XML (text-based)**.
- This means it can contain **markup and scripts** — like `<script>`, `<iframe>`, or event handlers (`onload`, `onclick`).
- If a web app allows **uploading SVGs** and then displays them **inline** in the browser (`<img src="...">` vs embedding the XML directly), it can lead to **XSS**.

---

## ⚡ Example payload

A malicious SVG file might look like this:

`<?xml version="1.0" standalone="yes"?> <svg xmlns="http://www.w3.org/2000/svg" onload="alert('XSS via SVG!')">     <circle r="50" cx="50" cy="50" fill="red"/> </svg>`

- When uploaded and rendered **inline**, the browser executes the `onload` handler → `alert()` pops.

---

### 🔹 Why this works

1. **MIME-type confusion (Content-Type mismatch)**
    
    - The server stores your file and serves it back with a **browser-interpretable MIME type** (e.g., `text/html`, `image/svg+xml`) instead of `audio/mpeg`.
        
    - If the browser interprets your "mp3" file as text/HTML, any `<script>` or event attributes inside can execute.
        
2. **File extension bypass**
    
    - Some applications only block `.html` / `.js` but allow `.mp3` or `.svg`.
        
    - If they don’t verify MIME type and just trust extension → XSS.
        
3. **Direct access**
    
    - If you can upload a file and then access it via a public URL (like `/uploads/file.mp3`), and the server mislabels the MIME type, the browser may parse your payload as **HTML or SVG**, executing XSS.
        

---

### 🔹 Example attack

Suppose the app blocks `.html` uploads but allows `.mp3`. You rename a malicious file:

**evil.mp3**

`<script>alert(document.domain)</script>`

If the server saves it and serves it with:

`Content-Type: text/html`

→ when someone visits `https://target.com/uploads/evil.mp3`, the script executes.

---

### 🔹 File types commonly abused for XSS

- **.svg** → Supports inline JavaScript and event handlers (`onload`, `onclick`).
- **.xml / .xsl** → Can contain script.
- **.txt / .json** → If served as `text/html`, the payload executes.
- **.mp3 / .jpg / .png** → If not validated and browser sees `<script>` inside (rare, but possible if server misconfigures content-type).
- **.pdf** → Can execute JS in some cases (old Adobe Reader, not modern browsers).
# link
https://sharmajijvs.medium.com/xss-via-file-upload-a2bcc1e5d7f7