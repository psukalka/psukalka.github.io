+++
date = '2025-07-26T05:42:36+05:30'
draft = false
title = 'SSH vs HTTPS'
+++
Before we can ssh into a server, we need to (manually) provide our public key to the server. Server stores this key in `authorized_keys` file under `~/.ssh` directory. Similarly, when we try to ssh to a server, our laptop asks if you want to add this server to known hosts. 
If selected `yes` an entry will be made to `known_hosts` file in local system. This entry is of the server's public key. 

*Magic is that anything encrypted with a public key can only be decrypted with corresponding private key.*

So, if I have to send a message to server (via ssh), my system will encrypt this message with server's public key. Since server has its own private key, it can decrypt this message. Similarly, when server has to send an encrypted message to me, it encrypts the message using my public key which it had stored in `authorized_keys` file. My laptop can decrypt it as I have the corresponding private key locally. This is how messages are exchanged securely over ssh.

---

But I don't have access to the remote server when I access it over `HTTPS`. So, I can't add my public key manually over there. 
So, how does HTTPS communicates securely ? 

As a starter, the website presents its public key. My browser verifies this from Certificate Authority whether the certificate is valid. Once validity is ensured, it means the server public key part is handled. Meaning I can send encrypted messages to server and it can decrypt it. 

But how will server send me encrypted message that only I can decrypt ? For that it will need something like my public key, right ?

My laptop encrypts and sends a `pre-master secret` to the server. Both server and my laptops generate key 
locally using `pre-master secret`, client random number and server random number. Which algorithm (say AES) would be used was also decided earlier. Thus both sides have generated same secret key independently and remotely. Hence this is called symmetric key encryption. 

But how do you know that both of them have generated same symmetric key ? 

Both side send `Finished` messages. If the other party is able to decrypt the message, it can be ascertained that both have generated the 
same symmetric key (without actually transferring the key)

These symmetric keys are used for all further encryption and decryptions. Asymmetric keys (like we saw for SSH) are slower compared to symmetric keys. To compare their speed, 2GB of data can be encrypted in a second with Symmetric encryption while only around 2MB data can be encrypted with Asymetric encryption. That is also another reason why symmetric encryption is used in HTTPS.

Let's move ahead. A website will have hundreds of servers running in the background. Given the ubiquitous REST architecture where a request can go on any server, won't it be an overhead to do TLS handshake with each server ? Indeed, it would be an 
overhead and complex to maintain keys and do TLS handshake on each server. Hence, in production TLS is `terminated` at load balancer generally. Once load balancer ensures that request is a valid encrypted one, it forwards the decrypted request locally over `http`. 
So, you must have noticed redirection to `http://localhost:3000` or something in your nginx / caddy load balancers. 
