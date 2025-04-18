+++
date = '2025-04-18T21:02:55+05:30'
draft = false
title = 'Memory Overload'
+++
An Issue was reported that videos were not being uploaded from google drive to S3. We have an asynchronous task that transfers files from google drive to S3 (for further processing). 

My first assumption was that the tasks might have been queued. Also there weren't any sentry alerts, so I ignored the issue. Even after few hours, the issue was not resolved so it seemed something was cooking. 

I tried to upload a 2MB file from drive and it got processed instantaneously. So, the pipeline was working fine. I looked at the file that they were trying to upload. It was a whopping 8.3GB file. That raised some doubts. Is google drive blocking such file ? Is some network connection getting reset ? Is S3 limiting file upload ? 

I added few loggers and ran the task synchronously for mentioned file. It was progressing fine. Then suddenly it stopped at 65% . Waited for about a minute and then task got killed. 65% of 8.3GB was somewhere around 5GB. I tried to check the file locally where it was being stored. But there wasn't any file to be found. 

We were storing the file in memory rather than writing it on disk. 
```
fh = io.BytesIO()
```
The machine on which task was running had 8GB of RAM and 3GB was already utilized. So, it could only store 5GB of downloaded file and the process got killed when memory was 100% utilized. 

Sentry alert was not raised as the process was killed by linux due to memory overload. 

To fix the issue, I stored the file locally in temp directory. Uploaded the file to S3 with `multi-part` upload (as recommended for larger files). Finally cleared the temp file after upload. 

Eventually, we could upload 10+GB files without any issue. 