Web application reconnaissance refers to the explorative data-gathering phase that generally occurs prior to hacking a web application.

The hacker should understand the purpose of the application from a functional perspective. Who are its users? How does the application generate revenue? For what purpose do users select the application over competitors? Who are the competitors? What functionality is found in the application?
Without deep understanding of the target application from a nontechnical perspective, it is actually difficult to determine what data and functionality matter.

we can use JSON formatting or YAML for reconnaissancing of target for example
```yaml
api_endpoints:
  sign_up:
    url: 'mywebsite.com/auth/sign_up'
    method: 'POST'
    shape:
      username:
        type: String
        required: true
        min: 6
        max: 18
      password:
        type: String
        required: true
        min: 6
        max: 32
      referralCode:
        type: String
        required: false
        min: 64
        max: 64
  sign_in:
    url: 'mywebsite.com/auth/sign_in'
    method: 'POST'
    shape:
      username:
        type: String
        required: true
        min: 6
        max: 18
      password:
        type: String
        required: true
        min: 6
        max: 32
  reset_password:
    url: 'mywebsite.com/auth/reset'
    method: 'POST'
    shape:
      username:
        type: String
        required: true
        min: 6
        max: 18
      password:
        type: String
        required: true
        min: 6
        max: 32
      newPassword:
        type: String
        required: true
        min: 6
        max: 32
features:
  comments: 
  uploads:
    file_sharing:
integrations:
  oath:
    twitter:
    facebook:
    youtube:
```
