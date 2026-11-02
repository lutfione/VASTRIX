\# VASTRIX — Backend Architecture



\*\*Document Type:\*\* Project Memory

\*\*Product:\*\* VASTRIX

\*\*Backend:\*\* ASP.NET Core / .NET 9

\*\*Architecture Style:\*\* Layered architecture with Core, Infrastructure, and API projects

\*\*Status:\*\* Source-derived baseline

\*\*Last Updated:\*\* 2026-09-16



\---



\# 1. Purpose



This document records the actual backend architecture of VASTRIX.



It is intended to help:



\* Human developers understand the backend.

\* AI coding assistants understand the project structure.

\* Future troubleshooting and maintenance.

\* Prevent architectural assumptions that do not match the source code.

\* Track the boundary between implemented architecture and planned architecture.



Detailed technical reference material is maintained separately under:



```text

docs\\backend\\

docs\\api\\

docs\\database\\

```



This document is the higher-level project-memory view.



\---



\# 2. Backend Solution



The backend is located at:



```text

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\

```



The solution file is:



```text

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.slnx

```



The solution currently contains three primary projects:



```text

VASTRIX

├── VASTRIX.Core

├── VASTRIX.Infrastructure

└── VASTRIX.API

```



\---



\# 3. Layered Architecture



The current dependency direction is:



```text

VASTRIX.API

&#x20;    │

&#x20;    ├──────────────► VASTRIX.Infrastructure

&#x20;    │

&#x20;    └──────────────► VASTRIX.Core

&#x20;                         ▲

&#x20;                         │

VASTRIX.Infrastructure ──┘

```



Dependency rules currently observed:



```text

API → Infrastructure

API → Core

Infrastructure → Core

Core → Nothing

```



The Core project is therefore the least dependent layer.



\---



\# 4. Architectural Responsibilities



\## 4.1 VASTRIX.Core



The Core project contains the application's fundamental domain/application model.



It is intended to remain independent of:



\* ASP.NET Core infrastructure.

\* Database provider details.

\* HTTP transport concerns.

\* Frontend concerns.

\* Hosting infrastructure.



Examples of core concepts include:



\* User

\* UserRole

\* Transfer

\* TransferFile

\* RecipientVerificationChallenge



The Core project has no dependency on the other VASTRIX projects.



\---



\# 5. VASTRIX.Infrastructure



The Infrastructure project contains implementation details required by the application.



Current responsibilities include:



\* Entity Framework Core.

\* PostgreSQL integration.

\* `ApplicationDbContext`.

\* Database mappings.

\* Database relationships.

\* Database migrations.

\* Email notification implementation.

\* Background workers.

\* File cleanup functionality.



Current important technologies include:



```text

Entity Framework Core 9

Npgsql

PostgreSQL

MailKit

Microsoft.Extensions.Hosting

```



Infrastructure depends on Core.



\---



\# 6. VASTRIX.API



The API project is the application entry point and HTTP layer.



Target framework:



```text

net9.0

```



Current responsibilities include:



\* Application startup.

\* HTTP endpoints.

\* Authentication.

\* Authorization.

\* JWT processing.

\* Middleware.

\* CORS.

\* Rate limiting.

\* Swagger/OpenAPI.

\* Transfer operations.

\* Upload handling.

\* Exception handling.

\* Request identification.

\* Administrative endpoints.

\* System health endpoints.



The API project references both Core and Infrastructure.



\---



\# 7. Application Startup



The main API startup is handled by:



```text

VASTRIX.API\\Program.cs

```



The application configures and registers the major runtime components before building the ASP.NET Core application host.



The startup process includes registration of:



\* Database services.

\* Authentication.

\* Authorization.

\* CORS.

\* Rate limiting.

\* Swagger/OpenAPI.

\* Controllers.

\* Application services.

\* Background workers.

\* Middleware.



\---



\# 8. HTTP Request Flow



The conceptual request flow is:



```text

Browser / Client

&#x20;      │

&#x20;      ▼

ASP.NET Core

&#x20;      │

&#x20;      ▼

Middleware

&#x20;      │

&#x20;      ├── Request ID

&#x20;      ├── Exception handling

&#x20;      ├── Authentication

&#x20;      ├── Authorization

&#x20;      ├── CORS / rate limiting

&#x20;      │

&#x20;      ▼

Controller / TUS handling

&#x20;      │

&#x20;      ▼

Application logic

&#x20;      │

&#x20;      ├── Core entities

&#x20;      ├── Infrastructure services

&#x20;      └── Database / filesystem / email

&#x20;      │

&#x20;      ▼

HTTP response

```



