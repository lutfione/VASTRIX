\# VASTRIX — Storage Architecture



\## 1. Purpose



This document describes how VASTRIX stores, references, uploads, downloads, and cleans up files.



It is intended to help:



\* Human developers understand the storage design.

\* AI coding assistants understand the current implementation.

\* Future development preserve the intended security boundaries.

\* Troubleshooting distinguish database metadata problems from physical file-storage problems.

\* Future migration from local filesystem storage to private object storage.



This document describes the \*\*storage architecture and decisions\*\*, not every implementation detail of the backend code.



For detailed source-level implementation, refer to:



```text

docs/backend/BACKEND\_ARCHITECTURE.md

docs/backend/BACKEND\_SERVICES.md

docs/database/DATABASE\_SCHEMA.md

docs/database/DATABASE\_RULES.md

docs/api/API\_ENDPOINTS.md

docs/api/UPLOAD\_DOWNLOAD.md

```



\---



\# 2. Source of Truth



The actual application source code is authoritative.



The documentation and PROJECT\_MEMORY files describe the source code and intended architecture but must not be treated as more authoritative than the implementation.



When a discrepancy is discovered:



```text

Actual source code

&#x20;     ↓

Verify behavior

&#x20;     ↓

Update documentation

&#x20;     ↓

Update PROJECT\_MEMORY

```



Do not modify application behavior merely to make documentation appear correct.



\---



\# 3. Current Storage Architecture



\## 3.1 Current Model



VASTRIX currently uses the local Windows filesystem for physical file storage.



```text

Browser

&#x20;  │

&#x20;  │ TUS upload

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  │ tusdotnet

&#x20;  ▼

Local Windows Filesystem

C:\\VASTRIX\_FileStore

&#x20;  │

&#x20;  │

&#x20;  └── Physical file data



PostgreSQL

&#x20;  │

&#x20;  └── File metadata

```



The large file itself is \*\*not stored in PostgreSQL\*\*.



PostgreSQL stores metadata describing the file and its relationship to a transfer.



\---



\# 4. Current Physical Storage Location



The currently configured storage root is:



```text

C:\\VASTRIX\_FileStore

```



The application currently uses this local filesystem location for uploaded files.



The storage root is currently hardcoded in relevant backend upload/download/cleanup code.



This should be considered a current implementation detail rather than the final production architecture.



\---



\# 5. Database Storage Model



PostgreSQL stores file metadata through the `TransferFile` entity.



Current important fields are:



```text

Id

TransferId

OriginalFileName

FileSizeBytes

ContentType

StorageObjectKey

UploadedAt

```



Conceptually:



```text

Transfer

&#x20;  │

&#x20;  └── TransferFile

&#x20;         ├── Id

&#x20;         ├── OriginalFileName

&#x20;         ├── FileSizeBytes

&#x20;         ├── ContentType

&#x20;         ├── StorageObjectKey

&#x20;         └── UploadedAt

```



The database therefore answers:



> What file belongs to which transfer, what was its original name, how large was it, what type was it, and where is its stored object identified?



The database does \*\*not\*\* contain the actual large-file contents.



\---



\# 6. StorageObjectKey



`TransferFile.StorageObjectKey` is the database reference to the stored physical object.



In the current TUS completion flow, the value is populated using the TUS `FileId`.



Conceptually:



```text

TUS FileId

&#x20;    │

&#x20;    ▼

StorageObjectKey

&#x20;    │

&#x20;    ▼

Physical stored file

```



This gives VASTRIX a storage abstraction that can eventually be mapped to object-storage keys.



The current implementation should not be assumed to provide a fully abstract storage-provider interface.



\---



\# 7. Upload Architecture



\## 7.1 Current Upload Flow



The current frontend performs a TUS-based upload.



High-level flow:



```text

User selects file

&#x20;      │

&#x20;      ▼

Frontend validates file size

&#x20;      │

&#x20;      ▼

Frontend creates Transfer

&#x20;      │

&#x20;      ▼

Backend returns SecureToken

&#x20;      │

&#x20;      ▼

Frontend starts TUS upload

&#x20;      │

&#x20;      ▼

POST /api/upload-stream

&#x20;      │

&#x20;      ▼

TUS upload session

&#x20;      │

&#x20;      ▼

File chunks uploaded

&#x20;      │

&#x20;      ▼

Local filesystem

C:\\VASTRIX\_FileStore

&#x20;      │

&#x20;      ▼

Upload completion

&#x20;      │

&#x20;      ▼

Backend validates final file

&#x20;      │

&#x20;      ▼

TransferFile metadata created

&#x20;      │

&#x20;      ▼

PostgreSQL

```



