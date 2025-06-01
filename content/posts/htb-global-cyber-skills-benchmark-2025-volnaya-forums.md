+++
title = 'HTB Global Cyber Skills Benchmark 2025 - web/volnaya-forums'
date = 2025-05-27T21:44:01-04:00
draft = true
+++


Volnaya Forums is a Next.js web application simulating a forum platform. The goal was to obtain the flag, which is only accessible to the admin user via the /api/auth endpoint.

We are first presented with a login page, where we can create a new user and login

![volnaya1](/images/volnaya1.png)

The forum is filled with posts, and we are not allowed to post but we can report posts.

![volnaya2](/images/volnaya2.png)

Also, we have a user profile:

![volnaya3](/images/volnaya3.png)



The flag is served at session.user.username === 'admin'
/api/auth and only returned when `session.user.username === 'admin'`. There is a Puppeteer bot ( bot.ts ) that logs in as admin and visits URLs submitted via the
report system. Report System: Users can submit reports that trigger the admin bot to visit specified URL's. 


In profile, the description field is not sanatized and vulnerable to xss:


```html
    <div
        className="prose"
        dangerouslySetInnerHTML={{ __html: profile.bio }}
    />
) : (
    <p className="text-muted-foreground">Not specified</p>
)}
```

To exploit, first we can inject XSS into our profile. Next, we create a report and have the admin bot vist our profile. Our XSS payload will steal the admin bots flag.

After trying a few payloads, this one worked:

`<img src=x onerror=alert(1)>`