The exact middleware ordering is controlled by the current `Program.cs` implementation and must be treated as authoritative over this conceptual diagram.



\---



\# 9. Authentication Architecture



VASTRIX currently uses:



```text

ASP.NET Core JWT Bearer Authentication

```



The authentication flow is approximately:



```text

Client

&#x20; │

&#x20; │ username/email + password

&#x20; ▼

POST /api/Auth/login

&#x20; │

&#x20; ▼

Credential validation

&#x20; │

&#x20; ▼

BCrypt password verification

&#x20; │

&#x20; ▼

JWT generation

&#x20; │

&#x20; ▼

Client

```



Authenticated requests then provide the JWT through the Authorization header.



The backend extracts identity and role information from the authenticated principal.



\---



\# 10. JWT Identity



The current JWT contains identity information including:



\* NameIdentifier

\* Name

\* Email

\* Role



The NameIdentifier represents the user's database GUID.



Protected operations use the authenticated identity rather than trusting an arbitrary user ID supplied by the client.



The backend also performs a database user lookup and checks the user's disabled state.



\---



\# 11. Authorization



VASTRIX currently has two implemented backend roles:



```text

User

Admin

```



The corresponding enum is:



```csharp

public enum UserRole

{

&#x20;   User = 0,

&#x20;   Admin = 1

}

```



Authorization is performed server-side.



The frontend must never be treated as the authoritative authorization mechanism.



\---



\# 12. Password Security



User passwords are not stored as plaintext.



The backend uses:



```text

BCrypt.Net-Next

```



Passwords are hashed before storage and verified during authentication.



Password hashes must not be returned through API responses or logged.



\---



\# 13. Database Architecture



VASTRIX uses:



```text

PostgreSQL

&#x20;   │

&#x20;   ▼

Entity Framework Core

&#x20;   │

&#x20;   ▼

ApplicationDbContext

```



The database stores application metadata.



Large uploaded file contents are \*\*not intended to be stored in PostgreSQL\*\*.



The current local storage architecture uses the filesystem for uploaded files.



\---



\# 14. ApplicationDbContext



The database context is responsible for the application's persistence model.



Current DbSets include:



```text

Users

Transfers

Files

RecipientVerificationChallenges

```



The main entities are:



```text

User

UserRole

Transfer

TransferFile

RecipientVerificationChallenge

```



Relationships and database mappings are defined through Entity Framework Core.



\---



\# 15. Transfer Ownership



Transfers are associated with users through:



```text

Transfer.UserId

```



The current security model is:



```text

JWT

&#x20;↓

Authenticated User ID

&#x20;↓

Transfer.UserId

&#x20;↓

Authorized transfer operation

```



The client should not be allowed to choose the owner of a transfer.



This is an important protection against broken object-level authorization / IDOR-style vulnerabilities.



\---



\# 16. Transfer API



The current transfer controller provides functionality including:



```text

GET  /api/Transfer

POST /api/Transfer/create

GET  /api/Transfer/details/{token}

GET  /api/Transfer/download/{token}/{fileName}

```



The transfer API currently supports:



\* Authenticated transfer creation.

\* Authenticated user's transfer listing.

\* Secure transfer token lookup.

\* Transfer expiration checks.

\* Anonymous transfer detail access.

\* Anonymous download access.

\* Soft deletion checks.



Several planned transfer security features remain incomplete.



\---



\# 17. TUS Upload Architecture



Large file uploads use the TUS resumable upload protocol through:



```text

/api/upload-stream

```



The backend integrates TUS handling with authenticated user and transfer validation.



The current maximum file size is:



```text

5 GiB

```



The local storage root is:



```text

C:\\VASTRIX\_FileStore

```



\---



\# 18. TUS Authorization Flow



Before accepting an upload, the current implementation validates several conditions.



Conceptually:



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

Upload-Length validation

&#x20;   │

&#x20;   ├── non-negative

&#x20;   └── ≤ 5 GiB

