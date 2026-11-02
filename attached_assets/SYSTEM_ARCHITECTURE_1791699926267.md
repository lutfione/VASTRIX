\# VASTRIX — System Architecture



\## 1. Purpose



This document provides the overall architectural view of VASTRIX.



It explains how the major parts of the system interact:



\* Frontend

\* Backend API

\* Authentication

\* PostgreSQL

\* File storage

\* TUS upload processing

\* Transfer management

\* Recipient access

\* Email notifications

\* Background workers

\* Future security and infrastructure components



This document intentionally stays at the \*\*system level\*\*.



Detailed implementation information belongs in the appropriate technical documentation under:



```text

docs\\

```



The PROJECT\_MEMORY version exists to help humans and AI assistants understand the system as a whole.



\---



\# 2. System Overview



VASTRIX is a secure large-file transfer platform.



The core purpose is:



```text

Authenticated user

&#x20;      │

&#x20;      ▼

Select large files

&#x20;      │

&#x20;      ▼

Create secure transfer

&#x20;      │

&#x20;      ▼

Upload files

&#x20;      │

&#x20;      ▼

Generate secure transfer link

&#x20;      │

&#x20;      ▼

Share link with recipient

&#x20;      │

&#x20;      ▼

Recipient accesses transfer

&#x20;      │

&#x20;      ▼

Recipient downloads files

```



The application is designed around temporary, controlled file transfers rather than permanent cloud-drive storage.



\---



\# 3. High-Level Architecture



The current architecture is:



```text

&#x20;                        ┌──────────────────────┐

&#x20;                        │      User Browser    │

&#x20;                        │                      │

&#x20;                        │ React + Vite         │

&#x20;                        │ React Router         │

&#x20;                        │ Tailwind             │

&#x20;                        └──────────┬───────────┘

&#x20;                                   │

&#x20;                   HTTP / JSON / TUS

&#x20;                                   │

&#x20;                                   ▼

&#x20;                        ┌──────────────────────┐

&#x20;                        │    VASTRIX API       │

&#x20;                        │                      │

&#x20;                        │ ASP.NET Core / .NET 9│

&#x20;                        │ JWT Authentication   │

&#x20;                        │ Transfer API         │

&#x20;                        │ TUS Upload Handling  │

&#x20;                        │ Download Handling    │

&#x20;                        │ Email Services       │

&#x20;                        └───────┬───────┬──────┘

&#x20;                                │       │

&#x20;                    EF Core     │       │ Files

&#x20;                                │       │

&#x20;                                ▼       ▼

&#x20;                        ┌────────────┐ ┌──────────────────┐

&#x20;                        │ PostgreSQL │ │ Local Filesystem │

&#x20;                        │            │ │                  │

&#x20;                        │ Metadata   │ │ C:\\VASTRIX\_     │

&#x20;                        │ Users      │ │ FileStore        │

&#x20;                        │ Transfers  │ │                  │

&#x20;                        │ Files      │ │ Large files      │

&#x20;                        │ Challenges │ │                  │

&#x20;                        └────────────┘ └──────────────────┘

```



\---



\# 4. Major System Components



VASTRIX currently consists of the following major components.



| Component          | Technology            | Purpose               | Status                |

| ------------------ | --------------------- | --------------------- | --------------------- |

| Frontend           | React + Vite          | User interface        | Implemented           |

| Routing            | React Router          | Browser routing       | Implemented           |

| Styling            | Tailwind CSS          | UI styling            | Implemented           |

| Animation          | Framer Motion         | UI animation          | Implemented           |

| Backend API        | ASP.NET Core / .NET 9 | Business/API layer    | Implemented           |

| Authentication     | JWT + BCrypt          | User authentication   | Implemented           |

| ORM                | Entity Framework Core | Database access       | Implemented           |

| Database           | PostgreSQL            | Metadata/persistence  | Implemented           |

| Upload protocol    | TUS / tusdotnet       | Resumable uploads     | Implemented           |

