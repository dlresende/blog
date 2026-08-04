---
layout: post
title: "HTTPS explained to my Product Manager"
date: 2018-05-10 00:00:00 +0000
tags: ["https", "ssl", "tls", "tls handshake"]
---

Recently, I've been working adding TLS to some of the products I work with, mainly TLS over HTTP (also known as [HTTPS](https://pt.wikipedia.org/wiki/Hyper_Text_Transfer_Protocol_Secure)).

One of my main activities as Engineering Lead is to have high level technical conversations with Product Managers in order to explain benefits, trade-offs, value, risks, etc. related to a particular technology or solution we're planning to implement. By doing that, we can have better discussions about business value versus engineering effort, and decide the best way forward.

At that point, it was very clear to me the we should be encrypting-all-the-things-no-matter-what and I took for granted that I could easily explain the whys and hows very easily. However, after a first attempt, I felt I was not as clear as I would like to have been, and then I decided to read a bit more about the whole thing, structure and put things together in a blog post.

# SSL x TLS

[SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security) stands for Secure Socket Layer and is a cryptographic protocol that enables secure communication over computer networks. Basically when two computers exchange information in a secure way, let's say using [HTTP](https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol), they add an extra layer of security, using SSL, at the top of HTTP in order to encrypt the data being exchanged.