\---



\# 8. TUS Upload



VASTRIX currently uses `tusdotnet` for resumable upload handling.



The current maximum individual upload size is:



```text

5 GiB

```



Equivalent byte limit:



```text

5,368,709,120 bytes

```



The backend also performs server-side validation.



The frontend's 5 GiB limit is therefore only a client-side usability restriction. The backend remains the security authority.



\---



\# 9. Upload Authorization and Storage Validation



The current TUS authorization process performs multiple checks.



Conceptually:



```text

Request

&#x20; │

&#x20; ├── Is JWT authentication valid?

&#x20; │

&#x20; ├── Is the user authenticated?

&#x20; │

&#x20; ├── Is the user identity available?

&#x20; │

&#x20; ├── Is Upload-Length present?

&#x20; │

&#x20; ├── Is Upload-Length valid?

&#x20; │

&#x20; ├── Is Upload-Length <= 5 GiB?

&#x20; │

&#x20; ├── Is the user's quota sufficient?

&#x20; │

&#x20; ├── Is upload metadata present?

&#x20; │

&#x20; ├── Is transferToken present?

&#x20; │

&#x20; ├── Does transfer exist?

&#x20; │

&#x20; ├── Does transfer belong to authenticated user?

&#x20; │

&#x20; ├── Is transfer deleted?

&#x20; │

&#x20; └── Is transfer expired?

```



For non-POST TUS operations, the implementation retrieves the associated transfer token using the upload/file identifier and then performs ownership and transfer-state validation.



\---



\# 10. Upload Completion



When a TUS upload completes, the backend performs additional validation.



Conceptually:



```text

TUS completion

&#x20;     │

&#x20;     ▼

Authenticate user

&#x20;     │

&#x20;     ▼

Read upload metadata

&#x20;     │

&#x20;     ▼

Validate transfer ownership

&#x20;     │

&#x20;     ▼

Validate transfer expiration

&#x20;     │

&#x20;     ▼

Determine physical file

&#x20;     │

&#x20;     ▼

Check file exists

&#x20;     │

&#x20;     ▼

Check actual file size

&#x20;     │

&#x20;     ▼

Begin database transaction

&#x20;     │

&#x20;     ▼

Lock user row

&#x20;     │

&#x20;     ▼

Recalculate storage usage

&#x20;     │

&#x20;     ▼

Check quota again

&#x20;     │

&#x20;     ▼

Create TransferFile metadata

&#x20;     │

&#x20;     ▼

Commit transaction

```



The second quota check is important because the initial authorization check and final upload completion can occur at different times.



\---



\# 11. User Storage Quota



The `User` entity currently contains:



```text

StorageQuotaBytes

```



The current default quota in the entity is:



```text

5 GiB

=

5,368,709,120 bytes

```



The upload system checks the user's current stored transfer-file usage before allowing uploads.



The completion process also recalculates usage before committing the final metadata.



This helps protect against concurrent uploads exceeding the available quota.



\---



\# 12. Current Storage Usage Calculation



Storage usage is based on stored `TransferFile` metadata.



Conceptually:



```text

User

&#x20;│

&#x20;└── Transfers

&#x20;      │

&#x20;      └── TransferFiles

&#x20;             │

&#x20;             └── FileSizeBytes

```



The application sums relevant file sizes when evaluating quota.



The physical filesystem is therefore not the source used for normal user quota calculations.



This creates an important distinction:



```text

Database metadata

&#x20;      ≠

Physical filesystem state

```



They are expected to correspond, but failures can temporarily make them inconsistent.



\---



\# 13. Storage Object Naming



The current implementation uses the TUS upload/file identifier as the storage object key.



The original user-visible filename is stored separately:



```text

OriginalFileName

```



Therefore the physical storage identifier does not need to be the original filename.



Conceptually:



```text

User filename:

report.pdf



Storage identifier:

<TUS FileId>



Database:

OriginalFileName = report.pdf

StorageObjectKey = <TUS FileId>

```



This separation is important for future storage-provider migration.



