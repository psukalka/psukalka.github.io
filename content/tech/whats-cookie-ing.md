+++
date = '2025-04-12T22:46:43+05:30'
draft = false
title = 'Whats Cookie-ing'
+++
We are trying to improve security of our CDN with cloudfront cookies. Cloudfront provides two approaches: one using signed urls and the other using signed cookies. The later suits better for our media usecase where there are lot of fragmented files. 

Anyway, I generated a privately signed cookie and sent it in an API response. Then I used this in sending an API request from Postman. And voila it worked. POC was successful. 

Now, it was time for integrating cookie in web player. FE sent the same cookie as was returned in API response. Surprisingly, web player was getting `403 Access Denied` error now. Same way worked in postman but not in web. 

First hunch was that CDN was dropping the headers. So, we updated the distribution policy to allow all headers. But that didn't work. Chrome didn't allow to set cookie like this when accessing a third party resource. 

So, we then decided to change the approach. Instead of sending the cookie in an API response, I sent it as a cookie response. Django sent `Set-Cookie` in response. In cookie, we set the allowed domain to `.cloudfront.net` as we wanted to access Cloudfront resource. Tested the solution in Postman. It worked. 

It was time for round 2 in Web Player. And it again failed. There was some yellow warning when we were checking the cookie in `Application` tab in dev tools. It wasn't allowing to set cross site cookies. Meaning I can't set cookie for `.cloudfront.net` from my `.example.com` domain. 

Another setback. What to do now ? So, we created another domain name reference `media.cdn.example.com` (through CNAME record) to access our resources. 

This time, browser allowed to set cookies. And finally we were able to access the CDN resource using cookie from web player. 

This made me realize the various levels of security that browser implements to prevent users from malicious attacks. 