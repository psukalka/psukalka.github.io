+++
date = '2025-10-13T23:31:34+05:30'
draft = false
title = "Who's Watching ??"
+++
After reducing the RDS size, I was eager to watch the reduction in monthly bill. Earlier, RDS was costing around $11/day and now it should have cost me $3/day. But after few days of monitoring, it did not budge below $7/day. On splitting the cost by usage type, some storage cost had increased for RDS. On inspecting, I found that this was due to the snapshots that I had created before deleting the previous RDS instance. AWS provides free storage only upto the size of RDS (200GB in my case). But the previous snapshots were of 2TB. Hence, they were beyond free storage and were costing around $4/day. On deleting these snapshots, the RDS bill had come down to the expected value of $3/day.

While I was hell-bent on RDS cost, something else had started sprawling. While I was expecting the monthly bill to reduce, it had started to increase. On further inspection, Cloudwatch cost had started to rise and it was costing $80/day. So, all the efforts to reduce bill had gone in vain. 

First thing when inspecting cloudwatch was logs. Are we publishing too many logs ? Few devs had touched ECS pipeline around that time so I guessed ECS logging was set to debug mode and that would be costing more. But that wasn't the reason. In fact whole logging was costing only $1/day. Still $75/day was coming from data processing. What kind of data processing ? I don't know. 

Since the timeline of cost increased matched our migration date, it had to do something with RDS migration. We use DMS to sync production db with analytics db (CDC pipeline). When I checked its config, logging was enabled here and that's our culprit (is what I thought). Though logging was enabled, but it wasn't costing us. It was hardly pushing 1GB logs. 

Since it was related to data processing, it had to be something related to metrics. I checked `PutMetrics` , `GetMetrics` usage but it returned nothing. So, I listed all the metrics: 

```
~ $ # List ALL metrics to find custom namespaces
~ $ aws cloudwatch list-metrics \
>   --query 'Metrics[].Namespace' \
>   --output text | tr '\t' '\n' | sort | uniq -c | sort -rn
  18493 Glue
    640 AWS/Usage
    462 AWS/EBS
    264 AWS/EC2
    153 AWS/RDS
    123 ECS/ContainerInsights
    121 AWS/DMS
    111 AWS/NetworkELB
     64 AWS/ApplicationELB
     45 AWS/Logs
     34 AWS/Firehose
     30 AWS/PrivateLinkEndpoints
     28 AWS/Events
     26 AWS/Lambda
     26 AWS/EKS
     20 AWS/S3
     17 AWS/Glue
     15 AWS/NATGateway
     15 AWS/Athena
      8 AWS/States
      8 AWS/Location
      6 AWS/Rekognition
      6 AWS/HealthLake
      5 AWS/Bedrock/DataAutomation
      3 AWS/KMS
      2 AWS/ECS
      1 AWS/SecretsManager
```

As can be seen, Glue was pushing 18k+ metrics / day. And that was the culprit. So, I started digging deeper into Glue jobs. There were two Glue jobs. Claude gave me commands to disable metrics in them. But I was hesitant - whoever created these Glue jobs likely used the AWS Console, not CLI. So I went hunting in the visual interface instead. And there it was: under Job Details → Monitoring options → "Job metrics" was checked. I confirmed it with claude and disabled these settings. This should reduce our cloudwatch billing from today. 

Just a single click cost us around $80/day. So, knowing what to click is crucial. 