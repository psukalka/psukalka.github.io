+++
date = '2025-10-07T14:15:38+05:30'
draft = false
title = 'Downscaling RDS'
+++
So, when we found out the root cause for [memory full](https://pvsukalkar.in/tech/memory_full/) the next step was to reduce the storage. 
After cleaning up the replication slot, WAL size had come down drastically.
Now, 3GB storage was enough to hold current databases. But we were still paying for 2TB usage. 
So, we had to downscale RDS to reduce monthly bill. 

Upscaling is easier in AWS and downscaling is equally tedious. 
RDS Storage are based on EBS volume which only expand and don't allow shrinking.
To upscale, it is just one click job (increase storage) but to downscale you need to create another database and switch traffic to that. 
In current setup, Blue-Green deployment provided by AWS seemed most convenient option (compared to DMS, or restore from storage).

Steps of Blue Green Deployment:
- Create a BG deployment 
- Blue is current database 
- Green is new database. In green database use new config (like 200GB storage instead of 2TB storage)
- Validate config and start deployment 

Deployment took about an hour. During the time, blue db was up so there wasn't any downtime. 
After the deployment, new database was up. I connected the prod server to it and tested. It was working fine. Cool, everything seemed perfect.
I asked Pradeep to enable CDC on new database. 

And after few minutes when I was away sentry alerts started flooding. 
Write queries (Insert, Update, Delete) queries were not working. The green db that was created was a read-only database. 
To make it primary, we had to `switch over` the deployment. 
Once the switch over was done, the green db would become writable. 

After few hours, when I didn't see any errors being reported I decided to delete the deployment. 
While deleting deployment I kept the green db (as it was the new database). 
After that deleted the blue database. Before deleting took snapshot of the db in case anything inexplicable happens. 
So, finally I was left only with the green db. Eventually renamed it. Sentry alerts again started coming. Built docker images again with new connection url and everything was fine. 

That's how I downscaled RDS (postgres) from 2TB to 200GB storage (eventually saving $200 monthly).

---

Steps for next downscaling (if required):
- Create BG deployment 
- Select green with desired configuration
- Once deployment is done, switch over 
- After switch over connect to the green database and ensure that it is writable. 
if not forcefully make it writable. 
- Switch production traffic to the green database 
- Ensure that no alerts are raised 
- Delete blue database when it has zero traffic (stop it temporarily before deleting if you want to ensure it is not being used)
