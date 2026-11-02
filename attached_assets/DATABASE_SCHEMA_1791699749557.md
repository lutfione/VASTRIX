\# VASTRIX — Database Schema



\## 1. Purpose



This document describes the \*\*current verified VASTRIX PostgreSQL database schema\*\*.



It documents:



\* Current database entities.

\* Tables and relationships.

\* Columns and data types.

\* Primary keys.

\* Foreign keys.

\* Indexes.

\* Default values.

\* Current database responsibilities.

\* Implemented versus missing product fields.

\* Migration history relevant to the current schema.



This document describes the \*\*actual current schema\*\*.



It must not be confused with the future product model or planned architecture.



\---



\# 2. Database Technology



Current database:



```text

PostgreSQL

```



Application access is implemented using:



```text

Entity Framework Core

\+

Npgsql

```



The backend targets:



```text

.NET 9

```



The Infrastructure project contains the EF Core database context and migrations.



\---



\# 3. Database Responsibilities



PostgreSQL is responsible for persistent application metadata.



It stores:



\* User accounts.

\* Authentication-related user data.

\* User roles.

\* User disabled state.

\* Storage quotas.

\* Transfer metadata.

\* Secure transfer tokens.

\* Transfer expiration.

\* File metadata.

\* Storage object references.

\* Recipient verification challenge data.



PostgreSQL does \*\*not\*\* store the actual large-file contents.



The physical file contents are currently stored on the local filesystem.



```text

PostgreSQL

&#x20;   │

&#x20;   └── Metadata



Filesystem

&#x20;   │

&#x20;   └── Actual file contents

```



\---



\# 4. Current Entity Model



The current domain entities are:



```text

User

UserRole

Transfer

TransferFile

RecipientVerificationChallenge

```



The `UserRole` is an enum rather than a database entity/table.



\---



\# 5. Current DbSets



The `ApplicationDbContext` currently exposes:



```text

Users

Transfers

Files

RecipientVerificationChallenges

```



Conceptually:



```text

ApplicationDbContext

│

├── Users

├── Transfers

├── Files

└── RecipientVerificationChallenges

```



\---



\# 6. Entity Relationship Diagram



The current relationship model is:



```text

┌─────────────────────┐

│       Users         │

├─────────────────────┤

│ Id                  │

│ Username            │

│ Email               │

│ PasswordHash        │

│ Role                │

│ IsDisabled          │

│ StorageQuotaBytes   │

│ CreatedAt           │

└──────────┬──────────┘

&#x20;          │

&#x20;          │ 1 : N

&#x20;          ▼

┌─────────────────────┐

│      Transfers      │

├─────────────────────┤

│ Id                  │

│ UserId              │

│ SecureToken         │

│ CustomMessage       │

│ PasswordHash        │

│ RequireRecipient... │

│ CreatedAt           │

│ ExpiresAt           │

│ IsDeleted           │

└──────────┬──────────┘

&#x20;          │

&#x20;          ├─────────────────────┐

&#x20;          │                     │

&#x20;          │ 1 : N               │ 1 : N

&#x20;          ▼                     ▼

┌─────────────────────┐   ┌──────────────────────────────┐

│    TransferFiles    │   │ RecipientVerification...     │

├─────────────────────┤   ├──────────────────────────────┤

│ Id                  │   │ Id                           │

│ TransferId          │   │ TransferId                   │

│ OriginalFileName    │   │ RecipientEmail               │

│ FileSizeBytes       │   │ CodeHash                     │

│ ContentType         │   │ CodeExpiresAt                │

│ StorageObjectKey    │   │ AttemptCount                 │

│ UploadedAt          │   │ CreatedAt                    │

└─────────────────────┘   │ ConsumedAt                   │

&#x20;                         └──────────────────────────────┘

```



\---



\# 7. Users Table



The `Users` table represents authenticated VASTRIX users.



Current entity:



```csharp

public class User

{

&#x20;   public Guid Id { get; set; } = Guid.NewGuid();



&#x20;   public string Username { get; set; } = string.Empty;



&#x20;   public string Email { get; set; } = string.Empty;



&#x20;   public string PasswordHash { get; set; } = string.Empty;



&#x20;   public UserRole Role { get; set; } = UserRole.User;



&#x20;   public bool IsDisabled { get; set; } = false;



&#x20;   public long StorageQuotaBytes { get; set; } = 5368709120;



&#x20;   public DateTime CreatedAt { get; set; } = DateTime.UtcNow;



&#x20;   public ICollection<Transfer> Transfers { get; set; }

&#x20;       = new List<Transfer>();

}

```



\---



\# 8. Users Columns



| Column            | PostgreSQL Type | Nullable | Current Purpose            |

| ----------------- | --------------- | -------: | -------------------------- |

| Id                | uuid            |       No | User primary key           |

| Username          | text            |       No | User login/name            |

| Email             | text            |       No | User email                 |

| PasswordHash      | text            |       No | BCrypt password hash       |

| Role              | integer         |       No | User role enum             |

| IsDisabled        | boolean         |       No | Account disabled state     |

| StorageQuotaBytes | bigint          |       No | User storage quota         |

| CreatedAt         | timestamptz     |       No | Account creation timestamp |



\---



\# 9. Users Primary Key



Primary key:



```text

Users.Id

```



Type:



```text

uuid

```



The application generates a GUID for new users.



\---



\# 10. Users Unique Constraints / Indexes



The current schema contains unique indexes for:



```text

Users.Email

Users.Username

```



Conceptually:



```text

IX\_Users\_Email

IX\_Users\_Username

```



Both are unique.



This prevents multiple user accounts from using the same email or username.



\---



\# 11. User Role



The application currently defines:



```csharp

public enum UserRole

{

&#x20;   User = 0,

&#x20;   Admin = 1

}

```



The PostgreSQL column is:



```text

Role integer

```



The enum is therefore represented numerically in the database.



Current role values:



```text

0 = User

1 = Admin

```



\---



\# 12. User Disabled State



The database contains:



```text

IsDisabled boolean

```



Default:



```text

false

```



A disabled user is not considered an active authenticated account by the relevant backend authentication checks.



\---



\# 13. User Storage Quota



The database contains:



```text

StorageQuotaBytes bigint

```



Current entity default:



```text

5,368,709,120 bytes

```



Equivalent:



```text

5 GiB

```



The quota is used by the upload system when determining whether the user can store additional files.



\---



\# 14. Transfers Table



The `Transfers` table represents a logical file transfer.



Current entity:



```csharp

public class Transfer

{

&#x20;   public Guid Id { get; set; } = Guid.NewGuid();



&#x20;   public Guid? UserId { get; set; }



&#x20;   public User? User { get; set; }



&#x20;   public string SecureToken { get; set; }

&#x20;       = Guid.NewGuid().ToString("N");



&#x20;   public string? CustomMessage { get; set; }



&#x20;   public string? PasswordHash { get; set; }



&#x20;   public bool RequireRecipientVerification { get; set; } = false;



&#x20;   public DateTime CreatedAt { get; set; }

&#x20;       = DateTime.UtcNow;



&#x20;   public DateTime ExpiresAt { get; set; }



&#x20;   public bool IsDeleted { get; set; } = false;



&#x20;   public ICollection<TransferFile> Files { get; set; }

&#x20;       = new List<TransferFile>();



&#x20;   public ICollection<RecipientVerificationChallenge>

&#x20;       RecipientVerificationChallenges { get; set; }

&#x20;       = new List<RecipientVerificationChallenge>();

}

```



\---



\# 15. Transfers Columns



| Column                       | PostgreSQL Type | Nullable | Current Purpose                            |

| ---------------------------- | --------------- | -------: | ------------------------------------------ |

| Id                           | uuid            |       No | Transfer primary key                       |

| UserId                       | uuid            |      Yes | Owner user                                 |

| SecureToken                  | text            |       No | Secure transfer access token               |

