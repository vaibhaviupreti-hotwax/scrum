## Wavepicking: 
- To decide high-level scheduling strategies for picking(to optimise the process of picking)
- Picking is done in bulk and within a time slot for volume operations; high sorting is required. here.
- Reduced travel time, better labor utilization, and fewer bottlenecks. 

## SSO
- Single sign On
- based on federated identity, without login each time.
- with the help of identity provider(sahre ID among services).
- has 2 protocols : SAML, Open ID connect
- SAML: Simple Assertion Markup Language - xml based - for exchanging identity info between services. Found in common work environmnt.
- Open ID - used at enterprise level like - gmail, workflow, slack etc.
- follows interactive process between client/browser, service provider/gmail, identity provider. 
- Atlassion crowd - example of SSO application: Atlassian Crowd enables Single Sign-On (SSO) across Atlassian products (Jira, Confluence, Bitbucket) and external applications by centralizing identity management. A typical example is a user logging into Jira (jira.example.com), which authenticates against Crowd, granting them an active session that automatically logs them into Confluence (confluence.example.com) without re-entering credentials.

## encryption algorithms
Encryption algorithms are mathematical procedures that transform data (plaintext) into an unreadable format (ciphertext) using keys to ensure confidentiality. They are primarily divided into symmetric (single key for encryption/decryption) and asymmetric (public-private key pair) types. 

There are also two important encryption modes:
- **Two-way encryption** (reversible): the original plaintext can be recovered from ciphertext using the correct key. Example: protecting customer payment data in a database so the application can decrypt it when processing orders.
- **One-way encryption** (irreversible): data is transformed into a hash or digest, and the original plaintext cannot be recovered. Example: hashing employee passwords in an HR system so logins can be verified without storing raw passwords.

Common robust algorithms include AES (used by governments), RSA (for secure transmission), and ECC (efficient for IoT).
