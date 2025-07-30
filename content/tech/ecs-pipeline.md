+++
date = '2025-07-30T10:48:04+05:30'
draft = false
title = 'ECS Pipeline'
+++
If you have a docker image of your service you can deploy it scalably using ECS. 
AWS handles all the orchestration for you. You need to provide a `task definition` and once deployment is run, ECS creates tasks (running containers) as per task definition. An ECS service ensures that desired number of tasks are always running.

For standard services you have container images publically available. But mostly if you are using ECS, you will be trying to create your own docker image that would run some server or anything. But then how does ECS know about your docker image ? 

For that, there is ECR (Elastic Container Repository). Once you create an image of your docker build you push it to ECR. Images uploaded to ECR are accessible to ECS and you can refer them in your task definition. 

Ok, now I have an orchestrator (ECS) and an image (stored in ECR) but how does it run ? Won't it need any server to run ? 

AWS provides 2 options: 
- Fargate : For serverless running
- EC2 : You manage EC2 and ECS scales containers on this 

So, while creating an ECS Service I can choose either Fargate or EC2 deployment type. 

But how would traffic come to my ECS service ? If I want to run a server on ECS, how would traffic reach there. A simple solution can be that I may run ECS in public subnet of VPC with public IP and that way traffic can come to it. A more secure solution would be to use load balancer to redirect traffic to ECS. Load balancer is in a public subnet of VPC having internet gateway. So, when traffic comes to the load balancer, it can redirect the traffic to ECS (which maybe in private subnet). 

Cool, now my ECS is working and everything is fine, but what happens when it starts working ? 
How do I debug it ? How do I see logs ? Under ECS service logs are captured for all incoming requests and can be seen in cloudwatch.

But doesn't this seem similar to kubernetes ? Indeed it is. ECS is AWS's simpler, managed alternative to Kubernetes. If you have requirement of scaling containers, you can simply do it with few clicks using ECS. You don't need to use Kubernetes. If at all you want to use Kubernetes, AWS has alternative Elastic Kubernetes Service (EKS). 