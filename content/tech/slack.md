+++
date = '2025-07-05T11:11:53+05:30'
draft = false
title = 'Slack Integration'
+++
With scale, time becomes of essence in resolving an issue. Thus, identifying and resolving issues quickly becomes of essence. Collating data from multiple sources to debug an issue is not desirable in such case. If your workspace is on Slack, then getting alerts on Slack increases visibility and saves time in crisis (when it matters). Slack can also be used to send alerts of any background tasks like celery tasks, jenkins deployments, ... 

To integrate slack in your workspace there are two ways: 
- Send alerts through slack webhook
- Send alerts through slack app 

Webhook is not a recommended way as it has limited support. Slack app is generally preferred. 

---
First step is to create a Slack app in your workspace at: `https://api.slack.com/apps` . Give it some meaningful name like `botalerts`. Your app is created in 2 clicks. 

Now, go to `OAuth & Permissions` section in the app. You see the option to `Install to workspace` . Click it and you will get an `Auth Token`. Note this, you will need it later. 

Also grant `chat:write` , `chat:write.public` permissions to your app so that it can write messages to your channels.

---

Now, install `slack-sdk` using pip: 
```
pip install slack_sdk==3.35.0
```

---

Use this sdk in your project (I used Django project) to create a WebClient. This WebClient will need the slack app auth token created above. Sample code to integrate slack service in your app: 
```
from django.conf import settings
from slack_sdk import WebClient


class SlackService:
    _instance = None
    _client = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super(SlackService, cls).__new__(cls)
            cls._instance._initialize_client()
        return cls._instance

    def _initialize_client(self):
        """Initialize the Slack client only once"""
        if self._client is None:
            self._client = WebClient(token=settings.SLACK_TOKEN)

    def send_message(self, channel='#system-reports', message=None):
        """Send a message to Slack channel"""
        if not channel or not message:
            return None
        self._client.chat_postMessage(
            channel=channel,
            text=message
        )
```

In your workspace create some channel, say `#system-reports` . Now you can send message to this channel as follows: 
```
SlackService().send_message(channel='#system-reports', text='Test system alerts')
```

---

That's it, your slack integration is done. Now you can utilize this to send any alerts from your app.