| File storage       | Windows filesystem    | Physical file storage | Implemented           |

| Email              | MailKit / SMTP        | Notifications         | Partially implemented |

| Background cleanup | Hosted worker         | Expired-file cleanup  | Implemented           |

| Object storage     | S3-compatible/etc.    | Scalable storage      | Planned               |

| Redis              | Redis                 | Caching/coordination  | Planned               |

| Queue              | Queue system          | Async processing      | Planned               |

| Malware scanning   | Scanner               | File security         | Planned               |

| Monitoring         | Monitoring stack      | Operations            | Planned               |



\---



\# 5. Backend Project Architecture



The backend solution is located at:



```text

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\

```



The solution contains:



```text

VASTRIX.slnx

│

├── VASTRIX.Core

├── VASTRIX.Infrastructure

└── VASTRIX.API

```



Dependency direction:



```text

VASTRIX.API

&#x20;    │

&#x20;    ├──────────────► VASTRIX.Infrastructure

&#x20;    │

&#x20;    └──────────────► VASTRIX.Core



VASTRIX.Infrastructure

&#x20;    │

&#x20;    └──────────────► VASTRIX.Core



VASTRIX.Core

&#x20;    │

&#x20;    └── No project dependency

```



The dependency direction is intentionally kept toward the Core project.



\---



\# 6. Backend Layer Responsibilities



\## 6.1 VASTRIX.Core



Core contains domain-level models and concepts.



Current entities include:



```text

User

UserRole

Transfer

TransferFile

RecipientVerificationChallenge

```



Core should not depend on infrastructure-specific implementation details.



\---



\## 6.2 VASTRIX.Infrastructure



Infrastructure contains implementation details such as:



\* Entity Framework Core.

\* PostgreSQL configuration.

\* Database context.

\* Migrations.

\* Hosted/background workers.

\* Email infrastructure.

\* Infrastructure services.



\---



\## 6.3 VASTRIX.API



The API project is responsible for:



\* HTTP endpoints.

\* Authentication.

\* Authorization.

\* Controllers.

\* Request/response handling.

\* Middleware.

\* Swagger/OpenAPI.

\* TUS upload integration.

\* Download handling.

\* API-level configuration.



\---



\# 7. Frontend Architecture



The actual frontend is:



```text

C:\\Akhil\\VASTRIX\\frontend

```



The current frontend is:



```text

React

\+

Vite

\+

JavaScript / JSX

\+

React Router

\+

Tailwind CSS

\+

Framer Motion

\+

Lucide React

```



It is \*\*not currently a Next.js application\*\*.



Any older documentation claiming that the VASTRIX frontend is Next.js should be treated as stale unless the source code is changed and verified.



\---



\# 8. Frontend Entry Point



The application starts from:



```text

src/main.jsx

```



The current routing includes:



```text

/

```



and:



```text

/t/:token

```



Current route architecture:



```text

Browser

&#x20;  │

&#x20;  ▼

BrowserRouter

&#x20;  │

&#x20;  ├── /

&#x20;  │    └── App.jsx

&#x20;  │

&#x20;  └── /t/:token

&#x20;       └── DownloadPage.jsx

```



\---



\# 9. Current Frontend Pages



\## 9.1 Landing / Transfer Page



Current route:



```text

/

```



Main component:



```text

App.jsx

```



Current functionality includes:



\* File selection.

\* Multiple-file queue.

\* Drag and drop.

\* File size validation.

\* Expiration selection.

\* Optional recipient verification checkbox.

\* Custom message.

\* Upload progress.

\* Transfer creation.

\* TUS upload.

\* Secure link generation.

\* Share functionality.



\---



\## 9.2 Download Page



Current route:



```text

/t/:token

```



Main component:



```text

DownloadPage.jsx

```



Current flow:



```text

Secure token

&#x20;   ↓

GET transfer details

&#x20;   ↓

Display transfer information

&#x20;   ↓

Display files

&#x20;   ↓

Download selected file

```



The current recipient page does not implement recipient OTP verification.



