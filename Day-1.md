This is the base knowledge you need, before you truly understand what is the use of CFSSL here in Kubernetes the hard way:

# PKI (Public key infrastructure):
It is a back-end cybersecurity measure, it is a set of rules, policies and procedures needed to create, manage, distribute and revoke the digital certificates and manage public-key encryption. It involves these primary steps:

- Creation of certificate authority.
- Provisioning of certificates.

The certificate Authority is provisioned and initialized first. Then, any two systems that want to authenticate using TLS send the CA a Certificate-signing-request (CSR), which gives them a TLS certificate which they can use to securely connect and communicate with other trusted services.

If you understand all of what I said, you don't need to read anything written below, but if no, just get started and once you finish, re-read the above section.

# Security basics:

I will be considering the client-server architecture here, because almost all the people are familiar with websites, clients etc. But the concepts explained here can be used for any two systems to communicate.

Let's consider two systems, they use a client-server architecture for communication. They can use a protocol like **HTTP** or other right ? but why all the websites (secure ones) you visit, via your internet have **HTTPS** and a "lock" icon in front of the URL ?

That is because of many flaws present in bare HTTP (many other protocols), the biggest one is that it sends and receives the data in **plain text**. Which means anyone who can gain access to it, can read it. Now imagine whatever you have to say (write) can be read by anyone ? specifically people who are bad enough to try to gain access to it in the first place.

The necessary things to be present when two systems communicate are:
- Confidentiality - Only the one supposed to see the message should see it, no unauthorized access.
- Integrity - The message was not altered before it reached from client to server.
- Identification - Who is the one the client is talking to ?
- Authentication - Is the server real one ? or they are impersonating to be someone else ?

This gives rise to security and it has a long history and iteration, which of course I am not going to talk about, we will start with this.

# Confidentiality with cryptography:

So the problem was that anybody can read messages right ? let's just **encrypt** it with a key, so that even if they gain access to your messages, it's all just some gibberish that doesn't make sense. But how do the person you actually want to talk, understands the messages ? it's a gibberish for them too ! We can use cryptography to solve the problem.

You can google what cryptography means or just check out "caesar cipher" and you'll understand. there are two ways:


## Single key / Symmetric Cryptography

So, the data that the client want's to send to the Server is now going to be *encrypted* with a *Key*. Server can use that key to *decrypt* the messages i.e. **SINGLE KEY FOR BOTH, ENCRYPTION AND DECRYPTION**.

This also has many problem, How will the client provide the key to the server ? and what if the key itself is stolen and the bad person (hacker) makes a copy of it? They can also use the same key to read client's secret messages ! what was the point of the whole encrypting thing anyway then. So, we have a better way *Asymmetric key encryption*.


## Public key / Asymmetric Cryptography:

This process uses **Mathematics**, but all you need to know is that there two keys (instead of one) and they are mathematically connected to each other in a way that, data can be encrypted with one key and decrypted only by the other i.e. **ONE KEY FOR ENCRYPTION, ANOTHER FOR DECRYPTION**. The key pair is called public-private key pair, one property is that you cannot derive *private key* from *public key* but the reverse is possible.

So, the client uses Private key to decrypt the messages and the server uses the public key for the encryption.

**Client wants to send a message to server:**
- It uses the server's public key that is openly available, to encrypt the message and sends it to the server. Only the private key can decrypt the message and is kept very safe and hidden.

**Server wants to send a message to client:**
- Server uses the client's public key to encrypt the message and send to the client, only client's private key can decrypt the message now.

The biggest upside is, the public key is openly available and can be used by anyone to encrypt the message, before sending it to the receiver.

What is the downside ? **Asymmetric encryption is slow in comparison to Symmetric encryption**. So, remember this flaw, we will address it later :D


# Integrity with Message Digests:

A message digest is a fixed number of characters that is Unique, it is a numeric representation of the contents of the messages.

As you know the size of the message can increase or decrease depending on the sender, but message digest is unique because it is created by transforming the content of a message (of changing size) into a numeric representation (of fixed size) using a hashing algorithm. The hash created has two properties:
- It is not reversible i.e It is impossible to convert the numeric representation back to the message again.
- No two messages can have the same digest. i.e. the digest is unique.

Now you might be wondering, how does integrity come in here ?

