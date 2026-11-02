\# VASTRIX — Product Requirements



\## 1. Document Purpose



This document defines the product requirements and intended behavior of VASTRIX.



It describes:



\* What VASTRIX is

\* Who it is for

\* The intended MVP

\* Core user journeys

\* Transfer capabilities

\* Security capabilities

\* Recipient experience

\* User experience requirements

\* Future product capabilities



This document describes the product requirement, not necessarily the current implementation.



Implementation status must be checked against:



\* Actual source code

\* Database migrations

\* Application configuration

\* Technical documentation

\* `PROJECT\_MEMORY/09\_CURRENT/`



\---



\# 2. Product Identity



\## Product Name



VASTRIX



\## Product Category



Secure large-file transfer platform.



\## Product Vision



VASTRIX is intended to provide a simple and secure way to send large files to another person through protected, temporary sharing links.



The product should make sending large files as easy as:



```text

Select Files

&#x20;   ↓

Configure Transfer

&#x20;   ↓

Upload

&#x20;   ↓

Get Secure Link

&#x20;   ↓

Share Link

&#x20;   ↓

Recipient Verifies Access

&#x20;   ↓

Recipient Downloads

```



\---



\# 3. Primary Product Objective



The primary objective is to allow users to securely transfer large files without requiring the sender or recipient to use traditional cloud-drive collaboration features.



The system should prioritize:



\* Simplicity

\* Security

\* Large-file support

\* Temporary sharing

\* Easy recipient access

\* Minimal manual administration

\* Clear user experience

\* Reliable file delivery



\---



\# 4. Target Use Case



A typical user should be able to upload one or more large files and generate a secure link that can be shared with another person.



Example:



```text

Sender

&#x20; ↓

Selects 1 or more large files

&#x20; ↓

Creates transfer

&#x20; ↓

Uploads files

&#x20; ↓

VASTRIX creates secure link

&#x20; ↓

Sender shares link

&#x20; ↓

Recipient opens link

&#x20; ↓

Recipient completes required verification

&#x20; ↓

Recipient downloads files

```



The recipient should ideally not need to create a full VASTRIX account merely to download a file.



\---



\# 5. MVP Scope



The intended MVP should focus on secure file transfer.



Core MVP capabilities:



\* User registration

\* User login

\* Large-file upload

\* Multiple-file transfer

\* Secure transfer links

\* Transfer expiration

\* Optional recipient verification

\* Optional password protection

\* Secure downloads

\* Download controls

\* Storage quota

\* Email notifications

\* Basic transfer management



\---



\# 6. Features Outside Initial MVP



VASTRIX should not initially attempt to become a complete cloud-drive or collaboration platform.



The following should not be treated as core MVP requirements:



\* Full cloud-drive replacement

\* Real-time document collaboration

\* Document editing

\* Enterprise team collaboration suite

\* Complex project management

\* Extensive file synchronization

\* Desktop drive replacement



These may be considered in future product versions if explicitly required.



\---



\# 7. User Types



\## Guest



A visitor who has not created or authenticated a VASTRIX account.



A guest may:



\* Visit the public website

\* Learn about VASTRIX

\* Access publicly available product information



A guest does not automatically receive access to private user operations.



\---



\## User



An authenticated VASTRIX customer.



A User should eventually be able to:



\* Register

\* Log in

\* Create transfers

\* Upload files

\* Configure transfer options

\* View their transfers

\* Share transfer links

\* Monitor transfer status

\* Manage downloads

\* Manage their account



\---



\## Recipient



A person receiving a shared transfer.



The intended recipient experience should be lightweight.



The recipient should ideally be able to:



\* Open a secure transfer link

\* View permitted transfer information

\* Complete required verification

\* Download permitted files



A recipient should not need a full VASTRIX account for normal file receipt.



\---



\## Admin



An administrative user responsible for system-level management.



Potential admin capabilities include:



\* System health monitoring

\* User management

\* Transfer monitoring

\* Storage monitoring

\* Security monitoring

\* Audit-log review

\* Operational troubleshooting



The exact administrative feature set remains subject to implementation.



\---



\# 8. Account Requirements



A User should register using:



\* Username

\* Email

\* Password



Passwords must never be stored in plaintext.



The system should use secure password hashing.



User accounts may be disabled by authorized administrative functionality.



\---



\# 9. User Authentication



Authenticated users should log in before performing protected user operations.



Authentication is intended to protect:



\* Transfer creation

\* Upload operations

\* User transfer listing

\* User-specific resources

\* Administrative resources



The authenticated identity should determine ownership.



A client should not be able to simply provide another user's ID and become the owner of a transfer.



\---



\# 10. Transfer Concept



A Transfer represents a secure delivery containing one or more files.



A transfer should have:



\* Unique identity

\* Secure sharing token

\* Owner where applicable

\* Creation timestamp

\* Expiration

\* Optional custom message

\* Security configuration

\* One or more files



The transfer is the main security boundary between the sender and recipient.



\---



\# 11. Multiple Files



A single transfer should support multiple files.



Example:



```text

Transfer

├── Contract.pdf

├── Invoice.xlsx

├── Photos.zip

└── Documentation.docx

```



The recipient should be able to view the files associated with the transfer and download the permitted files.



\---



\# 12. Large File Support



Large files are a primary purpose of VASTRIX.



The initial implementation targets files up to approximately:



\*\*5 GiB per file\*\*



The exact limits are controlled by the backend and product plan.



The frontend may provide earlier client-side validation for usability, but backend validation remains authoritative.



\---



\# 13. Upload Requirements



The upload experience should support:



\* File selection

\* Multiple files

\* Drag and drop

\* Upload progress

\* Large-file handling

\* Resumable upload architecture

\* Upload validation

\* Storage quota enforcement



The system should avoid loading entire large files into application memory unnecessarily.



\---



\# 14. Resumable Upload



Large uploads should use a resumable upload mechanism.



The current backend uses TUS/tusdotnet.



The intended behavior is to allow large uploads to be transferred in chunks rather than requiring one massive request.



This improves reliability for large files and unstable network connections.



\---



\# 15. Transfer Expiration



Transfers should expire automatically.



The intended product expiration options are:



```text

1 day

3 days

7 days

14 days

30 days

```



Expiration should prevent access after the transfer has expired.



Expired transfers should eventually be cleaned up.



\---



\# 16. Secure Sharing Link



Each transfer should have a secure, difficult-to-guess sharing token.



The intended recipient URL pattern is conceptually:



```text

https://<vastix-domain>/t/<secure-token>

```



The actual production domain is not defined in this document.



The transfer link should not expose database IDs unnecessarily.



\---



\# 17. Link Sharing



The sender should be able to share the transfer link using convenient mechanisms such as:



\* Copy link

\* Email

\* WhatsApp

\* Native device sharing

\* Other supported sharing mechanisms



The exact integrations may evolve.



\---



\# 18. Recipient Access



The recipient should open the transfer link without necessarily creating an account.



The system should then determine whether additional verification is required.



Possible future flow:



```text

Open Link

&#x20;   ↓

Transfer Validation

&#x20;   ↓

Password / Recipient Verification if required

&#x20;   ↓

Authorized Transfer Page

&#x20;   ↓

Download

```



\---



\# 19. Recipient Verification



Recipient verification is intended as an optional security feature.



When enabled, the sender should be able to require the recipient to verify access using email.



Conceptual flow:



```text

Recipient opens transfer

&#x20;       ↓

Recipient provides email

&#x20;       ↓

VASTRIX generates verification code

&#x20;       ↓

Code is sent by email

&#x20;       ↓

Recipient enters code

&#x20;       ↓

Code is validated

&#x20;       ↓

Recipient receives temporary verified access

&#x20;       ↓

Download becomes available

```