\---



\# 10. Frontend-to-Backend Communication



The frontend communicates with the backend through HTTP.



The API base is intended to be configurable using:



```text

VITE\_API\_BASE\_URL

```



The frontend also has a TUS endpoint derived from the API base.



However, the current source contains an important inconsistency:



```text

TUS endpoint:

API\_BASE + /api/upload-stream



Transfer creation:

hardcoded localhost:5000 endpoint

```



The backend currently runs on:



```text

http://localhost:8080

```



Therefore API base configuration must eventually be standardized.



\---



\# 11. Authentication Architecture



VASTRIX currently uses:



```text

JWT Bearer Authentication

```



High-level flow:



```text

User

&#x20;│

&#x20;│ username/email + password

&#x20;▼

POST /api/Auth/login

&#x20;│

&#x20;▼

Backend validates credentials

&#x20;│

&#x20;▼

JWT generated

&#x20;│

&#x20;▼

Client receives token

&#x20;│

&#x20;▼

Authorization: Bearer <JWT>

&#x20;│

&#x20;▼

Protected API

```



Password hashes use BCrypt.



\---



\# 12. JWT Claims



The current JWT contains identity information including:



```text

NameIdentifier

Name

Email

Role

```



The `NameIdentifier` contains the user's GUID.



The backend uses the authenticated identity to establish ownership.



The client should not be trusted to determine another user's identity.



\---



\# 13. Authentication Security Model



Current authentication protections include:



\* JWT issuer validation.

\* JWT audience validation.

\* Signature validation.

\* Expiration validation.

\* Zero clock skew.

\* BCrypt password hashing.

\* Disabled-user validation.

\* Authentication rate limiting.

\* Authorization policies.

\* CORS configuration.

\* HttpOnly Swagger authentication cookie.



Current authentication gaps include:



\* Refresh tokens.

\* Token revocation.

\* MFA.

\* Password reset.

\* Password change.

\* Advanced session/device management.

\* Account lockout.

\* Advanced abuse detection.



These are future security improvements unless separately implemented.



\---



\# 14. Transfer Architecture



A transfer represents a logical file-sharing operation.



Conceptually:



```text

User

&#x20;│

&#x20;├── creates

&#x20;│

&#x20;▼

Transfer

&#x20;│

&#x20;├── SecureToken

&#x20;├── CustomMessage

&#x20;├── Expiration

&#x20;├── RecipientVerification setting

&#x20;│

&#x20;└── TransferFiles

&#x20;        │

&#x20;        ├── File 1

&#x20;        ├── File 2

&#x20;        └── File N

```



\---



\# 15. Transfer Creation Flow



Current backend endpoint:



```text

POST /api/Transfer/create

```



Current high-level flow:



```text

Authenticated User

&#x20;      │

&#x20;      ▼

Create transfer request

&#x20;      │

&#x20;      ├── CustomMessage

&#x20;      ├── RequireRecipientVerification

&#x20;      └── ExpiryDays

&#x20;      │

&#x20;      ▼

Backend obtains owner from JWT

&#x20;      │

&#x20;      ▼

Create Transfer

&#x20;      │

&#x20;      ▼

Generate SecureToken

&#x20;      │

&#x20;      ▼

Return transfer information

```



The current backend defaults to 7 days if expiry is not positive and caps the requested value at 30 days.



The frontend currently exposes:



```text

1

3

7

14

30 days

```



The frontend and backend therefore have a contract mismatch because the backend accepts any positive value up to 30 rather than enforcing only those five options.



\---



\# 16. Current Transfer Lifecycle



The intended transfer lifecycle is:



```text

PENDING

&#x20;  ↓

UPLOADING

&#x20;  ↓

UPLOADED

&#x20;  ↓

SCANNING

&#x20;  ↓

AVAILABLE

&#x20;  ↓

QUARANTINED / FAILED

&#x20;  ↓

EXPIRED

&#x20;  ↓

DELETED

```



