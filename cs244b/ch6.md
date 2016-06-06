Accounts and Authentication
---------------------------

**Account** -> object that represents a principal
  * Distributed object, as all nodes in network need to check access control

**Role-Based Access Control**: Multiple accounts (group) assigned single role, privileges based on roles.

Capabilities: alternative to ACLs, where holder of capability has privileges
  * Problem: No way to be sure that the capability was gotten legitimately.  Stolen? Forged?

One way to do capabilities: make a connection a capability, then
  * Revocation easy: terminate connection
  * Easy to manage
  * Problem: cost of 1 connection per capability is expensive

In general, capabilities are very expensive

# Encryption

Attempts to give you three things:

  1. Confidentiality: only designated receiver can read messages
    * Usually by encryption
  2. Integrity: message cannot be altered unnoticed
    * Usually by calculating digest, then signing with private key (MAC)
  3. Authentication: can prove that this message came from the expected sender

Timeliness is a concern
  * Solved via timestamps (logical or real)
  * Also with one-time nonce that you check for

## Shared vs PubKey Encryption

3 parts of PKI:
  1. Lifetime
  2. Selection
  3. Distribution

"Secret key is like a tomato"
  * Can be very careful when selecting it, keep it safe, but sooner or later
    it will go bad, then you need to toss it.
  * Keys necessarily have finite (short) lifetimes. Get weaker over time, need refresh


  * Secure selection process absolutely essential for system to work
  * Lifetimes are short
  * Distribution mechanism most malleable

### Approaches to Distribution

1. External Channel
  * Key-signing parties
  * Word of mouth
  * **Inefficient, inconvenient**
2. Chain key distribution
  * In the communication protocol, parties agree on new secret key using existing
    secret key
  * **No Forward Secrecy:** break any old key in the past -> current and all future keys
3. Multi-level key distribution
  * One channel for distribution covered by a master key, separate data distribution
    channel with many ephemeral keys
  * Problem: single point of failure

**Claim**: We are __always__ forced back to External Channel key distribution

### Trusted third party

In normal P2P scheme, need to have O(N^2) separate key exchanges. Not scalable.

Alternative: provide centralized **key server** with secure connections to all nodes
  * Now have O(N) secure exchanges
  * a.k.a. **key distribution center**

Example usage:
  * Client C, Server S want to connect.
  * KDC gives secret key to both of S and C for communication.
    * Must trust that KDC only gave key to other party, no one else
    * In PKE, provides PubKey of C to S, PubKey of S to C

### Optimizing Authentication Service
  * Requesting keys from KDC for every secure conn. is expensive
  * Solution: **authenticator**
    * Conversation key + proof came from trusted 3rd party

# Authentication Schemes

Need to ensure parties communicating with KDC are valid principals and identities match
what they say. 3 schemes:

  1. Password: Principal supplies password (or key or authenticator)
  2. Challenge-Response: Principal proves identity by encrypting/decrypting chosen
     plaintext/ciphertext
  3. Transitive: Vouched for by another trusted third party

Problem with Password:
  * Principal must be sure sending password to true KDC, not an imposter
  * Passwords must be sent over confidential channel, recursive problem

Typical Challenge-Response:
  * Principal sends some encrypted data to Auth service to prove they know the key
  * Auth service sends back Ticket that requires secret key to decrypt + use

Problem with challenge-response:
  * Replay: eavesdropper can play back the initial send to create new sessions if
    keys are not short lived
  * Solution: non-reused ID's (sequence numbers, timestamps) with each response
  * Variante for PKE: Send encrypted with pubkey of principal, expect encrypted with
    KDC pubkey

## Kerberos

  * Client gets ticket for server
  * Uses ticket to build authenticator, sends request with ticket and authenticator
  * Server checks authenticator matches ticket info, sends [timestamp+1]Kc,s to client
  * Ticket granting service: Returns new session key + ticket for service
  * Lifetime of ticket limited by timestamp
    * Susceptible to insecure clock synchronization

## X.509

One-way auth:
  * P sends signed msg with nonce to avoid replay
  * Q securely gets pubkey for P, verifies sig + integrity

Two-way:
  * Extension similar to Kerberos


# Digital Signatures

Problem: prove principal really sent message M

Solution: __Digital Signature__. Encrypt with privkey

Sender cannot deny writing the message without exposing privkey.

Problem: Signature only good until key is exposed

Solution: Private key is not known by even the signer
  * Gets signature from the KDC
  * Main approach: ask trusted KDC
  * Alternative: certificates

# Certificate-Based Authentication

Certificate: signed assertion
  * eg. "This PubKey belongs to Andrew until June 7, 2016"
  * Signed by Certificate Authority

Revocation:
  * Distributed __certificate revocation list__ periodically sent out,
    warning about known bad certs
  * Cheriton believes it's cheaper to ask the CA each time to verify a cert, rather
    than having to worry about pulling revocations

Problems with certificates:
  * Revocation is difficult and expensive
    * Using revocation lists, hard to upper-bound the time to revoke something
  * Still relies on a long-lasting key (for the root CA), which will become "bad tomato"
  * Poor caching behavior: certificates must have long lifetimes to be useful caching
    mechanisms, but that conflicts with security.

More challenges:
  * Global CA's have failed due to liability (this must predate Let's Encrypt)
  * Very expensive to run CA infra, strong security needs
  * Customer perception: making people pay for service to prove I am me

Certification vs Authorization
  * Driver's License, Credit Card -> identify AND authorize
  * In DS, services are networked, so certification and authorization can be separated

# Large-Scale Authentication

**Relative Authentication**: KDC's peer to each other, and so clients can be
vouched by the KDC that they are authenticated with.

# Naming vs Authentication

Conventional wisdom calls for secured/trusted name server
  * Impossible to scale: requires trusting every node (or some subset)

Better: make the naming service highly available, highly observable
  * Distribute naming to all routers, so that attackers cannot compromise
    service without getting significant portion of the routers
  * Decoupling auth from directory service, now if you go to wrong host due to
    compromised dir service, you just fail auth.
  * Compromise becomes just a local unavailability

# Open Security, Physical Security, and Availability

Secrecy looks bad
  * Single point of failure (root key)
  * No way to prove it has been broken

People in the industry have confused __security__ with __confidentiality__
  * Many applications don't need confidentiality, but __availability__

Open Security: Availability first
  * Each routers holds naming/directories
  * Messages are exclusively exchanged through multicast
  * Redundancy: separate power sources, separate ISPs, possibly separate admins
  * Only use confidentiality for those applications that truly need it
