+++
date = '2025-03-30T19:34:07+05:30'
draft = false
title = 'Hosting a Site'
+++
# Hosting A WebSite

Now that you have purchased a domain, what will it show ? where will your code reside ? where your images / video files be stored ? how will you run scripts to build your code ? 

Let's go step by step. 
Say I just want to display an `index.html` page on my website to host my resume. 

1. If you have an AWS account, a simplest way is to create a static website in S3. Create a bucket in S3, create an `index.html` page in it. Enable `static website hosting` for your bucket. Create `CNAME` record in your domain to point to this static website and your resume is up. 

    a. The problem with this approach is you don't have code version control. 
    b. Even if number of pages grow in double digits, managing it would become quite difficult. 

2. A second alternative is to host website on github. If you have a github account, you can create a repository `<your-username>.github.io` . Add `index.html` in it. Go to `Pages` in repository setting and point your github to your domain (ex. `pvsukalkar.in`).
Also go to your domain and create `A record` to point to github IP addresses. Create a `CNAME record` to point to your github repository. And you are done. Your github repository will be accessible over internet. 

Pros: 
    - You have version control
    - You can take advantage of Static Site Generators (Jekyll, Hugo, Gatsby, ...) to build complex static websites

Cons:
    - Your website is still static. You can't add dynamic content to it
    - You can't track user activity, can't get feedbacks from users, ... In short, your website is not interactive. It is mostly read-only

3. If you want to go beyond this, then you will need to shell out some money and buy hosting service. Hosting service providers will give you dedicated machines where you can write, deploy code. Have access to databases, complex services, ... If you have light requirement of these services you an use Vercel. If you want to build a complex product that would require storage, transcoding, load balancers, database services, kafka, ... you can go with cloud providers like AWS, GCP, Azure, ...

So, hosting a website depends on requirement. If requirement is simple, you can even host a website freely. But if you want to build a business out of it, you will have to shell equivalent amount of money.