However, the current `Transfer` entity does not contain a `TransferStatus` property.



Therefore this lifecycle is currently an architectural target, not an implemented state machine.



\---



\# 17. Upload Architecture



Current upload technology:



```text

TUS

\+

tusdotnet

```



The browser performs resumable uploads through:



```text

/api/upload-stream

```



Current maximum individual file size:



```text

5 GiB

```



The frontend uses 4 MiB chunks in its custom TUS upload implementation.



\---



\# 18. Upload Security Flow



Current upload authorization conceptually works as:



```text

TUS request

&#x20;   │

&#x20;   ▼

JWT authentication

&#x20;   │

&#x20;   ▼

Authenticated user

&#x20;   │

&#x20;   ▼

Upload size validation

&#x20;   │

&#x20;   ▼

User quota validation

&#x20;   │

&#x20;   ▼

Transfer token validation

&#x20;   │

&#x20;   ▼

Transfer ownership validation

&#x20;   │

&#x20;   ▼

Transfer expiration validation

&#x20;   │

&#x20;   ▼

Upload permitted

```



Completion performs additional validation and database transaction processing.



\---



\# 19. File Storage Architecture



Current physical storage:



```text

C:\\VASTRIX\_FileStore

```



PostgreSQL stores file metadata.



High-level relationship:



```text

PostgreSQL

&#x20;  │

&#x20;  └── TransferFile

&#x20;         │

&#x20;         └── StorageObjectKey

&#x20;                   │

&#x20;                   ▼

&#x20;            Physical file

```



The detailed storage design is documented separately in:



```text

PROJECT\_MEMORY\\02\_ARCHITECTURE\\STORAGE\_ARCHITECTURE.md

```



\---



\# 20. Download Architecture



Current endpoint:



```text

GET /api/Transfer/download/{token}/{fileName}

```



Current download checks include:



\* Transfer exists.

\* Token matches.

\* Transfer is not deleted.

\* Transfer has not expired.

\* File belongs to transfer.

\* Physical file exists.

\* Resolved path remains inside storage root.



The backend then returns the physical file through a `FileStream`.



\---



\# 21. Recipient Architecture



The intended recipient flow is:



```text

Recipient receives secure link

&#x20;         │

&#x20;         ▼

Open /t/{token}

&#x20;         │

&#x20;         ▼

Transfer details

&#x20;         │

&#x20;         ├── Optional verification

&#x20;         │

&#x20;         ├── Optional password

&#x20;         │

&#x20;         └── File list

&#x20;         │

&#x20;         ▼

Authorized download

```



The current implementation is incomplete.



Currently:



```text

Secure token

&#x20;   ↓

Transfer details

&#x20;   ↓

File list

&#x20;   ↓

Direct download

```



Recipient OTP enforcement is not currently implemented.



\---



\# 22. Recipient Verification Architecture



The database contains:



```text

RecipientVerificationChallenge

```



It supports fields including:



\* TransferId.

\* RecipientEmail.

\* CodeHash.

\* CodeExpiresAt.

\* AttemptCount.

\* CreatedAt.

\* ConsumedAt.



This provides a database foundation for recipient verification.



However, the complete flow is not implemented.



Missing pieces include:



```text

OTP generation

OTP persistence workflow

OTP request endpoint

OTP verification endpoint

Attempt enforcement

Consumed-state enforcement

Recipient verification session

Download enforcement

Frontend OTP UI

```



Therefore:



> Recipient verification must currently be considered partially implemented, not complete.



\---



\# 23. Email Architecture



Current email infrastructure uses:



```text

MailKit

\+

SMTP

```



The application contains an email notification service.



Current notification concepts include:



\* Transfer sharing email.

\* Download notification.

\* Recipient verification email.



The recipient verification database foundation exists, but the complete verification workflow is not currently enforced.



The current SMTP configuration uses a non-secure socket option and requires future security review before production use.



Sensitive SMTP credentials must never be placed in PROJECT\_MEMORY.



\---



\# 24. Background Worker Architecture