Verification codes should:



\* Be securely generated

\* Be stored as hashes rather than plaintext

\* Expire

\* Have limited attempts

\* Be single-use

\* Be associated with the relevant transfer

\* Be protected against abuse



\---



\# 20. Recipient Email



The recipient's email should be associated with the verification process where recipient verification is enabled.



The email should not automatically imply that the recipient needs a VASTRIX account.



\---



\# 21. Password Protection



Transfers should eventually support optional password protection.



Conceptual flow:



```text

Recipient opens link

&#x20;       ↓

Password required

&#x20;       ↓

Correct password

&#x20;       ↓

Continue to transfer

```



Passwords should never be stored in plaintext.



The backend must enforce password protection.



The frontend must not be treated as the security boundary.



\---



\# 22. Download Controls



VASTRIX should eventually support controls such as:



\* Maximum download count

\* Download counter

\* Expiration

\* Recipient verification

\* Password protection



Example:



```text

Maximum downloads = 5

```



After the permitted number of downloads is reached, additional downloads should be denied.



These controls must be enforced server-side.



\---



\# 23. Download Experience



The recipient download page should provide:



\* Transfer information

\* Sender message where provided

\* File names

\* File sizes

\* Expiration information

\* Required verification state

\* Required password state

\* Download controls



The experience should remain simple and focused.



\---



\# 24. Download Security



Downloads should validate at minimum:



\* Transfer token

\* Transfer existence

\* Transfer deletion state

\* Transfer expiration

\* File association

\* File existence

\* Path/object ownership

\* Any required recipient verification

\* Any required password

\* Any download-limit restrictions



Future object-storage architecture should use short-lived signed download URLs rather than permanent public file URLs.



\---



\# 25. Storage



Large file contents should not be stored inside PostgreSQL.



PostgreSQL should primarily store metadata.



Current development storage uses the local filesystem.



The long-term architecture is intended to move toward private object storage.



Conceptually:



```text

Large File

&#x20;   ↓

Private Object Storage



Metadata

&#x20;   ↓

PostgreSQL

```



\---



\# 26. Storage Quota



Users should have storage quotas.



The current development default is approximately:



\*\*5 GiB\*\*



Quota enforcement should occur server-side.



The system should account for active stored files when determining whether a new upload is permitted.



Future subscription plans may have different limits.



\---



\# 27. Product Plans



Potential product plans include:



\## Free



Conceptual limits:



\* Up to 2 GB storage

\* Up to 7-day transfer expiration



\## Pro



Conceptual limits:



\* Up to 20 GB

\* Up to 30-day expiration



\## Business



Conceptual limits:



\* 100 GB+

\* Extended storage and transfer capabilities



These plan limits are product planning information and are not necessarily implemented in the current system.



\---



\# 28. Transfer Status



The intended product lifecycle includes:



```text

PENDING

UPLOADING

UPLOADED

SCANNING

AVAILABLE

QUARANTINED

FAILED

EXPIRED

DELETED

```



Meaning:



\### PENDING



Transfer has been created but upload has not started or completed.



\### UPLOADING



One or more files are currently being uploaded.



\### UPLOADED



Files have completed upload but may still require processing.



\### SCANNING



Files are undergoing security scanning.



\### AVAILABLE



Files are ready for recipient download.



\### QUARANTINED



A security process has prevented normal access.



\### FAILED



Processing or upload failed.



\### EXPIRED



The transfer passed its expiration time.



\### DELETED



The transfer has been removed from active availability.



The current backend does not yet implement this complete status model.



\---



\# 29. File Security



Future VASTRIX architecture should support malware scanning.



Conceptual lifecycle:



```text

Upload

&#x20; ↓

Security Scan

&#x20; ↓

Safe

&#x20; ↓

AVAILABLE

```



or:



```text

Upload

&#x20; ↓

Security Scan

&#x20; ↓

Threat Detected

&#x20; ↓

QUARANTINED

```



Malware scanning is planned functionality.



\---



\# 30. Automatic Cleanup



Expired transfers should eventually be automatically cleaned.



Cleanup should:



\* Detect expired transfers

\* Remove associated physical/object-storage files

\* Update transfer state

\* Prevent future access

\* Handle failures safely

\* Detect orphaned files where possible



Cleanup should not accidentally delete files belonging to another transfer.



\---



\# 31. Email Notifications



The product may provide email notifications for:



\* Transfer sharing

\* Recipient verification

\* Download notifications

\* Other important transfer events



Notification reliability and retry behavior may be improved in future versions.



\---



\# 32. User Dashboard



A future authenticated dashboard should allow users to manage their transfers.



Potential information:



\* Active transfers

\* Expired transfers

\* Upload progress

\* Transfer creation date

\* Expiration date

\* File count

\* File sizes

\* Download activity

\* Transfer status



The current frontend does not yet contain a dashboard.



\---



\# 33. Admin Experience



A future admin interface may provide:



\* System health

\* User management

\* Transfer monitoring

\* Storage statistics

\* Cleanup status

\* Security events

\* Operational metrics

\* Audit logs



The current backend has basic health endpoints, but a full admin UI is not implemented.



\---



\# 34. Security Requirements



VASTRIX should prioritize:



\* Secure authentication

\* Secure password storage

\* Authorization

\* User isolation

\* Secure transfer tokens

\* Expiration enforcement

\* Recipient verification

\* Password protection

\* Download limits

\* Path traversal protection

\* Upload validation

\* Storage quota enforcement

\* Rate limiting

\* Audit logging

\* Malware scanning

\* Private storage

\* Short-lived download authorization



\---



\# 35. User Data Isolation



Users must only be able to manage resources they are authorized to access.



For example:



```text

User A

&#x20;  ↓

Own Transfers Only

```



User A must not be able to access User B's private transfers simply by changing an identifier in a request.



This is a core authorization requirement.



\---



\# 36. Security Boundary



The backend is the authoritative security boundary.



Frontend checks are for usability.



For example:



```text

Frontend

&#x20;   ↓

UX validation

&#x20;   ↓

Backend

&#x20;   ↓

Security validation

&#x20;   ↓

Database / Storage

```



A malicious client can bypass frontend checks, so every important security requirement must be enforced server-side.



\---



\# 37. Privacy



VASTRIX should minimize unnecessary exposure of sensitive information.



Sensitive data should not be:



\* Logged unnecessarily

\* Exposed through APIs

\* Included in URLs unnecessarily

\* Stored in plaintext when hashing is appropriate

\* Returned to unauthorized users



Secrets must remain outside source control.



\---



\# 38. Accessibility



The frontend should target:



\*\*WCAG 2.2 AA\*\*



The UI should consider:



\* Keyboard navigation

\* Focus states

\* Screen readers

\* Color contrast

\* Accessible labels

\* Error messaging

\* Responsive layouts

\* Reduced-motion preferences



\---



\# 39. User Experience



The product should feel:



\* Modern

\* Clean

\* Fast

\* Secure

\* Simple

\* Professional

\* Trustworthy



The sender should not need to understand the technical upload architecture.



The interface should explain important security choices clearly.



\---



\# 40. Visual Direction



The current frontend uses a dark/violet visual direction.



Current design technologies include:



\* Tailwind CSS

\* Framer Motion

\* Lucide icons



The visual design may evolve, but changes should preserve usability and accessibility.



\---



\# 41. SEO Requirements



The public website should eventually target search queries related to:



\* Large file transfer

\* Send large files

\* Secure file sharing

\* Secure large-file transfer

\* Temporary file sharing



Planned SEO pages include concepts such as:



```text

/large-file-transfer

/send-large-files

/secure-file-sharing

```