| CustomMessage                | text            |      Yes | Optional sender message                    |

| PasswordHash                 | text            |      Yes | Password protection foundation             |

| RequireRecipientVerification | boolean         |       No | Indicates requested recipient verification |

| CreatedAt                    | timestamptz     |       No | Transfer creation timestamp                |

| ExpiresAt                    | timestamptz     |       No | Transfer expiration                        |

| IsDeleted                    | boolean         |       No | Soft-delete state                          |



\---



\# 16. Transfer Primary Key



Primary key:



```text

Transfers.Id

```



Type:



```text

uuid

```



\---



\# 17. Transfer Ownership



The current relationship is:



```text

User 1 ─────── N Transfer

```



`Transfers.UserId` is nullable.



Relationship:



```text

Transfers.UserId

&#x20;      │

&#x20;      ▼

Users.Id

```



The backend normally establishes ownership using the authenticated user's identity.



\---



\# 18. Transfer Ownership Index



The current schema contains an index on:



```text

Transfers.UserId

```



Conceptually:



```text

IX\_Transfers\_UserId

```



This supports queries involving transfers belonging to a user.



\---



\# 19. Secure Transfer Token



The current transfer contains:



```text

SecureToken

```



The entity initializes it using:



```text

Guid.NewGuid().ToString("N")

```



The database has a unique index:



```text

IX\_Transfers\_SecureToken

```



The token is used by anonymous transfer-access endpoints.



Because the token functions as a capability for accessing a transfer, it should be treated as sensitive data.



\---



\# 20. Transfer Expiration



The database contains:



```text

ExpiresAt timestamptz

```



The backend checks transfer expiration in relevant transfer and download operations.



The current transfer creation behavior uses a default of 7 days when the supplied expiration is not positive and limits the value to a maximum of 30 days.



The frontend currently exposes:



```text

1 day

3 days

7 days

14 days

30 days

```



The frontend/backend expiration contract is not yet fully standardized.



\---



\# 21. Transfer Soft Deletion



The current schema contains:



```text

IsDeleted boolean

```



Default:



```text

false

```



VASTRIX currently uses soft deletion for transfers.



A deleted transfer should not normally be accessible through the standard transfer/download APIs.



Physical file cleanup is handled separately.



\---



\# 22. Transfer Password Field



The current `Transfer` entity contains:



```text

PasswordHash

```



This indicates that the database has a place to store a transfer password hash.



However:



> The existence of `PasswordHash` does not mean transfer password protection is fully implemented.



The current backend transfer creation flow does not provide a complete transfer-password feature.



The database field should therefore be considered \*\*partially prepared / unused for the complete feature\*\* until the complete API and download enforcement workflow exists.



\---



\# 23. Recipient Verification Flag



The current transfer contains:



```text

RequireRecipientVerification

```



Default:



```text

false

```



This records whether recipient verification was requested.



However:



> This field alone does not enforce recipient verification.



The current recipient verification system is incomplete.



\---



\# 24. Missing Transfer Controls



The current `Transfer` entity does not contain fields for:



```text

TransferStatus

RecipientEmail

MaxDownloads

DownloadCount

```



These are relevant to the intended product but are not currently implemented in the transfer model.



Future implementation must be based on an explicit design decision rather than assuming these fields already exist.



\---



\# 25. TransferFiles Table



The database entity is:



```csharp

public class TransferFile

{

&#x20;   public Guid Id { get; set; } = Guid.NewGuid();



&#x20;   public Guid TransferId { get; set; }



&#x20;   public Transfer Transfer { get; set; } = null!;



&#x20;   public string OriginalFileName { get; set; }

&#x20;       = string.Empty;



&#x20;   public long FileSizeBytes { get; set; }



&#x20;   public string ContentType { get; set; }

&#x20;       = string.Empty;



&#x20;   public string StorageObjectKey { get; set; }

&#x20;       = string.Empty;



&#x20;   public DateTime UploadedAt { get; set; }

&#x20;       = DateTime.UtcNow;

}

```