VASTRIX currently registers:



```text

DiskSweeperWorker

```



It runs approximately hourly.



Its responsibilities include processing expired transfers and associated physical files.



Conceptually:



```text

Background Worker

&#x20;     │

&#x20;     ▼

Find expired transfers

&#x20;     │

&#x20;     ▼

Delete associated files

&#x20;     │

&#x20;     ▼

Mark transfers deleted

```



\---



\# 25. Cleanup Limitations



The current cleanup architecture is not a complete storage reconciliation system.



Potential problems include:



```text

Physical file exists

but database metadata does not



Database says deleted

but physical file remains



Physical deletion fails

while database state changes



Upload completes

but metadata creation fails

```



A future reconciliation process is required for a robust production storage architecture.



\---



\# 26. FileCleanupWorker



A `FileCleanupWorker` exists in the Infrastructure project.



It is currently not registered as an active hosted worker.



AI agents must not assume that the existence of the class means that it is running.



The currently registered cleanup worker is:



```text

DiskSweeperWorker

```



\---



\# 27. Database Architecture



Current database:



```text

PostgreSQL

```



Main entities:



```text

Users

Transfers

Files

RecipientVerificationChallenges

```



Conceptual relationship:



```text

User

&#x20;│

&#x20;└── 0..N Transfers

&#x20;         │

&#x20;         ├── 0..N Files

&#x20;         │

&#x20;         └── 0..N RecipientVerificationChallenges

```



The detailed schema is documented in:



```text

docs/database/DATABASE\_SCHEMA.md

```



\---



\# 28. Database Responsibilities



PostgreSQL is responsible for persistent application metadata.



It stores:



\* User accounts.

\* Password hashes.

\* Roles.

\* Disabled state.

\* Storage quotas.

\* Transfer metadata.

\* Secure tokens.

\* Transfer expiration.

\* File metadata.

\* Storage object keys.

\* Recipient verification challenge data.



It does not store large file contents.



\---



\# 29. Data Ownership Model



The ownership model is:



```text

Authenticated JWT

&#x20;      │

&#x20;      ▼

User ID

&#x20;      │

&#x20;      ▼

Transfer.UserId

&#x20;      │

&#x20;      ▼

TransferFile.TransferId

```



This creates the logical ownership chain:



```text

User

&#x20; ↓

Transfer

&#x20; ↓

File

```



Protected APIs must use authenticated identity rather than trusting client-supplied ownership identifiers.



\---



\# 30. User Isolation



The current transfer listing endpoint retrieves only the authenticated user's transfers.



Conceptually:



```text

Logged-in User A

&#x20;      │

&#x20;      ▼

Transfers where UserId = User A

```



A user should not be able to access another user's protected transfer records by manipulating IDs.



BOLA/IDOR protection is therefore an important architectural principle.



\---



\# 31. Admin Architecture



The backend currently contains an admin health endpoint:



```text

GET /api/admin/health

```



It requires administrative authorization.



There is also an anonymous system health endpoint:



```text

GET /api/v1/system/health

```



The frontend currently does not contain an admin dashboard.



\---



\# 32. Swagger Architecture



Swagger/OpenAPI is available for backend API exploration.



Swagger authentication is separately protected using a cookie-based mechanism.



Current concept:



```text

Swagger login

&#x20;     │

&#x20;     ▼

Swagger\_Auth\_Token

&#x20;     │

&#x20;     ▼

HttpOnly cookie

```



Swagger authentication should not be confused with the normal frontend JWT authentication model.



\---



\# 33. Security Boundaries



The major security boundaries are:



```text

&#x20;                   Internet

&#x20;                      │

&#x20;                      ▼

&#x20;                ┌───────────┐

&#x20;                │ Frontend  │

&#x20;                └─────┬─────┘

&#x20;                      │

&#x20;                      ▼

&#x20;                ┌───────────┐

&#x20;                │ API       │

&#x20;                │ Security  │

&#x20;                └─────┬─────┘

&#x20;                      │

&#x20;             ┌────────┴────────┐

&#x20;             ▼                 ▼

&#x20;       PostgreSQL          File Storage

```



