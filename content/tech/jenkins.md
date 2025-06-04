+++
date = '2025-06-04T23:01:50+05:30'
draft = false
title = 'Building Jenkins'
+++
I have used Jenkins to deploy jobs but that's the extent of my usage. I was going to deep-dive into Jenkins. But before that I got a thought, why not think of building Jenkins if I had to build one ? If I was to build Jenkins, how would I have done it ? 

So, Jenkins for me is an `automated deployment` tool. So, before automating something, how would I have done the deployment manually ? Say, I want to deploy a Django project from GitHub to production. I would follow these steps:

1. Detect that there are some changes in the `master` branch 
2. SSH to the production machine
3. Run the deployment script

So, if I want to automate this, the steps would be as follows:
1. Detection can be manual, through cron job (pull mechanism) or some webhook (push mechanism)
2. To SSH to the machine I would need two things. IP address of the machine and access to that machine
3. To run the deployment script I would need it present somewhere. It can vary depending on codebases, languages, ... Better if this is part of the codebase itself. 

----

Now let's think about the Jenkins process that would be running somewhere. The nudge that something has been pushed to git branch can come anytime. So, this process would have to continue listen over network. So, this should be some server. I'm comfortable in Django, so maybe Django server. But it would consume more memory so might search for some other alternatives. 

Now if it is a server, will it have any database ? Otherwise how will it store the status of deployment, time taken to process the deployment, logs of deployment, ... ? This data should be stored somewhere. sqlite would be the most obvious option to not increase server footprint. 

Now, when a deployment starts we have identified that the script to run would be stored in some (JenkinsFile) inside the codebase. But how would my Jenkins server contact the production server where code is to be deployed ? How will it know about the IP addresses ? 

Since this is manual information to start the deployment this should be provided while creating the `Job`. 

So, there should be one API like: 
```
POST /api/v1/jobs/

body: 
{
    "branch": "master",
    "codebase_url": "github path",
    "ip_addr": "static address or script if the IP is dynamic like spot instances"
}
```
In response a job would be created with some id.

Then I can trigger deployment on this job with: 
```
POST /api/v1/jobs/<job-id>/deployments/

body: 
{
    "params": "if any"
}
```

And finally I can track status of the deployment using: 
```
GET /api/v1/jobs/<job-id>/deployments/<deployment-id>/
```

---
Ok, let's think about the deployment script running on the production server. If it is running on the server, how will logs come back to Jenkins ? Does it copy them everytime or it serves them from production server when needed ? 
Secondly, should the tracking API be pull based ? I guess deployments won't be that frequent and server should be able to handle these many GET requests so that should be fine. 

----
So, we have thought about the core functionality that Jenkins should provide. But how should its architecture be ? Like at each stage there can be different instances: 
1. Users can use multiple version controls like GitHub, BitBucket, Perforce, CVS, ... so a plugin architecture for this
2. The production machine can be inhouse, on AWS cloud, GCP cloud, ... so some standard mechanism to grant permission to Jenkins and some standard mechanism to find dynamic IPs at runtime
3. Deployment scripts can vary depending on languages like Python, Java, Rust, ... In Python, just reload server would be enough. For Java compiling and then deploying the jars is needed. Requirements can vary as per languages so some plugin for languages also should be needed. 
Finally, the platform where this is run (Linux / Windows / ...) should also be handled. 
With this in mind, I'll now go and deep-dive in Jenkins on how they built it. 