SEO content should remain useful and not be artificially keyword-stuffed.



\---



\# 42. Responsive Design



The frontend should work across:



\* Desktop

\* Laptop

\* Tablet

\* Mobile



Large-file transfer should remain usable on smaller screens.



\---



\# 43. Reliability Requirements



Large uploads should be designed to tolerate:



\* Temporary network interruptions

\* Browser instability

\* Large file sizes

\* Slow networks

\* Partial upload failures



Resumable upload technology is therefore an important part of the architecture.



\---



\# 44. Product Principles



VASTRIX should follow these product principles:



\### Simple



The user should not need technical knowledge.



\### Secure



Security controls should be enabled and enforced at the appropriate layer.



\### Temporary



Transfers should naturally expire.



\### Private



Files should remain private unless explicitly authorized.



\### Large-file focused



The product should be optimized for large file delivery.



\### Minimal operations



The platform should require minimal manual intervention.



\### Portable



The architecture should support future migration from local storage to object storage.



\---



\# 45. Current Product Reality



The product requirements described above are not equivalent to current implementation.



Current reality includes:



\* Backend authentication implemented

\* Transfer creation implemented

\* TUS upload implemented

\* Local storage implemented

\* Secure transfer tokens implemented

\* Transfer expiration implemented

\* Basic anonymous download implemented

\* Recipient verification foundation partially implemented

\* Frontend landing/transfer interface implemented

\* Frontend download page implemented



However, several intended product capabilities remain incomplete.



\---



\# 46. Major Product Gaps



Important gaps include:



\* Complete frontend authentication

\* JWT integration in transfer creation

\* JWT integration in TUS upload

\* Complete recipient verification

\* OTP generation

\* OTP validation

\* Recipient verification enforcement

\* Transfer password

\* Maximum download count

\* Download counter

\* Transfer status

\* Object storage

\* Signed download URLs

\* Malware scanning

\* Full dashboard

\* Admin UI

\* Advanced audit logging

\* Production-grade operational infrastructure



\---



\# 47. Product Development Rule



A planned feature must not be represented as an implemented feature.



Use the following terminology:



```text

IMPLEMENTED

PARTIALLY IMPLEMENTED

PLANNED

MISSING

BLOCKED

DEPRECATED

```



Product requirements describe intended behavior.



Technical documentation and source code determine actual implementation status.



\---



\# 48. Future Evolution



Possible future capabilities include:



\* SMS verification

\* Magic-link verification

\* Advanced account security

\* Multi-factor authentication

\* More storage plans

\* Advanced analytics

\* Team/business accounts

\* Organization management

\* API access

\* Webhooks

\* Object-storage lifecycle policies

\* Distributed workers

\* Advanced monitoring

\* Billing/subscriptions



These should not be implemented unless explicitly approved as product requirements.



\---



\# 49. Product Success Criteria



A successful VASTRIX MVP should allow a normal user to complete the following journey reliably:



```text

Register

&#x20;  ↓

Login

&#x20;  ↓

Create Transfer

&#x20;  ↓

Select Large File(s)

&#x20;  ↓

Configure Expiration

&#x20;  ↓

Configure Security

&#x20;  ↓

Upload

&#x20;  ↓

Receive Secure Link

&#x20;  ↓

Share Link

&#x20;  ↓

Recipient Opens Link

&#x20;  ↓

Recipient Completes Required Security

&#x20;  ↓

Recipient Downloads

&#x20;  ↓

Transfer Eventually Expires

&#x20;  ↓

Files Are Cleaned Up

```



Every stage should be secure, understandable, and observable.



\---



\# 50. Final Product Principle



VASTRIX should remain focused on one primary problem:



\*\*Securely sending large files from one person to another with minimal complexity.\*\*



Additional capabilities should support this core purpose rather than turning the MVP into an unnecessarily complex cloud-storage platform.