SSL 3.0 was deprecated in 2014 after it was found vulnerable to the [POODLE attack](https://en.wikipedia.org/wiki/POODLE). TLS, or Transport Layer Security is then the successor of SSL.

Today SSL and TLS are used interchangeably, but they both refer to TLS.

# Why HTTPS is important

Mainly because of 3 things.

## Confidentiality

How do I keep my things secret, like passwords, bank details, messages, etc.? This is what confidentiality is about: protecting people's data.

In 2010, when Facebook was still using insecure HTTP, [it was possible to use a Firefox extension called Firesheep to capture authentication cookies](https://www.theregister.co.uk/2010/10/25/firesheep_cookie_capture_peril/) within the same open network. So let's say you were sitting in a café using some insecure Wi-Fi connection. If someone else on that same café was using the Firesheep plugin, that person would be able to sniff network packages in transit, including yours. Since back in time authentication cookies were sent over insecure HTTP, it was possible to steal them. With someone else's authentication cookie in hands, an attacker could hijack someone else's Facebook session 😱

## Integrity

Integrity means that you receive the very same content someone else or a web server has sent to you (which means the message hasn't been modified). Lack of integrity makes you vulnerable to malicious tracking, injected malware, etc.

During the Tunisian Revolution, the Tunisian government injected keyloggers on Facebook login pages. As a consequence, [many Facebook credentials were stolen](http://www.businessinsider.com/tunisia-facebook-2011-1?IR=T). Changing the content of the page was only possible because Facebook did not serve the login page via HTTPS some time ago.

## Authenticity

Authenticity helps us to be sure we are really talking to who we think we are. It prevents [DNS hijacking](https://en.wikipedia.org/wiki/DNS_hijacking), [phishing](https://en.wikipedia.org/wiki/Phishing) and malicious host file entries.

# HTTPS fundamentals

## Certificate Authority

[Certificate Authority (CA)](https://en.wikipedia.org/wiki/Certificate_authority) is an entity that issues digital certificates. CAs work like a trusted third-party clients and servers can rely on.

A certificate certifies (as you might have guessed) the ownership of a [public key](https://en.wikipedia.org/wiki/Public-key_cryptography) by the named subject of the certificate (the organisation that owns a web domain). The client can then use that public key to encrypt the data it sends back to a server. In other words, a certificate is a piece of information clients, like web browsers, can use to validate the authenticity of a server and encrypt data.

The owner of a domain can request a certificate from a Certificate Authority which will validate its ownership. A client can then verify that a certificate provided by that domain is authentic if it trusts the CA that issued the certificate. Generally, operating systems keep a list of trusted Root Certificate Authorities. Many browsers use that list with exception of Firefox, which keeps its own list of trusted Root CAs.

In general, CAs will charge money to issue certificates. Although, nowadays it is possible to have certificates for free thanks to CAs like [Let's Encrypt](https://letsencrypt.org/).

### Certificate

Digital certificate, SSL certificate, SSL/TLS certificate, public key certificate and identity certificate all relate to the same thing and can be used interchangeably.

Generally, a certificate is a file that ends in .pem, .crt or .cer (more details on extensions [here](https://serverfault.com/a/9717)). This file contains a bunch of information, like:

- issuer (the Certificate Authority)
- valid date range (until when the certificate is valid)
- state, country of the organisation
- URL (the domain)
- the organisation (the owner of a domain)

If you open a certificate in a text editor you will see only bunch of random characters. That's because all the information it contains is encoded. You can decode a certificate using some online tool (like [SSL Shopper](https://www.sslshopper.com/certificate-decoder.html) or [SSL Decoder](https://ssldecoder.org/)) or using the [openssl](https://www.openssl.org/docs/manmaster/man1/openssl.html) command line tool:

```bash
$ openssl x509 -text -noout -in certificate.crt
```

#### Domain scopes

Different CAs can offer some or all the scopes listed below.

##### Single domain

When a public key is certified for a single domain, like www.mydomain.com. Note the subdomain www. In this case, the certificate will include and only be valid to that particular subdomain.

##### Wildcard

By using wildcard certificates we can certify multiple subdomains within the same domain, like `*.mydomain.com`.

##### Multi-domain

Many domains can use the same public key, therefore the same certificate: mydomain.com, myotherdomain.org, etc.

##### Unified Communications/Subject Alternative Names

Similar to multi-domain certificates, but primarily used by Microsoft for some of its products.

#### Validation Levels

Validation level is the effort a CA will perform to validate a domain. All types of validation will use the same key, same encryption. The more difficult the validation is, more money a CA can charge.

##### Domain Validation (DV)

Most common, cheapest, fastest, easiest. Basically verifies the public key and the domain name are related. An e-mail is generally sent to the domain's owner who needs to respond to validate they own the domain. The CA can also ask you to make a specific file publicly available on your website (since you own it, you should be able to do it), so that it can verify you really own the domain.

Worth mentioning that [Let's Encrypt offer domain validation](https://letsencrypt.org/docs/faq/#what-services-does-let-s-encrypt-offer).

##### Organisation Validation (OV)

Same validation as for DV, plus it confirms the authenticity of the organisation, by checking its physical address plus other things.

##### Extended Validation (EV)

Most expensive. Same as OV, plus one person will contact the organisation and talk to people there. That's mainly to check the business really exists and is really who they claim they are.

### Chain of trust

There are only a few Root Certificate Authorities in the world and that number is not enough to provide certificates to everyone. At that point, Intermediate CAs were created to issue more certificates. The Root CA will then vouche for an Intermediate CA who can issue certificates to particular domains. So if a web browser (or another client) trusts a Root CA, it will automatically trust the certificates issued by the Intermediate CAs that the Root CA has vouched for. This mechanism is called "Chain of trust".

### Self-signed certificates

Self-signed certificates are certificates that have not been approved by a CA. OK, so why/when should we use self-signed certificates?

Self-signed certificates are still certificates, so they still allow traffic to be encrypted. It's useful to use them in two main situations:

- if systems already trust each other (like 2 servers inside the same organisation), so no need to have a trusted third-party.
- in testing environments, during development time (sometimes it can be too much effort to set-up certificates throughout all your test environments);

Follow instructions on [this blog post](https://jamielinux.com/docs/openssl-certificate-authority/index-full.html) if you need to generate a root CA, certificates, etc.

## TLS handshake

[TLS handshake](https://en.wikipedia.org/wiki/Handshaking) is in fact an agreement between a client and a server on how they are going to communicate securely. The handshake, as the name suggests, happens when a secure connection is established.

When the client (which can be a browser for instance) connects to a server, it sends what we call in TLS handshake a Client Hello. Inside this request, the client communicates the highest version of TLS it will support, supported [cypher suits](https://en.wikipedia.org/wiki/Cipher_suite), etc. The server then responds with a Server Hello, confirming the TLS version plus cypher suits they both will use, but also providing the server's certificate back to the client. The server's certificate contains a public key plus other data related to the server's identity. The client will then verify the server's public key against its list of certificate authorities and if the certificate it still valid. If the public key was actually issued (or signed) by one of the CAs the client has on its list, the client then knows it is communicating to whom it thinks it is. Worth calling out this initial communication is not encrypted yet, it's just a negotiation phase. A man-in-the-middle could potentially eavesdrop what's being transmitted. Since there's no content being exchanged yet, that's OK.

Next, the client will perform a Key Exchange with the server. This request from client to server is now encrypted using the server's public key (so only the server can decrypt the request using the corresponding private key). During this phase, client and server will agree on a password they can both use to encrypt data. The server will respond with a Server Finished response, and at that point the communication can begin. Once they have both the same password, they can switch to [symmetric-key cryptography](https://en.wikipedia.org/wiki/Symmetric-key_algorithm). Worth calling out this password is temporary and attached to that particular [session](https://en.wikipedia.org/wiki/Session_(computer_science)).

But hang on, why switch back to symmetric-key and not keep using public-key cryptography? It's mainly to take advantage of both worlds. Public-key cryptography allows us to communicate private information in public, but the algorithms involved can be a bit slow. Symmetric-key cryptography makes it difficult to agree on a shared password publicly, but algorithms are faster.

Obviously, this is a high level explanation of TLS handshake. There is much more details related to TLS negotiation, which are not covered here. However, that's enough details my Product Manager needs to know at this level.

For more details check out [this nice TLS connection illustration](https://tls.ulfheim.net/).

# More on HTTPS, TLS

Here's a list of resources I personally recommend if you want to go beyond the high level concepts of HTTPS:

- [Director SSL Certificate Configuration with OpenSSL](https://bosh.io/docs/director-certs-openssl/)
- [TLS Basics](https://www.internetsociety.org/deploy360/tls/basics/)
- [Public key cryptography – Diffie-Hellman Key Exchange (full version)](https://www.youtube.com/watch?v=YEBfamv-_do)
- [Public Key Cryptography: RSA Encryption Algorithm](https://www.youtube.com/watch?v=wXB-V_Keiu8)
- [A Detailed Look at RFC 8446 (a.k.a. TLS 1.3)](https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/)
- [RFCs related to TLS](https://tools.ietf.org/wg/tls/)
- [Percentage of Web Pages Loaded by Firefox Using HTTPS](https://letsencrypt.org/stats/#percent-pageloads)
- [The Changelog – Episode #243 Let's Encrypt the Web](https://changelog.com/podcast/243)
- [SSL Certificates for Web Developers](https://www.linkedin.com/learning/ssl-certificates-for-web-developers) course on LinkedIn Learning
- [Can I use](https://caniuse.com/) (website that tells you which security features your web browser supports)
- [An overview of the SSL or TLS handshake](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm)
- [OpenSSL Essentials: Working with SSL Certificates, Private Keys and CSRs](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs/)
- [What Every Developer Must Know About HTTPS](https://app.pluralsight.com/library/courses/https-every-developer-must-know/table-of-contents) course on Pluralsight (you can alternatively check [Troy Hunt](https://twitter.com/troyhunt) who publishes a lot of good content about TLS)