The backend API is the main application security boundary.



The frontend is not trusted for authorization.



\---



\# 34. Current Security Principles



VASTRIX follows these architectural security principles:



1\. Authentication is enforced by the backend.

2\. Ownership is determined from authenticated identity.

3\. Large files are not stored in PostgreSQL.

4\. File storage is private.

5\. Transfer tokens act as access capabilities.

6\. Transfer expiration is enforced server-side.

7\. Filesystem paths must remain inside the configured storage root.

8\. Backend validation is authoritative.

9\. Frontend validation is primarily for user experience.

10\. Sensitive authentication information should not be logged.

11\. Protected APIs must enforce authorization.

12\. Security controls should not be weakened to solve frontend integration problems.



\---



\# 35. Current Frontend/Backend Integration Gaps



The current implementation has several known integration gaps.



\## 35.1 API Port Mismatch



Backend currently runs on:



```text

8080

```



The frontend's transfer creation code currently contains a hardcoded:



```text

5000

```



endpoint.



This must eventually be standardized.



\---



\## 35.2 Missing JWT in Upload Flow



The backend TUS endpoint requires authentication.



The current frontend upload implementation does not currently send the JWT Authorization header.



The current transfer creation request also does not currently include the expected Authorization header.



This is an integration issue.



The correct solution is to integrate authentication into the frontend rather than weakening backend authorization.



\---



\## 35.3 Recipient Verification Mismatch



Frontend:



```text

Recipient verification checkbox

```



Backend/database:



```text

RequireRecipientVerification

\+

RecipientVerificationChallenge foundation

```



But complete enforcement is missing.



\---



\## 35.4 Expiration Contract Mismatch



Frontend exposes:



```text

1 / 3 / 7 / 14 / 30

```



Backend accepts:



```text

positive value up to 30

```



The API contract should eventually be made explicit and consistent.



\---



\# 36. Current End-to-End User Flow



The current intended authenticated user flow is:



```text

User

&#x20;│

&#x20;▼

Frontend

&#x20;│

&#x20;├── select files

&#x20;├── configure transfer

&#x20;│

&#x20;▼

POST /api/Transfer/create

&#x20;│

&#x20;▼

Transfer created

&#x20;│

&#x20;▼

SecureToken

&#x20;│

&#x20;▼

TUS upload

&#x20;│

&#x20;▼

Physical storage

&#x20;│

&#x20;▼

TransferFile metadata

&#x20;│

&#x20;▼

Generate share link

&#x20;│

&#x20;▼

Share link with recipient

```



The current frontend authentication integration is incomplete, so the above represents the intended secured flow rather than a fully functioning end-to-end production flow.



\---



\# 37. Current End-to-End Recipient Flow



Current implementation:



```text

Recipient

&#x20;   │

&#x20;   ▼

/t/{token}

&#x20;   │

&#x20;   ▼

GET /api/Transfer/details/{token}

&#x20;   │

&#x20;   ▼

Transfer information

&#x20;   │

&#x20;   ▼

File list

&#x20;   │

&#x20;   ▼

Download

&#x20;   │

&#x20;   ▼

GET /api/Transfer/download/{token}/{fileName}

```



Current recipient verification is not enforced in this flow.



\---



\# 38. Intended Secure Recipient Flow



The target architecture is:



```text

Recipient

&#x20;   │

&#x20;   ▼

Secure transfer URL

&#x20;   │

&#x20;   ▼

Validate transfer

&#x20;   │

&#x20;   ├── expired? ──► deny

&#x20;   │

&#x20;   ├── password? ──► authenticate

&#x20;   │

&#x20;   ├── OTP required? ──► verify recipient

&#x20;   │

&#x20;   └── allowed

&#x20;            │

&#x20;            ▼

&#x20;       Download control

&#x20;            │

&#x20;            ▼

&#x20;   Short-lived signed URL

&#x20;            │

&#x20;            ▼

&#x20;      Private storage

```



