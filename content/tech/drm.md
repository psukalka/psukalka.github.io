+++
date = '2025-05-16T00:59:07+05:30'
draft = false
title = 'DRM'
+++
Billions Of Dollars are annually lost by production companies due to piracy. 
Someone can download your content and distribute it on free channel. Thus, you lose on revenue that user might have paid.

How do you solve this problem ? Or can we even solve it ? 
If someone is playing a movie on TV and then recording it again through high quality camera (thus recreating it), you can't block this. This is called Analog Hole. Thus, you can't protect 100% piracy. 

But at least can we think of blocking piracy in digital content ? How would we approach it ?

Say, we are distributing the movie to 10 distributors. I do not trust anyone. 
Someone might leak my movie. Can I at least know who leaked my movie ? 
So, that I can at least legally sue the distributor and blacklist it from further distribution ? 

A simple way can be adding watermark in my videos. Just add name of the distributor in one corner. You must have seen such watermarks on TV or some online content.
So, if any distributor leaks the content we can easily spot who leaked it.

But is distributor the only problem ? Even the user who is watching the movie can share it with their friends. 
Downloaded movie can be shared shared across platforms too. So, what do I do ? 

Let's prevent users from downloading the movie itself. If he can't download it, he won't share it. 

But user is able to watch the movie it means there is some file that is being played. User just needs to figure out how to store the streamed file. 
People with little bit of technical expertise can still download your content from CDN / S3 endpoints. 

Plus, downloading is a legitimate feature. If I want users to encourage watching movies when there is no internet, I should provide download feature. 
But how would I then prevent user from sharing it ? 

I can encrypt the movie and then share it to the user. I can share the encryption key with user and then user's client can decrypt the content and play it locally. 
Now even if user has downloaded the content it is useless without the key. And the key would be shared by us after checking validity of the user. 

Cool, this is how digital rights of creators are protected. Managing digital rights is called Digital Rights Management (DRM).

Now, say I made a movie and am giving it to some distributor (like DramaBox). The channel claims that it has encryption and decryption mechanism. 
Would I give the movie to such distributor ? Despite their claims, I don't have trust over the strength of their security. 
Nor do I have expertise to test the security as I am a movie producer. 

Hence the big players pitch in here. To solve the piracy problem, Google has its Widevine DRM and iOS has its Fairplay DRM. 
The encryption-decryption that we talked about is handled by license providers certified by Google / Apple. 
If a distributor claims to use DRM of some certified provider I have to trust them. 

If license management is handled properly, I can claim my movie is digitally safe from piracy.