The EF DbSet is exposed as:



```text

Files

```



The underlying database table is the file metadata table.



\---



\# 26. TransferFiles Columns



| Column           | PostgreSQL Type | Nullable | Current Purpose                   |

| ---------------- | --------------- | -------: | --------------------------------- |

| Id               | uuid            |       No | File metadata primary key         |

| TransferId       | uuid            |       No | Parent transfer                   |

| OriginalFileName | text            |       No | Original user-visible filename    |

| FileSizeBytes    | bigint          |       No | File size                         |

| ContentType      | text            |       No | MIME/content type                 |

| StorageObjectKey | text            |       No | Physical/object storage reference |

| UploadedAt       | timestamptz     |       No | Upload completion timestamp       |



\---



\# 27. TransferFile Primary Key



Primary key:



```text

Files.Id

```



Type:



```text

uuid

```



The identifier is generated by the application.



\---



\# 28. TransferFile Relationship



The current relationship is:



```text

Transfer 1 ─────── N TransferFile

```



Foreign key:



```text

Files.TransferId

&#x20;      │

&#x20;      ▼

Transfers.Id

```



The relationship is required because every `TransferFile` belongs to a transfer.



\---



\# 29. TransferFile Delete Behavior



The current relationship uses cascade delete.



Conceptually:



```text

Delete Transfer

&#x20;      │

&#x20;      ▼

Associated TransferFiles

&#x20;      │

&#x20;      ▼

Database metadata deleted

```



This database relationship does not automatically mean that the physical file is deleted.



Physical filesystem cleanup is handled by application code.



\---



\# 30. TransferFile Index



The current schema contains:



```text

IX\_Files\_TransferId

```



This index supports retrieving files associated with a transfer.



\---



\# 31. Original Filename



The database stores:



```text

OriginalFileName

```



This is presentation metadata.



The physical storage identifier is represented separately by:



```text

StorageObjectKey

```



The current download endpoint still uses the original filename as part of the URL.



Long-term, a file ID is preferable as the primary download identifier.



\---



\# 32. File Size



The database stores:



```text

FileSizeBytes bigint

```



The value is used by the application for storage quota calculations.



The TUS completion flow performs validation against the actual physical file size before creating the final metadata record.



\---



\# 33. Content Type



The database stores:



```text

ContentType text

```



This records the file's content/MIME type metadata.



The value originates from upload metadata.



It should not be treated as proof that the physical file contents are safe or that the declared type is trustworthy.



Future malware/content validation should operate independently.



\---



\# 34. StorageObjectKey



The database stores:



```text

StorageObjectKey

```



This identifies the stored physical/object-storage object.



Current implementation:



```text

StorageObjectKey

&#x20;       =

TUS FileId

```



The current physical storage provider is the local Windows filesystem.



Future object storage can use the same conceptual metadata field to reference an object key.



\---



\# 35. RecipientVerificationChallenges Table



The current entity is:



```csharp

public class RecipientVerificationChallenge

{

&#x20;   public Guid Id { get; set; } = Guid.NewGuid();



&#x20;   public Guid TransferId { get; set; }



&#x20;   public Transfer Transfer { get; set; } = null!;



&#x20;   public string RecipientEmail { get; set; }

&#x20;       = string.Empty;



&#x20;   public string CodeHash { get; set; }

&#x20;       = string.Empty;



&#x20;   public DateTime CodeExpiresAt { get; set; }



&#x20;   public int AttemptCount { get; set; } = 0;



&#x20;   public DateTime CreatedAt { get; set; }

&#x20;       = DateTime.UtcNow;



&#x20;   public DateTime? ConsumedAt { get; set; }

}

```



\---



\# 36. RecipientVerificationChallenges Columns



| Column         | PostgreSQL Type | Nullable | Current Purpose            |

| -------------- | --------------- | -------: | -------------------------- |

| Id             | uuid            |       No | Challenge primary key      |

