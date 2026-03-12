+++
date = '2026-03-12T14:50:41+05:30'
draft = false
title = 'Digital Signature'
+++
If I have a digitally signed document, how do I verify if it is really signed ? How can I ensure that it is more than just a green tick photoshop ? 

What if the amount of Rs. 10000 in document is tampered and made Rs. 100000 after I signed it ? 
What exactly happens when we digitally sign a document ? How secure is this system ? 

Say I want to sign a document then two keys will be created. 
- One private key 
- one corresponding public key. 

Private key is never to be shared with anyone. If it is stolen, then someone can forge your signature.

The signature of document will contain hash of document which is then encrypted with private key. 

```
hash = SHA256(document_bytes)          # fingerprint of the document
signature = encrypt(hash, private_key) # only YOUR private key can produce this
```

Anything encrypted with private key can be decrypted with public key. Thus anyone with the public key can verify the signature. 
```
expected_hash = decrypt(signature, public_key)  # anyone can do this
actual_hash = SHA256(document_bytes)
valid = (expected_hash == actual_hash)
```

So, what does SignDesk do ? It proves that this public key belongs to this specific human. 
```
"I, a trusted Certificate Authority (CA), certify that this public key belongs to Pavan, PAN: XXXXX, Aadhaar: YYYY"
— signed by the CA
```

So, your signature alone proves that document was not tampered. 
Certificate from CA proves that signer is you specifically and 
together they make a legally admissible e-sign. 

The green tick means all of these passed simultaneously:

```
1. SHA256(current document) == hash inside signature  → not tampered
2. Signature decrypts correctly with public key        → right private key used
3. Certificate says this public key = this person      → identity proven
4. Certificate chain → Root CA in your OS trust store  → CA is trustworthy
5. TSA timestamp proves cert was valid at signing time → timing proven
```

All five failing independently would require breaking SHA256,
RSA, the CA, the OS vendor, and the TSA simultaneously.
That's what makes it more than a green tick photoshop.