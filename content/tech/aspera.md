+++
date = '2025-06-28T07:42:43+05:30'
draft = false
title = 'Aspera'
+++
I heard about Aspera first time today. It is not a file storage, rather a file transfer technology. But what is the benefit ? and why do you need another technology ? Let's dig deeper. 

Say, you are a production house. You produce movies or videos. The raw content is generally quite large in size. Like 100s of GB and sometimes can even go in TBs. Transferring this much content via network is quite slow. Instead just sending the content via hard disk is faster. Aspera shines for such cases. Despite size of content you get consistent speed. This is what happens internally. 

Aspera created their protocol FASP (Fast, adaptive, secure protocol) which is built on UDP instead of TCP. Other protocols of transfer like FTP, HTTP are built over TCP. TCP uses congestion control that backs off when their is packet loss, interpreting loss as network congestion. On high-latency, high-bandwidth networks (like intercontinental links), TCP's window scaling becomes a bottleneck. TCP's acknowledgement requirements create round-trip dependencies that limit throughput. FASP bypasses congestion control and implements its own adaptive rate control thus enabling it to use almost 100% of available bandwidth utilization even on high-latency links.

If it is using UDP then how does it ensure reliability ? A transferred file having even 1% data loss won't play correctly, right ? Yes. To ensure reliability Aspera client maintains count of received packets. If it detects any packet loss it can ask aspera server to send those particular packets again thus ensuring reliability. 

As you might have guessed, since this (FASP) is a new protocol (not built-in) it needs its own client and server to understand it. So, you need to install Aspera client to download / upload files. Eventually you would be using some S3 / GCS as file storage but Aspera can't directly store files in these (as these storages understand TCP). You can solve this problem by installing Aspera server on EC2 that then stores files in S3. Here Aspera server acts as protocol translator, using UDP for client <-> Aspera server high speed transfer and HTTP / HTTPs (via TCP) for Aspera server <-> S3 APIs.