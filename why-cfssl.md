
# What does CFSSL does ?
- It can work as a certificate bundler as well as lightweight Certificate Authority.

======================================================== We have to put an image here =======================================================================>

### How it works as a Bundler or, what even is a Bundler?
- In the most simple terms, whenever you see the TLS certificate of any trusted website, you'll see a chain, usually split between root, intermediate and leaf.
- The leaf certificate is what is owned by the owner of a website, and it is trusted by the authority above it, which itself is trusted by another. It is a chain, which traces back to the root authority, which self signs it's certificates.
- Why does the root issuer does not have an issuer and is allowed to self sign? well, this authority has been through some rigorous rigorous audit and they have been there for a while (the whole trust thing). So, eventually we need someone to start the chain of trust right? that is them.
- There can be one or many intermediate organizations signing the certificates and that is where the whole "managing the bundle" comes in. Why ?
- In order for your browser to trust a server, it must trust the root authority. Different browsers support different root authorities !
- for example:
	- Firefox trusts 143 root certificates but Windows 8.1 trusts 391, you see the problem here ? what if one of your customers uses Firefox and one uses windows 8.1, if you choose the wrong root, one user might not be able to access your product.
	- Older systems don't even know about new root authorities, how will they trust them ?
	- Older systems don't even support cryptography.
- So, there is this entire bundle of certificates that are connected in the form of a chain of trust. If you choose the wrong bundle, some users loose access to your service, how CFSSL solves it ?
- CFSSL bundler has two flavors, the CLI and an API, but it is pretty simple, you just give them your certificate (API/CLI any way) and it will the necessary information you need.
- If you want to create a bundle with CFSSL, then you have to tell them about your target audience (eg: like who will visit your website) and the certificates their system trusts

### CFSSL as a certificate Authority:
- It does all what is basically needed for certificate creation, that includes creating a private key, building a CSR and eventually signing the certificates.
- So, if you want to build certificates on your own, use CFSSL as the CA, that will sign your certificates !

## Why is it used in Kubernetes the hard way ?

- Since Kubernetes itself if a distributed system, it contains many different components like API server, Kube controller manager, Kube scheduler, Kube proxy etc. All of them are separate services and need to communicate to each other in order for the cluster to work. For security purposes, these services use TLS to communicate and follow the PKI infrastructure, therefore they need things like (trusted) TLS certificates and CA to provision them. CFSSL does that for us.