This is the target design, not current implementation.



\---



\# 39. Future Production Architecture



The long-term architecture is expected to evolve toward:



```text

&#x20;                         ┌────────────────────┐

&#x20;                         │      Browser       │

&#x20;                         │                    │

&#x20;                         │ React / Vite       │

&#x20;                         └─────────┬──────────┘

&#x20;                                   │

&#x20;                                   ▼

&#x20;                        ┌──────────────────────┐

&#x20;                        │    VASTRIX API       │

&#x20;                        │                      │

&#x20;                        │ Authentication       │

&#x20;                        │ Authorization        │

&#x20;                        │ Transfer Management  │

&#x20;                        │ Storage Orchestration│

&#x20;                        │ Recipient Security   │

&#x20;                        └──────┬───────┬───────┘

&#x20;                               │       │

&#x20;                   metadata    │       │ signed access

&#x20;                               │       │

&#x20;                               ▼       ▼

&#x20;                       ┌────────────┐ ┌──────────────────┐

&#x20;                       │ PostgreSQL │ │ Private Object   │

&#x20;                       │            │ │ Storage          │

&#x20;                       └────────────┘ └──────────────────┘

&#x20;                               ▲

&#x20;                               │

&#x20;                        ┌──────┴───────┐

&#x20;                        │ Background   │

&#x20;                        │ Processing   │

&#x20;                        └──────┬───────┘

&#x20;                               │

&#x20;                   ┌───────────┼────────────┐

&#x20;                   ▼           ▼            ▼

&#x20;                Cleanup     Scanning     Notifications

```



Additional infrastructure may later include:



```text

Redis

Queue

Monitoring

Audit Logging

```



\---



\# 40. Future Upload Architecture



Target:



```text

Browser

&#x20;  │

&#x20;  │ request authorized upload

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  │ generate authorization

&#x20;  ▼

Private Object Storage

&#x20;  ▲

&#x20;  │

&#x20;  │ direct resumable/multipart upload

&#x20;  │

Browser

```



The API should orchestrate authorization and metadata rather than unnecessarily proxying large files.



\---



\# 41. Future Download Architecture



Target:



```text

Recipient

&#x20;  │

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  ├── transfer validation

&#x20;  ├── expiration validation

&#x20;  ├── recipient verification

&#x20;  ├── password validation

&#x20;  └── download limit validation

&#x20;  │

&#x20;  ▼

Short-lived signed URL

&#x20;  │

&#x20;  ▼

Private Object Storage

&#x20;  │

&#x20;  ▼

Recipient

```



\---



\# 42. Future Asynchronous Processing



Large-scale production processing may evolve toward:



```text

Upload complete

&#x20;     │

&#x20;     ▼

Queue

&#x20;     │

&#x20;     ├── Malware scan

&#x20;     ├── Metadata processing

&#x20;     ├── Notification

&#x20;     ├── Audit event

&#x20;     └── Cleanup

```



This is planned architecture.



It is not currently a complete queue-based implementation.



\---



\# 43. Architecture Evolution Strategy



VASTRIX should evolve incrementally.



Recommended architectural progression:



```text

Phase 1

Current local filesystem

&#x20;       │

&#x20;       ▼

Complete core application behavior

&#x20;       │

&#x20;       ▼

Phase 2

Harden authentication and recipient security

&#x20;       │

&#x20;       ▼

Phase 3

Formalize storage abstraction

&#x20;       │

&#x20;       ▼

Phase 4

Private object storage

&#x20;       │

&#x20;       ▼

Phase 5

Signed upload/download

&#x20;       │

&#x20;       ▼

Phase 6

Async processing / scanning / reconciliation

&#x20;       │

&#x20;       ▼

Phase 7

Scaling infrastructure

```



The exact implementation sequence may change as development progresses.



\---



\# 44. What Is Implemented Today



\### Implemented



\* React/Vite frontend.