\---



\# 14. Download Architecture



The current download endpoint is:



```text

GET /api/Transfer/download/{token}/{fileName}

```



The current flow is:



```text

Recipient

&#x20;  │

&#x20;  ▼

Secure transfer token

&#x20;  │

&#x20;  ▼

Transfer details

&#x20;  │

&#x20;  ▼

File metadata

&#x20;  │

&#x20;  ▼

Download request

&#x20;  │

&#x20;  ▼

Backend validates:

&#x20;  ├── transfer exists

&#x20;  ├── token matches

&#x20;  ├── transfer is not deleted

&#x20;  ├── transfer is not expired

&#x20;  └── requested filename belongs to transfer

&#x20;  │

&#x20;  ▼

Filesystem path

&#x20;  │

&#x20;  ▼

C:\\VASTRIX\_FileStore

&#x20;  │

&#x20;  ▼

FileStream

&#x20;  │

&#x20;  ▼

Recipient

```



\---



\# 15. Download Path Security



The current download implementation performs path containment validation.



The application constructs the expected physical path and verifies that the resolved path remains inside:



```text

C:\\VASTRIX\_FileStore

```



This is intended to prevent path traversal from escaping the storage root.



The storage root is therefore a security boundary.



Do not remove or weaken path-containment validation merely to make downloads work.



\---



\# 16. Current Download Filename Limitation



The current public download endpoint identifies the requested file using:



```text

Transfer token

\+

Original filename

```



This is functional for the current implementation but is not the preferred long-term design.



Potential issues include:



\* Duplicate filenames within the same transfer.

\* Filename encoding complexity.

\* Filename manipulation concerns.

\* Less precise object identification.



The preferred future design is to identify the stored file using an internal file identifier.



Conceptually:



```text

Current:



/download/{token}/{fileName}





Preferred future:



/download/{token}/{fileId}

```



The original filename should remain presentation metadata rather than the primary object identifier.



\---



\# 17. Current Recipient Download Security



The current download system validates:



\* Transfer token.

\* Transfer existence.

\* Soft-deletion state.

\* Expiration.

\* File association.

\* Filesystem path containment.

\* Physical file existence.



However, the following controls are \*\*not currently enforced\*\*:



```text

Recipient OTP verification

Transfer password

Maximum download count

Download count tracking

Recipient-specific download session

```



Therefore possession of a valid transfer token is currently sufficient to reach the download path, subject to the existing transfer/file checks.



Recipient verification must not be considered implemented merely because a verification flag exists in the database or frontend.



\---



\# 18. Cleanup Architecture



VASTRIX currently has a registered:



```text

DiskSweeperWorker

```



The worker runs approximately hourly.



Its responsibility includes processing expired transfers and associated files.



Conceptually:



```text

Hourly worker

&#x20;     │

&#x20;     ▼

Find expired transfers

&#x20;     │

&#x20;     ▼

Find associated files

&#x20;     │

&#x20;     ▼

Delete physical files

&#x20;     │

&#x20;     ▼

Mark transfer IsDeleted

```



\---



\# 19. Cleanup Limitations



The current cleanup process has several architectural limitations.



\## 19.1 Database and Filesystem Are Not Atomic



The filesystem and PostgreSQL transaction cannot currently be treated as one atomic transaction.



For example:



```text

Delete physical file

&#x20;     │

&#x20;     X

Database operation fails

```



or:



```text

Database marked deleted

&#x20;     │

&#x20;     X

Physical file deletion fails

```



can leave inconsistent state.



\---



\## 19.2 Orphan Files



A possible failure scenario is:



```text

TUS upload succeeds

&#x20;     │

&#x20;     ▼

Physical file exists

&#x20;     │

&#x20;     ▼

Database metadata creation fails

&#x20;     │

&#x20;     ▼

Physical file remains

```



This creates an orphan physical file.



The current system does not provide a complete orphan-file reconciliation system.



\---



\## 19.3 Locked Files



If a physical file cannot be deleted because it is locked or otherwise inaccessible, cleanup may still proceed with the database-side deletion state.



This means physical deletion failure and logical transfer deletion are not currently guaranteed to be one atomic operation.



\---



\## 19.4 Hardcoded Storage Root



Cleanup currently relies on the local filesystem storage root.



This will need to change when storage becomes provider-independent.



\---



