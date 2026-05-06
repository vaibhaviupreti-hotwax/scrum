# Session 10:30 - Notes

## Discussed about:

- **User_Login entity** - party ID
- **Security_permission entity** - all permissions defined: pick/pack/view permissions
- **Security_Group** - different type of user - admin, picker
- **Security_Group_Permission ASSOC**
- **UserLoginSecurityGroup** — when user is linked to security group—group ID SUPER—user ke according
- It is an Example of a party classification group entity

---

## Ashish sir talked about 

### Setup to do asap
- 24-09 ofbiz setup - challenges
- Postman REST API testing 

### Authentication 
- A deprecated approach is asking Security questions - their management is tricky. 
- Same goes for password history. No need to manage now

### SSO
- Login on standalone app vs sso app for multiple applications. Why came?
- Existing software for SSO?

---

## Read more:

### Microservices architecture ? 
- How data is communicated/shared/interacted ?
- Communication protocol - socket io etc/http too.. 
- Forms of data sharing - json/xls/plain-text/xml
- Http status codes. Client server architecture data response/code - 200/300
- OFbiz using REST - communicate with - magento / shopify /others

### Token generation and calling REST api
- User login entity importance understanding
- token/security key concept
- Permission: role based access - component specific and super permission

### Entities and data + business in assignment
- Jira, Atlassian, Confluence uses ofbiz in Entity and Service engine. 
- Atlassion crowd - example of SSO application (how it really works in ofbiz etc..)

### Infrastructure
- 90-95% servers are set on LINUX - package: Lightweight directory access protocol

### Additional Topics
- Read about Disable flag for user 

### Important Links
- https://github.com/apache/ofbiz-framework 
- Read about Gradlew build tool - why is it needed? + related commands - load all, clean all..

### Upcoming Learning
- User login and security aspect - framework - behind the scene work. - java file - security - crud - all permissions- how managed on FW side..?  - important - to think next level..
- User login - password - encrypted - SHA algo 256/512 - 1 way and 2 way encryption
- Encryption algorithms are mathematical procedures that transform data (plaintext) into an unreadable format (ciphertext) using keys to ensure confidentiality. They are primarily divided into symmetric (single key for encryption/decryption) and asymmetric (public-private key pair) types. Common robust algorithms include AES (used by governments), RSA (for secure transmission), and ECC (efficient for IoT).

### Assignments
- **Parnika mam Assignment:**
  - Next day: Product data model..  
  - Data model 3rd chapter
  - Party data model assignment - weekend takk

---

## Caution
- Next gen shopify shop access token - don't use anywhere