\* Basic landing/transfer UI.

\* File queue.

\* Drag/drop.

\* Client-side file size validation.

\* Transfer creation API.

\* Secure transfer token.

\* TUS upload infrastructure.

\* Local file storage.

\* PostgreSQL metadata.

\* User authentication.

\* JWT authorization infrastructure.

\* User ownership relationship.

\* Transfer expiration.

\* Anonymous transfer details.

\* Anonymous file download.

\* Filesystem path containment.

\* Storage quota model.

\* Download notifications.

\* Background cleanup worker.

\* Admin health endpoint.

\* System health endpoint.



\---



\# 45. What Is Partially Implemented



\* Recipient verification.

\* Email notification system.

\* Storage quota consistency.

\* Frontend/backend API integration.

\* TUS frontend authentication integration.

\* Transfer expiration contract.

\* Cleanup/reconciliation.

\* Storage abstraction.



\---



\# 46. What Is Not Yet Implemented



\* Complete recipient OTP workflow.

\* Recipient verification enforcement.

\* Transfer password protection.

\* Maximum download count.

\* Download counter.

\* Transfer status state machine.

\* Malware scanning.

\* Private object storage.

\* Signed upload URLs.

\* Signed download URLs.

\* Orphan-object reconciliation.

\* Full storage-provider abstraction.

\* Redis-based infrastructure.

\* Queue-based processing.

\* Production monitoring.

\* Full audit architecture.

\* User dashboard.

\* User registration/login UI.

\* Admin UI.



\---



\# 47. AI Guidance



When an AI assistant works on VASTRIX, it should first determine:



```text

Is this feature:



Implemented?

Partially implemented?

Missing?

Planned?

Blocked?

Deprecated?

```



It should not assume that a database field, UI control, service class, or architectural document means the complete feature is implemented.



For example:



```text

RequireRecipientVerification = true

```



does not mean:



```text

Recipient verification is fully implemented.

```



The actual request/verification/enforcement flow must exist.



\---



\# 48. AI Source Verification Rule



Before changing architecture-sensitive code, AI should verify the actual source.



Priority:



```text

Current source code

&#x20;      >

Current database model/migrations

&#x20;      >

Current configuration

&#x20;      >

Technical docs

&#x20;      >

PROJECT\_MEMORY

&#x20;      >

Older documentation

```



PROJECT\_MEMORY describes the known state but must not override verified source code.



\---



\# 49. AI Change Discipline



AI assistants should avoid making unrelated architectural changes.



If the requested task is:



```text

Fix TUS authentication

```



do not automatically:



\* Migrate to S3.

\* Introduce Redis.

\* Rewrite the frontend.

\* Add Kubernetes.

\* Replace TUS.

\* Change the database architecture.



unless those changes are explicitly requested and justified.



\---



\# 50. Architectural Principle



The central VASTRIX architecture can be summarized as:



```text

Frontend

&#x20;   ↓

API

&#x20;   ↓

Business/security decisions

&#x20;   ↓

PostgreSQL metadata

&#x20;   +

Private file storage

```



The API controls:



```text

Who

What

When

How

```



The database records:



```text

Who owns what

What transfer exists

What files belong to it

Where the stored object is referenced

```



Storage holds:



```text

Actual file contents

```



Future object storage and signed URLs should preserve this separation.



\---



\# 51. Final Architectural Rule



> \*\*VASTRIX is a secure transfer system, not a cloud drive.\*\*



The architecture should remain focused on:



```text

Upload

&#x20;  ↓

Secure transfer

&#x20;  ↓

Controlled sharing

&#x20;  ↓

Controlled download

&#x20;  ↓

Expiration

&#x20;  ↓

Cleanup

```



Features that do not directly support this core transfer lifecycle should be evaluated carefully before being introduced.



The architecture should prioritize:



```text

Security

Correctness

Reliability

Maintainability

Scalability

Operational simplicity

```



while avoiding unnecessary infrastructure until it is justified by actual product requirements.