&#x20;   │

&#x20;   ▼

User storage quota

&#x20;   │

&#x20;   ▼

Upload metadata

&#x20;   │

&#x20;   ▼

Transfer token

&#x20;   │

&#x20;   ▼

Transfer ownership

&#x20;   │

&#x20;   ▼

Transfer expiration

&#x20;   │

&#x20;   ▼

Allow upload

```



For non-POST TUS operations, the implementation uses the upload's file identifier to resolve the associated transfer metadata.



\---



\# 19. TUS File Completion



When a TUS upload completes, the backend performs additional validation before creating the database file record.



The process includes:



```text

Upload complete

&#x20;     │

&#x20;     ▼

Authenticated user

&#x20;     │

&#x20;     ▼

Transfer metadata

&#x20;     │

&#x20;     ▼

Transfer ownership / expiration validation

&#x20;     │

&#x20;     ▼

Physical file existence

&#x20;     │

&#x20;     ▼

Actual file size validation

&#x20;     │

&#x20;     ▼

Database transaction

&#x20;     │

&#x20;     ▼

User row locking

&#x20;     │

&#x20;     ▼

Recalculate storage usage

&#x20;     │

&#x20;     ▼

Final quota validation

&#x20;     │

&#x20;     ▼

Create TransferFile

&#x20;     │

&#x20;     ▼

Commit

```



This is intended to prevent quota bypass through concurrent uploads.



\---



\# 20. Local File Storage



The current backend uses local Windows filesystem storage:



```text

C:\\VASTRIX\_FileStore

```



Large files are therefore stored outside PostgreSQL.



The database stores metadata such as:



\* Original filename.

\* File size.

\* Content type.

\* Transfer ID.

\* Storage object key.

\* Upload timestamp.



The current `StorageObjectKey` is associated with the TUS file identifier.



\---



\# 21. Download Architecture



The current download endpoint is:



```text

GET /api/Transfer/download/{token}/{fileName}

```



The backend:



1\. Resolves the transfer using the secure token.

2\. Confirms the transfer exists.

3\. Confirms it has not been deleted.

4\. Confirms it has not expired.

5\. Decodes the requested filename.

6\. Finds the corresponding transfer file.

7\. Builds the physical path.

8\. Performs a path-containment check.

9\. Confirms the file exists.

10\. Streams the file to the client.



The current implementation does not yet use object-storage signed download URLs.



\---



\# 22. File Path Security



The download implementation performs containment validation so that a requested file path cannot escape the configured storage root.



This is an important security boundary.



The intended rule is:



```text

Requested path

&#x20;     │

&#x20;     ▼

Resolved physical path

&#x20;     │

&#x20;     ▼

Must remain inside storage root

&#x20;     │

&#x20;     ├── YES → continue

&#x20;     └── NO  → reject

```



The filesystem path must never be exposed to the client.



\---



\# 23. Background Workers



The backend currently contains background cleanup functionality.



`DiskSweeperWorker` is registered and runs periodically.



Its current responsibilities include processing expired transfers and attempting to delete associated files.



There is also a `FileCleanupWorker` implementation in Infrastructure, but it is not currently registered based on the verified implementation.



Therefore:



```text

DiskSweeperWorker → Registered

FileCleanupWorker → Exists but not registered

```



Future changes must not assume that every worker class present in the repository is actively running.



\---



\# 24. Cleanup Architecture



The current cleanup model is based on local filesystem storage.



Conceptually:



```text

Expired transfer

&#x20;     │

&#x20;     ▼

Identify files

&#x20;     │

&#x20;     ▼

Attempt physical deletion

&#x20;     │

&#x20;     ▼

Mark transfer deleted

```



Current limitations include:



\* No atomic transaction between filesystem deletion and database state.

\* Locked-file deletion can fail.

\* Orphaned physical files can remain.

\* Metadata is retained after soft deletion.

\* No verified orphan-file scanning mechanism is currently active.



These are known architectural limitations rather than reasons to weaken security controls.



\---



\# 25. Email Architecture



Email functionality is provided through:



```text

IEmailNotificationService

EmailNotificationService

```



The implementation uses:



```text

MailKit

SMTP