## How it happens:
Any package that a client sends to the server to communicate, contains the following things:
- The encrypted message (encrypted with receiver's public key).
- The message digest
- The hashing function used to create the message digest.

Now, on receiving the message, only the server can decrypt it because it was signed by it's public key, hence **Confidentiality**. After decrypting. the server uses the hashing function to convert the (decrypted) message into a message digest. Now, the server compares the newly created message digest with the one that was created and passed by the client. If they both are same, the message was not altered it's all good hence, **Integrity**. Otherwise something is wrong and someone was able to alter the message, maybe the keys are compromised.


# Identification:
For a server to be connected to a client, it must be unique. That uniqueness is how the client/server will know whom they are talking to. This is comparatively easy to provide.
- Mail
- Organization name
- Domain name etc.

All you need to know is, Identification part can be implemented in many ways for different things. Eg: A server can identify a client on the basis of the username/password combination. Maybe their email etc. but they just have to be unique to identify an entity in the system containing many.

In the above topics, we have achieved:
- **Confidentiality**, where we talked about using encryption to prevent unauthorized access. 
- **Integrity**, where we talk about checking if the messages were changed using the message digests.
- **Identification**, this is about uniquely identifying someone. 

# Authentication with Digital certificates:
Certificates and the protocols used for verification have a long history. Starting from SSL to a world where it is deprecated and TSL is used. here are the basics.

## Basics
So, the client wants to talk to a server, server's public key is openly available so messages can be sent to it, but, what is the proof that there is no one impersonating to be that server, After all, there is no proof that the public key really do belong to that server and not some random person, who put their own public key in place of server's. This "trust" or "assurance" is given by the digital certificates.

## What is a certificate:
A certificate is a way of binding a unique **Identification** with a **public key**. It is created by **Certificate authorities** (CA). So, when the certificate is issued, it is the CA's way of vouching for the owner of the certificate, that they really are who they are claiming to be. All the browsers who trust that CA, trust the certificate and hence trust the server.
- It helps in preventing man in the middle attack.
- It follows X.509 and contains following information:
	- Owner's public key.
	- Owner's distinguished name.
	- CA's distinguished name.
	- Serial number, no two certificates signed by the same CA have same serial number.
	- Current version of X.509 standard
	- Date of creation and expiration of Certificate.
- Put the photo of the entire process here.

The organization who want's to be verified sends it's unique identification and the public key to a CA, the request is called Certificate-signing-request (CSR), the CA takes the info and verifies it. Eventually *Issues* a certificate by signing the information with it's own private key. Now the organization is *trusted* and their certificate contains all the information needed to authenticate the server.

**Server's side**:
Before a server starts serving traffic, it has to get a TLS/SSL certificate, that is signed by a trusted Certificate Authority (CA). Once it is certified, it's HTTP connection is secured and is now HTTPS. Now, all the connections with the client will be TLS secured.

**Client's side**:
Before connecting to any server, the client will check if the server has a TLS/SSL certificate ? In other words, does it use HTTPS ? if yes, only then will it try to connect to the server using **TLS Handshake**. Once connection is established, they can start communicating.

Now, you might be wondering, how can we trust the CA ? well, they have been through some rigorous security tests and have been around for a while. They are the first block in the *chain of trust*.

## Cryptographic protocols SSL/TLS:
- These protocols enable two parties to identify and authenticate as well communicate with data integrity and confidentiality.
- These protocols have two layers: Record protocol and Handshake protocol.
- It enables two parties to identify each other and share information with confidentiality.

## TLS record protocol:
- It is responsible for confidentiality and data integrity

## TLS handshake (Handshake protocol):
Now, the server is certified, TLS secured and ready to serve the traffic, all the client needs to do is *check* if the server is trusted, has a real/valid TLS certificate and it can establish the connection with the server.

Today, any website of remote server, that is TLS secured and you try to connect, a TLS handshake takes place, it happens so fast that you might not even realize, but it is there. It is there because it enables the connection between the client and server to be secure and fast. It leverages both Symmetric and Asymmetric cryptography hence, it is fast and secure, let's just see what it is.

### What it is:
- It is a negotiation between two parties on a network, they decide many things that are needed for a proper communication between them.
- What version of TLS is going to be used.
- What is the cipher suite.
- Verifies the server (Authentication).
- Finally establishes a secure connection between the machines.

### This is how it happens, a simple version:
- It starts with a **"Client Hello"** message, that contains following information:
	- The TLS protocol version supported by the client, in an order of preference.
	- The **cipher suits** supported by the client.
	- Some random byte (you will know why ? later)
- The server responds with a **"Server Hello"**, that contains the following information:
	- The selected Cipher suite and the TLS version.
	- A session ID.
	- It's digital certificate.
	- Some random bytes.
	- Also, if the server wants client's certificate, it can also make a "client certificate request"
- The client verifies the certificate (How ? it's below).
- Now the client and server do a **Key Exchange** to eventually produce a **shared secret**, which is insanely secure and start using that secret as a key for encrypting and decrypting (symmetric key cryptography) the messages, so that the are fast and secured.

Want to know more about TLS handshake ? watch these videos:
- TLS handshake.
- Secret key exchange (Diffie-Hellman).
- Network stack and the internet.


### How are the certificates verified ?

Since the certificate was signed by using the CA's private key, it can be decrypted by the client using it's public key freely available online. The client decrypts the certificate to get out the information about the server. There is a lot of ground to cover here which is not the scope of this article. You can check more about online but here is something interesting for you. You can click on the "lock" icon present in front of URL of any (secure) website and you will get to see it's certificate, the issuer's name, chain of trust etc.

TLS handshake leads to creation of a TLS secured session, where the shared-secret key is used for communication for a while until it is recreated (different from previous ones) when either it expires or the client establishes a new session by rejoining the connection. In both the cases a TLS handshake happens again.

This makes use of both public key cryptography (to exchange keys and create a shared secret) and symmetric key cryptography (for fast encryption and decryption of messages).

This is all the prerequisite knowledge we needed to get started with CFSSL and why was it used in Kubernetes the hard way. Go and read the first paragraph of this article now, You will surely understand what PKI is all about.