| TransferId     | uuid            |       No | Associated transfer        |

| RecipientEmail | text            |       No | Recipient email            |

| CodeHash       | text            |       No | Hashed verification code   |

| CodeExpiresAt  | timestamptz     |       No | Code expiration            |

| AttemptCount   | integer         |       No | Verification attempts      |

| CreatedAt      | timestamptz     |       No | Challenge creation time    |

| ConsumedAt     | timestamptz     |      Yes | Challenge consumption time |



\---



\# 37. Recipient Verification Relationship



The relationship is:



```text

Transfer 1 ─────── N RecipientVerificationChallenge

```



Foreign key:



```text

RecipientVerificationChallenges.TransferId

&#x20;      │

&#x20;      ▼

Transfers.Id

```



Cascade delete is configured.



\---



\# 38. Recipient Verification Index



The current schema contains:



```text

IX\_RecipientVerificationChallenges\_TransferId\_RecipientEmail

```



This index covers:



```text

TransferId

RecipientEmail

```



It is \*\*not unique\*\*.



Therefore multiple verification challenges can exist for the same transfer/recipient combination.



\---



\# 39. Recipient Verification Status



The database provides fields necessary for a challenge lifecycle:



```text

CreatedAt

CodeExpiresAt

AttemptCount

ConsumedAt

```



However, the complete business workflow is not currently implemented.



Missing application-level pieces include:



\* Complete code generation.

\* Challenge creation endpoint.

\* Challenge verification endpoint.

\* Attempt-limit enforcement.

\* Expiration enforcement workflow.

\* Consumed-state enforcement.

\* Recipient authorization/session state.

\* Download enforcement.



Therefore the database should be described as:



```text

Verification data foundation: Implemented

Complete recipient verification feature: Not implemented

```



\---



\# 40. PostgreSQL Data Type Mapping



Current EF/Core to PostgreSQL mappings include:



| C# Type           | PostgreSQL Type      |

| ----------------- | -------------------- |

| Guid              | uuid                 |

| string            | text                 |

| long              | bigint               |

| bool              | boolean              |

| DateTime          | timestamptz          |

| int               | integer              |

| nullable Guid     | nullable uuid        |

| nullable DateTime | nullable timestamptz |

| nullable string   | nullable text        |



\---



\# 41. Current Index Inventory



The current important indexes are:



```text

IX\_Transfers\_SecureToken

&#x20;   Unique



IX\_Transfers\_UserId

&#x20;   Non-unique



IX\_Files\_TransferId

&#x20;   Non-unique



IX\_Users\_Email

&#x20;   Unique



IX\_Users\_Username

&#x20;   Unique



IX\_RecipientVerificationChallenges\_TransferId\_RecipientEmail

&#x20;   Non-unique

```



Indexes should not be added or removed without considering actual query patterns and migration impact.



\---



\# 42. Foreign Key Summary



Current relationships:



| Child                                      | Parent       | Cardinality | Delete Behavior       |

| ------------------------------------------ | ------------ | ----------- | --------------------- |

| Transfers.UserId                           | Users.Id     | N:1         | Nullable relationship |

| Files.TransferId                           | Transfers.Id | N:1         | Cascade               |

| RecipientVerificationChallenges.TransferId | Transfers.Id | N:1         | Cascade               |



\---



\# 43. Current Schema Snapshot



The current database can be summarized as:



```text

Users

│

├── Id

├── Username

├── Email

├── PasswordHash

├── Role

├── IsDisabled

├── StorageQuotaBytes

└── CreatedAt

&#x20;    │

&#x20;    │

&#x20;    └───────────────┐

&#x20;                    ▼

&#x20;                Transfers

&#x20;                │

&#x20;                ├── Id

&#x20;                ├── UserId

&#x20;                ├── SecureToken

&#x20;                ├── CustomMessage

&#x20;                ├── PasswordHash

&#x20;                ├── RequireRecipientVerification

&#x20;                ├── CreatedAt

&#x20;                ├── ExpiresAt

&#x20;                └── IsDeleted

&#x20;                      │

&#x20;             ┌────────┴─────────┐

&#x20;             ▼                  ▼

&#x20;         Files             Verification

&#x20;         │                  Challenges

&#x20;         ├── Id             ├── Id

&#x20;         ├── TransferId     ├── TransferId

&#x20;         ├── Original...    ├── RecipientEmail

&#x20;         ├── FileSize...    ├── CodeHash

&#x20;         ├── ContentType    ├── CodeExpiresAt

&#x20;         ├── Storage...     ├── AttemptCount

&#x20;         └── UploadedAt     ├── CreatedAt

&#x20;                            └── ConsumedAt

```



\---



\# 44. Current Product-to-Database Mapping



| Product Requirement            | Current Database Support               | Status      |

| ------------------------------ | -------------------------------------- | ----------- |

| User accounts                  | Users                                  | Implemented |

| Username                       | Users.Username                         | Implemented |

| Email                          | Users.Email                            | Implemented |

| Password authentication        | Users.PasswordHash                     | Implemented |

| User roles                     | Users.Role                             | Implemented |

| Disabled accounts              | Users.IsDisabled                       | Implemented |

| Storage quota                  | Users.StorageQuotaBytes                | Implemented |

| Secure transfers               | Transfers.SecureToken                  | Implemented |

| Transfer expiration            | Transfers.ExpiresAt                    | Implemented |

| Custom transfer message        | Transfers.CustomMessage                | Implemented |

| Soft deletion                  | Transfers.IsDeleted                    | Implemented |

| Password protection            | Transfers.PasswordHash                 | Partial     |

| Recipient verification request | Transfers.RequireRecipientVerification | Partial     |

| Recipient OTP data             | VerificationChallenges                 | Partial     |

| File metadata                  | Files                                  | Implemented |

| File storage reference         | Files.StorageObjectKey                 | Implemented |

| Multiple files per transfer    | Transfer → Files                       | Implemented |

| Transfer status                | —                                      | Missing     |

| Recipient email on transfer    | —                                      | Missing     |

| Maximum downloads              | —                                      | Missing     |

| Download count                 | —                                      | Missing     |

| Malware status                 | —                                      | Missing     |

| Audit events                   | —                                      | Missing     |



\---



\# 45. Important Schema Distinctions



The following distinctions are important.



\## Database Field ≠ Complete Feature



A field may exist before the complete business feature exists.



Examples:



```text

Transfers.PasswordHash

Transfers.RequireRecipientVerification

RecipientVerificationChallenges

```



These indicate database preparation/foundation, not necessarily complete functionality.



\---



\## Database Metadata ≠ Physical File



`Files.StorageObjectKey` identifies the stored object.



It does not contain the actual file.



\---



\## Soft Delete ≠ Physical Delete



`Transfers.IsDeleted` represents logical deletion.



The physical file requires separate filesystem/object-storage cleanup.



\---



\# 46. Migration History



The current schema has evolved through these verified migrations.



\## 46.1 Initial PostgreSQL Setup



```text

20260830155238\_InitialPostgresSetup

```



Created the initial:



```text

Users

Transfers

Files

```



schema.



\---



\## 46.2 User Schema Constraints



```text

20260831013412\_UpdateUserSchemaConstraints

```



Included:



\* `Users.Name` → `Users.Username`.

\* Unique `Users.Email`.

\* Unique `Users.Username`.



\---



\## 46.3 User Roles



```text

20260909004514\_AddUserRoles

```



Added:



```text

Users.Role

```



with default role:



```text

0

```



\---



\## 46.4 User Disabled Status



```text

20260909174806\_AddUserDisabledStatus

```



Added:



```text

Users.IsDisabled

```



with default:



```text

false

```



\---



\## 46.5 Recipient Verification Challenge



```text

20260914032002\_AddRecipientVerificationChallenge

```



Added:



```text

RecipientVerificationChallenges

```



including:



\* Recipient email.

\* Hashed verification code.