```



Current notification purposes include:



\* Transfer link sharing.

\* Download notifications.

\* Recipient verification email.



Recipient verification has a database foundation, but the complete verification workflow is not yet implemented.



\---



\# 26. API Security Components



The current API includes security-related infrastructure for:



\* JWT authentication.

\* Role authorization.

\* BCrypt password hashing.

\* CORS.

\* Authentication rate limiting.

\* Request ID middleware.

\* Global exception handling.

\* Secure Swagger authentication.

\* Ownership checks.

\* Transfer token checks.

\* Expiration checks.

\* File path containment.



Security testing is treated as a separate project concern and is documented under the security documentation.



\---



\# 27. Swagger Security



Swagger/OpenAPI is protected by a separate authentication mechanism.



The backend provides:



```text

POST /api/Auth/swagger-login

POST /api/Auth/swagger-logout

```



Swagger authentication uses an HttpOnly cookie named:



```text

Swagger\_Auth\_Token

```



The cookie is configured for secure browser behavior.



This Swagger mechanism must not be confused with the normal frontend JWT authentication flow.



\---



\# 28. Health Endpoints



The backend currently exposes:



```text

GET /api/v1/system/health

```



for system-level health information.



An administrative health endpoint also exists:



```text

GET /api/admin/health

```



The administrative endpoint requires Admin authorization.



\---



\# 29. Exception Handling



The backend includes centralized exception handling through a global exception-handling mechanism.



The objective is to prevent uncontrolled exceptions from becoming raw server responses and to provide consistent API error behavior.



Sensitive internal information should not be exposed through production error responses.



\---



\# 30. Request Identification



The backend includes request ID middleware.



Request IDs are intended to help correlate:



```text

Client request

&#x20;     ↓

Middleware

&#x20;     ↓

Controller/service operations

&#x20;     ↓

Logs / diagnostics

```



Request identifiers should be useful for troubleshooting without exposing sensitive authentication or file information.



\---



\# 31. CORS



The API uses a configured CORS policy.



Allowed origins are controlled through application configuration rather than being inherently open to every frontend origin.



The current frontend/backend integration must keep the configured origin aligned with the actual development and deployment environment.



\---



\# 32. Rate Limiting



Authentication endpoints have rate-limiting protection.



The currently documented authentication limit is:



```text

10 requests per minute per IP

```



with no queue allowance.



This protects authentication endpoints from basic high-volume abuse.



Additional abuse protection remains a future improvement.



\---



\# 33. Configuration



The backend uses application configuration for items such as:



\* Database connection.

\* JWT settings.

\* CORS allowed origins.

\* SMTP configuration.

\* Storage configuration where applicable.

\* Swagger authentication configuration.



Secrets and credentials must not be committed to source control.



Configuration values containing credentials, tokens, passwords, or private connection information must be treated as sensitive.



\---



\# 34. Current Architecture — Implemented



The following architectural components are currently implemented:



```text

ASP.NET Core API                  ✅

.NET 9                            ✅

Core project                      ✅

Infrastructure project           ✅

PostgreSQL                        ✅

Entity Framework Core             ✅

JWT authentication                ✅

BCrypt password hashing           ✅

Role authorization                ✅

Transfer ownership checks         ✅

TUS resumable upload              ✅

Local filesystem storage          ✅

Transfer expiration               ✅

Anonymous transfer access         ✅

Anonymous download                ✅

Email service foundation          ✅

Background cleanup worker         ✅

Global exception handling         ✅

Request ID middleware             ✅

CORS                              ✅

Authentication rate limiting      ✅

Swagger authentication            ✅

Health endpoints                  ✅

```



\---



\# 35. Current Architecture — Partial



The following areas exist but are incomplete or have limitations:



```text

Recipient verification             ⚠️

Storage quota consistency          ⚠️

File cleanup/orphan handling       ⚠️

Download notifications              ⚠️

Transfer security controls         ⚠️

Administrative functionality       ⚠️

Frontend/API authentication        ⚠️

```



\---



\# 36. Current Architecture — Not Yet Implemented



The following are planned architectural capabilities rather than current functionality:



```text

Private object storage

Signed upload URLs

Signed download URLs

Redis

Distributed queue

Malware scanning

Dedicated scanning pipeline

Advanced audit logging

Advanced monitoring

Production-grade distributed workers

Docker deployment

Kubernetes deployment

CI/CD pipeline