\# 20. FileCleanupWorker



A `FileCleanupWorker` exists in the Infrastructure project.



However, it is currently \*\*not registered\*\* as an active hosted worker based on the verified implementation.



Therefore AI agents and developers must not assume that this worker is running.



The currently registered cleanup worker is:



```text

DiskSweeperWorker

```



\---



\# 21. Failure and Consistency Model



The current storage architecture can be summarized as:



```text

&#x20;                PostgreSQL

&#x20;             File Metadata

&#x20;                  ▲

&#x20;                  │

&#x20;                  │

&#x20;            Upload Complete

&#x20;                  │

&#x20;                  │

&#x20;            Local Filesystem

&#x20;         C:\\VASTRIX\_FileStore

&#x20;                  ▲

&#x20;                  │

&#x20;                  │

&#x20;               TUS

&#x20;                  ▲

&#x20;                  │

&#x20;               Browser

```



There are two separate states:



```text

Logical state

= PostgreSQL metadata



Physical state

= filesystem object

```



A robust production architecture must eventually include reconciliation between these two states.



\---



\# 22. Security Principles



The following principles apply to storage.



\## 22.1 Files Are Not Stored in PostgreSQL



Large file contents must remain outside the relational database.



PostgreSQL stores metadata only.



\---



\## 22.2 Files Must Not Be Publicly Accessible



The intended architecture is:



```text

Private storage

&#x20;     +

controlled application access

```



rather than:



```text

Public permanent file URL

```



\---



\## 22.3 Transfer Tokens Are Secrets



A transfer token grants access to transfer resources.



It should therefore be treated as sensitive capability data.



Tokens should not be unnecessarily exposed in logs or diagnostic output.



\---



\## 22.4 Ownership Comes From Authentication



For authenticated uploads, the backend determines the user from the authenticated JWT identity.



The client must not be trusted to specify another user's ownership relationship.



\---



\## 22.5 Backend Is the Security Authority



Frontend limits are not security controls.



For example:



```text

Frontend:

5 GiB validation



Backend:

5 GiB validation

quota validation

ownership validation

transfer validation

expiration validation

```



The backend controls the actual security boundary.



\---



\# 23. Current vs Planned Architecture



\## Current



```text

Browser

&#x20;  │

&#x20;  │ authenticated TUS

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  ▼

Local Windows Filesystem

C:\\VASTRIX\_FileStore



PostgreSQL

&#x20;  │

&#x20;  └── file metadata

```



\## Planned



```text

Browser

&#x20;  │

&#x20;  │ signed/resumable upload

&#x20;  ▼

Private Object Storage

&#x20;  │

&#x20;  └── file object



VASTRIX API

&#x20;  │

&#x20;  └── PostgreSQL metadata

```



The future architecture should remove the requirement for the API server to act as the long-term file-storage layer.



\---



\# 24. Planned Object Storage



The long-term architecture is intended to support private object storage.



Possible providers may include:



\* Amazon S3

\* Azure Blob Storage

\* Cloudflare R2

\* Another S3-compatible private object-storage provider



The provider has not been finalized.



Therefore provider-specific implementation must not be treated as currently implemented.



\---



\# 25. Planned Upload Model



The intended future upload architecture is:



```text

Browser

&#x20;  │

&#x20;  │ request upload authorization

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  │ creates authorized upload

&#x20;  ▼

Short-lived signed upload URL

&#x20;  │

&#x20;  ▼

Private Object Storage

```



This reduces the need for the API server to carry the entire file payload.



For large files, resumable/multipart upload should remain part of the design.



\---



\# 26. Planned Download Model



The intended future download architecture is:



```text

Recipient

&#x20;  │

&#x20;  ▼

VASTRIX API

&#x20;  │

&#x20;  ├── validate transfer

&#x20;  ├── validate recipient controls

&#x20;  ├── validate expiration

&#x20;  ├── validate download limits

&#x20;  │

&#x20;  ▼

Short-lived signed download URL

&#x20;  │

&#x20;  ▼

Private Object Storage

&#x20;  │

&#x20;  ▼

Recipient

```



Permanent public object URLs should not be used as the security model.



\---



\# 27. Planned File Lifecycle



The intended long-term file lifecycle is:



```text

Upload

&#x20;  │

&#x20;  ▼

PENDING

&#x20;  │

&#x20;  ▼

UPLOADING

&#x20;  │

&#x20;  ▼

UPLOADED

&#x20;  │

&#x20;  ▼

SCANNING

&#x20;  │

&#x20;  ├── malware detected

&#x20;  │        ▼

&#x20;  │    QUARANTINED

&#x20;  │

&#x20;  └── scan passed

&#x20;           ▼

&#x20;        AVAILABLE

&#x20;           │

&#x20;           ▼

&#x20;        EXPIRED

&#x20;           │

&#x20;           ▼

&#x20;        DELETED

```



`TransferStatus` is not currently implemented in the database.



This lifecycle is therefore an architectural target, not current behavior.



\---



\# 28. Planned Malware Scanning



The future storage architecture should support malware scanning before a transfer becomes downloadable.



Conceptually:



```text

Upload completed

&#x20;     │

&#x20;     ▼

SCANNING

&#x20;     │

&#x20;     ├── infected ──► QUARANTINED

&#x20;     │

&#x20;     └── clean ─────► AVAILABLE

```



The current implementation does not provide a complete malware-scanning pipeline.



\---



\# 29. Planned Reconciliation



A future storage service should periodically compare:



```text

Database metadata

&#x20;       vs

Physical/object storage

```



and identify:



\* Database record without physical object.

\* Physical object without database record.

\* Incorrect object size.

\* Unexpected storage objects.

\* Failed deletion.

\* Expired objects that remain stored.



This becomes particularly important when storage is externalized to object storage.



\---



\# 30. Storage Abstraction Goal



The long-term architecture should allow VASTRIX to change storage providers without rewriting transfer/business logic.



Conceptually:



```text

Transfer Service

&#x20;     │

&#x20;     ▼

Storage Service Interface

&#x20;     │

&#x20;     ├── Local Filesystem Provider

&#x20;     │

&#x20;     ├── S3 Provider

&#x20;     │

&#x20;     ├── Azure Blob Provider

&#x20;     │

&#x20;     └── Other Provider

```



The exact interface and provider implementation are not currently finalized.



Do not introduce an abstraction solely for theoretical flexibility unless it is needed by the implementation stage.



\---



\# 31. Implemented / Partial / Planned Matrix



| Capability                     | Status               | Notes                                           |

| ------------------------------ | -------------------- | ----------------------------------------------- |

| Local filesystem storage       | Implemented          | `C:\\VASTRIX\_FileStore`                          |

| PostgreSQL metadata storage    | Implemented          | File contents are not stored in DB              |

| TUS resumable upload           | Implemented          | Backend uses tusdotnet                          |

| 5 GiB individual upload limit  | Implemented          | Backend enforced                                |

| User storage quota             | Implemented          | Current quota model exists                      |

| Final upload quota recheck     | Implemented          | Performed during completion                     |

| StorageObjectKey               | Implemented          | Uses current TUS FileId                         |

| Upload ownership validation    | Implemented          | Based on authenticated user                     |

| Transfer expiration validation | Implemented          | Upload/download paths validate expiration       |

| Download path containment      | Implemented          | Local storage security check                    |

| Anonymous token download       | Implemented          | Subject to current transfer checks              |

| Filename-based download        | Implemented          | Long-term replacement recommended               |

| Recipient OTP enforcement      | Missing              | Database foundation exists, enforcement missing |

| Transfer password              | Missing              | Not currently enforced                          |

| Download limits                | Missing              | No max-download model                           |

| Download counter               | Missing              | Not implemented                                 |

| Transfer status lifecycle      | Missing              | Architectural target only                       |

| Malware scanning               | Missing              | Planned                                         |

| Orphan-file reconciliation     | Missing              | Planned                                         |

| Object storage                 | Planned              | Not implemented                                 |

| Signed upload URLs             | Planned              | Not implemented                                 |

| Signed download URLs           | Planned              | Not implemented                                 |

| Storage provider abstraction   | Planned              | Not finalized                                   |

| Atomic DB/storage operations   | Partial              | Filesystem and DB cannot be fully atomic        |

| Cleanup worker                 | Implemented          | DiskSweeperWorker registered                    |

| FileCleanupWorker              | Present but inactive | Exists but not registered                       |

| Storage monitoring             | Planned              | Not implemented                                 |

| Storage audit/reconciliation   | Planned              | Not implemented                                 |



\---



