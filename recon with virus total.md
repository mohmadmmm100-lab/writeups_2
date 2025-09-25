#recon
# summary
- recon using good web tools
# report
## 🧠 About the Target

The target application is a collaborative chat platform that allows users to:

- Communicate via private or group messages
- Create and manage groups
- Add or remove members
- Share files and collaborate in real-time

Each chat group can be accessed through a unique URL containing an alphanumeric token, such as:

https://example.com/inbox/78c8bc2b-25a3-4492-8c8e-61870949?ref=calendar

When this URL is visited, it grants **anonymous access** to the entire group — including chat history, shared files, and member messages — **without requiring authentication**, provided a certain setting is enabled by the group creator.

The website includes a feature labeled **“Available by link”**, which, when enabled, allows **anyone with the group URL** to access the chat room **without authentication**. This grants full visibility into the group’s **entire chat history, shared files, and ongoing conversations**, posing a significant security and privacy risk if the URL is exposed.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*okY5CnaoFTtLm8FbG6WwCQ.png)

If this is turned on anyone can access the group link

## 🔍 Reconnaissance & Discovery

Once I understood this functionality, I began hunting for exposed group URLs across the internet using multiple open-source intelligence (OSINT) techniques such as:

- Google and Bing Dorking
- Wayback Machine URLs (`waybackurls`)
- URLScan.io
- VirusTotal

During my reconnaissance on **VirusTotal**, I struck gold.

## 🛠️ How to Use VirusTotal for Recon

1. **Create an account** on VirusTotal ([https://www.virustotal.com](https://www.virustotal.com/)).
2. Generate or locate your API key.
3. Use the following endpoint to fetch URLs associated with a specific domain:

==https://www.virustotal.com/vtapi/v2/domain/report?apikey=YOUR_API_KEY&domain=example.com==

4. Replace `YOUR_API_KEY` with your VirusTotal API key, and `example.com` with your target domain.

5. In the JSON response, look under the `undetected_urls` section.  
These are URLs that were fetched or scanned by VirusTotal but haven't been flagged as malicious—**often a goldmine for sensitive endpoints**.

## 💥 The Vulnerability

While reviewing the `undetected_urls`, I found several live group chat URLs belonging to different organizations. Anyone with these links could access group conversations, download shared files, and read sensitive discussions **without authentication**. This clearly qualified as an **Information Disclosure vulnerability**.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*4KGph9IfjIwDqu98Vk5GQQ.png)

undetected_url exposing groups urls

## 🧾 Reporting & Bounty

I responsibly reported the issue to the program via their vulnerability disclosure platform. The team acknowledged the bug, confirmed its severity, and rewarded me with a **monetary bounty** as a token of appreciation for my findings.
# link
https://kingcoolvikas.medium.com/how-i-earned-a-bounty-using-virustotal-recon-93024ee964ed