```



These should not be introduced prematurely while the MVP architecture is still being stabilized.



\---



\# 37. Planned Storage Evolution



Current:



```text

Browser

&#x20;  │

&#x20;  ▼

VASTRIX API / TUS

&#x20;  │

&#x20;  ▼

Windows filesystem

C:\\VASTRIX\_FileStore

```



Future target:



```text

Browser

&#x20;  │

&#x20;  ▼

Signed resumable upload

&#x20;  │

&#x20;  ▼

Private object storage

&#x20;  │

&#x20;  ├── File contents

&#x20;  │

&#x20;  └── PostgreSQL

&#x20;        │

&#x20;        └── Metadata

```



Future download architecture is expected to use short-lived signed URLs rather than permanent public URLs.



\---



\# 38. Planned Processing Pipeline



The intended long-term file lifecycle is:



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



The corresponding `TransferStatus` model is not currently implemented.



The lifecycle is therefore a product/architecture target rather than current backend behavior.



\---



\# 39. Important Architectural Gaps



The current backend does not yet fully implement several requirements from the intended product design.



Important gaps include:



1\. Recipient verification is not enforced end-to-end.

2\. Transfer password protection is not implemented.

3\. Maximum download limits are not implemented.

4\. Download counters are not implemented.

5\. Transfer status is not implemented.

6\. Recipient email is not stored directly on `Transfer`.

7\. Object storage is not implemented.

8\. Signed upload URLs are not implemented.

9\. Signed download URLs are not implemented.

10\. Refresh tokens are not implemented.

11\. MFA is not implemented.

12\. Password reset/change functionality is not implemented.

13\. Account lockout/session/device management is not implemented.

14\. Advanced abuse detection is not implemented.

15\. Malware scanning is not implemented.

16\. Full audit logging is not implemented.

17\. Production distributed processing infrastructure is not implemented.



\---



\# 40. Architecture Principles



Future backend development should preserve these principles:



\### Principle 1 — Server Is Authoritative



Security decisions must be made by the backend.



\### Principle 2 — Ownership Comes From Authentication



Do not trust client-provided ownership identifiers.



\### Principle 3 — Database Stores Metadata



Large file contents should not be stored in PostgreSQL.



\### Principle 4 — Files Remain Private



Files must not become permanently public.



\### Principle 5 — Tokens Are Sensitive



Transfer tokens should be treated as secret access capabilities.



\### Principle 6 — Defense in Depth



Validation should exist at appropriate layers rather than relying on a single frontend check.



\### Principle 7 — Least Privilege



Users, recipients, administrators, services, and workers should receive only the access they require.



\### Principle 8 — Do Not Weaken Security to Fix Integration Problems



Frontend integration problems must be solved through correct authentication and API integration rather than disabling backend authorization.



\### Principle 9 — Actual Source Overrides Memory



This document describes the verified architecture at the time of writing.



If source code changes, the source code becomes authoritative and this memory must be updated accordingly.



\---



\# 41. AI Development Guidance



When an AI assistant works on the VASTRIX backend, it should first understand:



```text

Core

&#x20;↓

Infrastructure

&#x20;↓

API

```



and the dependency direction:



```text

API → Infrastructure → Core

API → Core

```



The assistant should not:



\* Move database entities into the API without architectural reason.

\* Put large file contents into PostgreSQL.

\* Trust client ownership IDs.

\* Disable authentication to make an endpoint work.

\* Make transfer files permanently public.

\* Assume planned features already exist.

\* Assume every worker class is registered.

\* Assume the frontend is correctly integrated merely because the API works independently.



Before making significant architectural changes, inspect the current source and update the relevant project memory.



\---



\# 42. Source of Truth



For implementation details, the authoritative sources are the actual backend code.



Primary locations:



```text

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.Core

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.Infrastructure

C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.API

```



Supporting technical documentation:



```text

C:\\Akhil\\VASTRIX\\docs\\

```



Project memory:



```text

C:\\Akhil\\VASTRIX\\PROJECT\_MEMORY\\

```



The hierarchy is:



```text

Actual source code

&#x20;      ↓

Technical documentation

&#x20;      ↓

Project memory

&#x20;      ↓

AI / human understanding

```



Project memory must describe the source accurately and must not become an independent competing implementation specification.