\# 32. Important Current Limitations



The following limitations must remain visible to AI agents and developers:



1\. Storage is currently local filesystem based.

2\. The storage root is hardcoded.

3\. PostgreSQL and filesystem operations are not one atomic transaction.

4\. Orphan physical files are possible.

5\. Cleanup is not a complete reconciliation system.

6\. Filename-based download identification is weak for duplicate filenames.

7\. Recipient verification is not currently enforced on downloads.

8\. Password protection is not implemented.

9\. Download limits and counters are not implemented.

10\. Malware scanning is not implemented.

11\. Object storage is not implemented.

12\. Signed upload/download URLs are not implemented.

13\. `TransferStatus` is not implemented.

14\. `FileCleanupWorker` exists but is not registered.



\---



\# 33. AI Development Rules



When working on VASTRIX storage functionality, AI agents must follow these rules.



\### Rule 1 — Do Not Assume Object Storage Exists



Current storage is:



```text

C:\\VASTRIX\_FileStore

```



Do not write code assuming S3, Azure Blob, R2, or another object-storage provider is already configured.



\---



\### Rule 2 — Do Not Store Large Files in PostgreSQL



The database is for metadata.



Do not introduce database BLOB storage for normal VASTRIX file transfers.



\---



\### Rule 3 — Do Not Remove Security Validation



Do not remove:



\* Authentication checks.

\* Ownership checks.

\* Transfer expiration checks.

\* Path containment checks.

\* Quota validation.

\* Final upload validation.



to make an upload or download test pass.



\---



\### Rule 4 — Do Not Treat Frontend Validation as Security



A frontend 5 GiB limit does not replace backend enforcement.



\---



\### Rule 5 — Do Not Assume Cleanup Is Perfect



Any storage change must consider:



```text

Database failure

Filesystem failure

Partial upload

Abandoned upload

Orphan object

Locked file

Failed deletion

```



\---



\### Rule 6 — Preserve StorageObjectKey Semantics



Until the storage architecture is deliberately redesigned, `StorageObjectKey` must remain the application's metadata reference to the stored object.



\---



\### Rule 7 — Do Not Implement Future Architecture Prematurely



Object storage, signed URLs, malware scanning, reconciliation, queues, and storage abstractions should be implemented when their corresponding development phase is reached.



Do not introduce unnecessary infrastructure merely because it appears in the long-term architecture.



\---



\# 34. Target Production Architecture



The intended production architecture is approximately:



```text

&#x20;                        ┌─────────────────────┐

&#x20;                        │      Browser        │

&#x20;                        └──────────┬──────────┘

&#x20;                                   │

&#x20;                        authenticated API

&#x20;                                   │

&#x20;                                   ▼

&#x20;                        ┌─────────────────────┐

&#x20;                        │    VASTRIX API      │

&#x20;                        │                     │

&#x20;                        │ Auth                │

&#x20;                        │ Transfer            │

&#x20;                        │ Security            │

&#x20;                        │ Storage orchestration│

&#x20;                        └───────┬───────┬─────┘

&#x20;                                │       │

&#x20;                        metadata│       │signed access

&#x20;                                │       │

&#x20;                                ▼       ▼

&#x20;                      ┌─────────────┐  ┌─────────────────┐

&#x20;                      │ PostgreSQL  │  │ Private Object  │

&#x20;                      │             │  │ Storage         │

&#x20;                      │ Users       │  │                 │

&#x20;                      │ Transfers   │  │ File Objects    │

&#x20;                      │ Files       │  │                 │

&#x20;                      │ Challenges  │  │                 │

&#x20;                      └─────────────┘  └─────────────────┘

```



Additional infrastructure can later be introduced around this:



```text

Redis

Queues

Workers

Malware Scanner

Monitoring

Audit Logging

```



but these are not all currently implemented.



\---



\# 35. Architectural Principle



The core storage principle for VASTRIX is:



> \*\*PostgreSQL knows what the file is; private storage holds the file itself; the API controls who may access it.\*\*



The long-term goal is:



```text

Metadata

&#x20;   ↓

PostgreSQL



File contents

&#x20;   ↓

Private object storage



Authorization

&#x20;   ↓

VASTRIX API



Temporary file access

&#x20;   ↓

Short-lived signed URLs

```



This separation should remain a central architectural principle as VASTRIX evolves.



