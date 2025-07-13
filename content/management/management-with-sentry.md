+++
date = '2025-07-13T21:24:34+05:30'
draft = false
title = 'Management With Sentry'
+++
I'm helping a friend in managing his tech venture. He got the project built with a Consultancy Service and they have done quite a good job. When I started on the project it was a live app working in production. I had to manage the tech team given that the consultants had now done their job. 

Codebase was in Python (Django) which I am already familiar with from my office work. I started going through the codebase. I understood the flows in a few days. I started giving time after office hours while their team used to work in daytime. So, there was no syncing between me and the team. Neither did I have any test app build that I could use to identify important API flows. On top of that everything seemed to work, so I didn't have any way to barge in. As a production project whatever I would imagine should be in place was already there and even better than I would think in some places. When something is working, it is difficult to make your way in.

So, a couple of weeks went by. I hadn't written a single line of code. Neither could I make any other contribution to guide team as I wasn't syncing with them. I used to go through Pull requests but that also didn't help much. 

There were complaints though that app usually crashes in production. Also, there was less visibility in BE bugs, if any. So, it was natural for me to integrate Sentry. I have used Sentry but never integrated it myself in a project. I started the activity. I integrated it in test environment. 

Voila! A few bugs popped up in dev environment. That gave a breather. These were few things that I could start from. I wanted to but didn't actually do anything on that too. I wasn't confident of deployment flows. Later I decided to push the sentry to production. Once I did, alerts started coming in heaps and bounds. 

And the stage was set to barge in. There was an alert that popped up for around 10k times in a day. I fixed it and that became my first PR. After merging the PR and deploying it to production I finally felt like first step was taken. Alerts kept on popping and I kept fixing the bugs. While I couldn't push a single line of code in 4 weeks I pushed 5 PRs in 2 days. Sentries started coming under control. 

In absence of Sentry, the project was written in logging friendly way. But that wouldn't work in a scaling app. Any tiny alert should be flagged on the go. So, we started changing the structure of few important flows in BE. Sentry helped us in finding performance issues (like multiple N+1 queries that were slowing the app). It also helped in finding a rare concurrency issue where a payment was being captured twice when Google returned two successful webhook requests within difference of micro seconds. This resulted in creation of duplicate orders. Such issue would have been extremely difficult to find without Sentry. 

Then team started pointing some Call related issues. I picked the calling flows and started understanding them. With Sentry I had confidence that if I push anything invalid that would be flagged immediately. With this confidence I could provide valuable inputs to the team on how and what things should be captured. 

Without Sentry, identifying bugs was quite difficult so team had started on logging every flows in detail. This consumed their bandwidth and a vacancy was created for new BE developer. Once Sentry was integrated there was no need for such exercise, and we started utilizing bandwidth of existing developers in more productive way. The BE vacancy got removed. 

Thus, just with a Sentry integration (i.e. setting up a right process), I was able to gain confidence and manage the product in better and confident way. 