\* Expiration.

\* Attempt count.

\* Consumed timestamp.



\---



\# 47. Migration / Model Synchronization



The current EF Core model has been verified as synchronized with migrations.



The command used was:



```powershell

dotnet ef migrations has-pending-model-changes `

&#x20;   --project "C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.Infrastructure" `

&#x20;   --startup-project "C:\\Akhil\\VASTRIX\\backend\\VASTRIX\\VASTRIX.API"

```



Verified result:



```text

No changes have been made to the model since the last migration.

```



This confirms that the checked EF model does not currently have un-migrated model changes.



\---



\# 48. Current Schema Status



Overall database status:



```text

Users                         IMPLEMENTED

User roles                    IMPLEMENTED

Disabled users                IMPLEMENTED

Storage quota                 IMPLEMENTED

Transfers                     IMPLEMENTED

Transfer expiration           IMPLEMENTED

Transfer soft deletion        IMPLEMENTED

File metadata                 IMPLEMENTED

Storage object reference      IMPLEMENTED

Recipient challenge storage   IMPLEMENTED

Password feature              PARTIAL

Recipient verification        PARTIAL

Transfer status               MISSING

Download controls             MISSING

Audit model                   MISSING

Malware state                 MISSING

```



\---



\# 49. Future Schema Direction



Future schema changes may eventually introduce fields/entities for:



```text

TransferStatus

RecipientEmail

MaxDownloads

DownloadCount

RecipientAccessSession

MalwareScanStatus

MalwareScanTimestamp

AuditEvents

Storage reconciliation

```



These are architectural possibilities/requirements and are \*\*not current schema objects\*\*.



They must only be added through deliberate database design and EF migrations.



\---



\# 50. Database Development Rules



AI assistants and developers must follow these rules when changing the VASTRIX database.



\### Rule 1 — Verify the Existing Model First



Before adding a new entity or field, verify whether it already exists in:



```text

VASTRIX.Core

ApplicationDbContext

EF migrations

```



\---



\### Rule 2 — Do Not Assume Planned Fields Exist



For example, do not write code using:



```text

Transfer.Status

Transfer.MaxDownloads

Transfer.DownloadCount

Transfer.RecipientEmail

```



without first implementing the corresponding model/migration.



\---



\### Rule 3 — Use EF Migrations



Schema changes must be represented through Entity Framework Core migrations.



Do not make undocumented manual schema changes and leave the migration history inconsistent.



\---



\### Rule 4 — Do Not Store Large Files in PostgreSQL



The database stores file metadata.



Actual file contents belong in the storage layer.



\---



\### Rule 5 — Preserve Ownership Relationships



The ownership chain is:



```text

User

&#x20; ↓

Transfer

&#x20; ↓

TransferFile

```



Security-sensitive queries must preserve this relationship.



\---



\### Rule 6 — Do Not Remove Security Indexes Casually



Unique indexes on:



```text

Users.Email

Users.Username

Transfers.SecureToken

```



are part of the current data-integrity/security model.



\---



\### Rule 7 — Verify Migration State



After schema changes, verify that:



```text

EF model

&#x20;     =

latest migration

```



before considering the database change complete.



\---



\# 51. AI Interpretation Rule



When reading this file, AI agents should distinguish:



```text

CURRENT SCHEMA

```



from:



```text

PRODUCT REQUIREMENT

```



and:



```text

FUTURE SCHEMA

```



The current schema is authoritative for what exists today.



Future fields listed in this document must not be treated as existing database properties.



\---



\# 52. Final Database Principle



The VASTRIX database is primarily a \*\*metadata and application-state database\*\*.



The fundamental separation is:



```text

PostgreSQL

&#x20;   │

&#x20;   ├── Users

&#x20;   ├── Transfers

&#x20;   ├── File metadata

&#x20;   └── Verification metadata



Storage Layer

&#x20;   │

&#x20;   └── Actual large-file contents

```



This separation must be preserved as VASTRIX evolves toward private object storage.



