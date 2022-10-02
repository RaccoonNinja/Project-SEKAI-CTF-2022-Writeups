## Skills involved: Open Redirect + JWT Token Forgery

This challenge is very clear about what to do. I have read of the exploit some time ago but actually crafting one is super fun.

## Solution:

Upon visiting the site, we're greeted by this friendly message:

![image](https://user-images.githubusercontent.com/114584910/193477052-fa816151-8349-484e-b717-83e915c705c3.png)

*OK*. After pulling the source down I checked if there's any outdated packages - none.

I was not very familiar with the Python Flask framework but here's a summary:
- Anything, absolutely anything will be used in the `template.html` template due to `@app.after_request` (*remark: there is no SSTI*)
- There's a jwks.json file that gives a public key used by the server (*remark: it is not possible to crack it - I tried*)
- Accessing `/api/flag` as admin will give us the flag right away. (*remark: this can be confirmed by removing the authorize code*)

The *key* here is that we can *CHOOSE* our host for the jwks.json:

```py
valid_issuer_domain = os.getenv("HOST")
valid_algo = "RS256"
def get_public_key_url(token):
    is_valid_issuer = lambda issuer: urlparse(issuer).netloc == valid_issuer_domain

    header = jwt.get_unverified_header(token)
    if "issuer" not in header:
        raise Exception("issuer not found in JWT header")
    token_issuer = header["issuer"]

    if not is_valid_issuer(token_issuer):
        raise Exception(
            "Invalid issuer netloc: {issuer}. Should be: {valid_issuer}".format(
                issuer=urlparse(token_issuer).netloc, valid_issuer=valid_issuer_domain
            )
        )

    pubkey_url = "{host}/.well-known/jwks.json".format(host=token_issuer)
    return pubkey_url
```

While the domain is whitelisted and a proper URL check is done (as opposed to `/^http://localhost:8080/`), there's another **open-redirect vulnerability**. Namely we can redirect the server to our own domain, bypassing the whitelist:

```py
@app.route("/logout")
def logout():
    session.clear()
    redirect_uri = request.args.get('redirect', url_for('home'))
    return redirect(redirect_uri)
```

This is basically a challenge based on https://www.invicti.com/blog/web-security/json-web-token-jwt-attacks-vulnerabilities/, which included this exact exploitation path.

Now comes the technical part:
- For crafting the public/private key pair:
  - I referenced:
    - https://blog.digital-craftsman.de/generate-a-new-jwt-public-and-private-key/
    - https://www.misterpki.com/openssl-genrsa/
  - Being a self-proclaimed RSA expert, I used openssl to do the work.
- For hosting the key:
  - I once had immense difficulty on this when another challenge from somewhere else asked me to host an HTML file for CSRF.
  - Thanks to this challenge I realized that we all have a very powerful tool that we all already know: https://webhook.site
  - *Yes, we can change the default return message, status code and content type with webhook.site*
  - Remark: care needed to be taken to create the correct data structure, remove leading and trailing content, etc
- For generating signed JWT:
  - I used the PyJWT package, but using https://jwt.io is equally fine:

```py
import jwt

private_key = open("private.pem").read().strip()
public_key = open("public.pem").read().strip()

encoded=jwt.encode({"user":"admin"}, private_key, algorithm="RS256",
headers={"issuer":"http://localhost:8080/logout?redirect=https://webhook.site/..."})
```

The flag should come to you in no time.
