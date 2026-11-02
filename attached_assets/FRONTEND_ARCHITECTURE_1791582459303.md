\# VASTRIX — Frontend Architecture



\*\*Document Type:\*\* Project Memory

\*\*Product:\*\* VASTRIX

\*\*Frontend:\*\* React 19 + Vite 8

\*\*Language:\*\* JavaScript / JSX

\*\*Routing:\*\* React Router

\*\*Styling:\*\* Tailwind CSS

\*\*Status:\*\* Source-derived baseline

\*\*Last Updated:\*\* 2026-09-17



\---



\# 1. Purpose



This document records the actual frontend architecture currently present in the VASTRIX repository.



It exists to help:



\* Human developers understand the frontend.

\* AI coding assistants understand the current project structure.

\* Troubleshoot frontend/backend integration.

\* Prevent outdated architectural assumptions.

\* Track implemented versus planned frontend functionality.



Detailed implementation information should remain in the source code and supporting technical documentation.



\---



\# 2. Frontend Location



The actual VASTRIX frontend is located at:



```text

C:\\Akhil\\VASTRIX\\frontend

```



The frontend is a separate application from the ASP.NET Core backend.



Repository-level structure:



```text

C:\\Akhil\\VASTRIX

├── backend

├── frontend

├── docs

├── PROJECT\_MEMORY

├── Db Backup

├── security-test.txt

└── swagger.json

```



\---



\# 3. Important Architecture Correction



The current frontend is:



```text

React + Vite + JavaScript/JSX

```



It is \*\*not currently Next.js\*\*.



It is also \*\*not currently TypeScript\*\*.



Any older documentation or memory that describes the current frontend as:



```text

Next.js

TypeScript

```



must be treated as outdated unless the actual source code is changed later.



Current frontend technology:



```text

Vite

React

JavaScript / JSX

React Router

Tailwind CSS

Framer Motion

Lucide React

```



\---



\# 4. Frontend Package Configuration



The current package configuration includes:



```text

React                  19.2.8

React DOM              19.2.8

Vite                   8.2.2

React Router DOM       7.18.3

Tailwind CSS tooling

Framer Motion          13.2.0

Lucide React           1.43.0

ESLint                 10.9.0

@vitejs/plugin-react   6.1.1

```



The application uses ES modules.



The package scripts currently include:



```text

npm run dev

npm run build

npm run lint

npm run preview

```



\---



\# 5. Frontend Application Structure



The important current source structure is:



```text

frontend\\

├── index.html

├── package.json

├── package-lock.json

├── vite.config.js

├── tailwind.config.js

├── eslint.config.js

├── README.md

└── src\\

&#x20;   ├── main.jsx

&#x20;   ├── App.jsx

&#x20;   ├── DownloadPage.jsx

&#x20;   ├── App.css

&#x20;   └── index.css

```



Additional repository files/folders exist, but the primary application flow is currently centered around the `src` directory.



\---



\# 6. Application Entry Point



The frontend entry point is:



```text

src\\main.jsx

```



The application uses:



```text

ReactDOM.createRoot(...)

```



and wraps the application in:



```text

React.StrictMode

BrowserRouter

```



The current routing structure is:



```text

BrowserRouter

&#x20;   │

&#x20;   └── Routes

&#x20;        │

&#x20;        ├── /

&#x20;        │    └── App.jsx

&#x20;        │

&#x20;        └── /t/:token

&#x20;             └── DownloadPage.jsx

```



\---



\# 7. Current Routes



\## 7.1 Landing / Transfer Route



```text

/

```



Component:



```text

App.jsx

```



This route currently contains the primary transfer creation interface.



\---



\## 7.2 Transfer Download Route



```text

/t/:token

```



Component:



```text

DownloadPage.jsx

```



The route uses the transfer's secure token as a URL parameter.



Example conceptual URL:



```text

https://frontend-host/t/<secure-token>

```



The actual production hostname is environment-dependent.



\---



\# 8. App.jsx Responsibilities



`App.jsx` currently contains most of the primary transfer UI and transfer workflow.



Its responsibilities include:



\* Landing page.

\* File selection.

\* Multiple-file queue.

\* Drag-and-drop handling.

\* Client-side file-size validation.

\* Expiration selection.

\* Custom transfer message.

\* Recipient verification selection.

\* Transfer creation.

\* TUS upload.

\* Upload progress.

\* Share-link generation.

\* UI animations.

\* SEO metadata rendering.

\* Transfer status presentation.



This means the current frontend is still relatively centralized rather than being divided into a large component/module architecture.



\---



\# 9. Transfer Creation Flow



The current frontend transfer process is conceptually:



```text

User selects files

&#x20;     ↓

Client validates file queue

&#x20;     ↓

User selects transfer options

&#x20;     ↓

Create transfer API request

&#x20;     ↓

Receive secure transfer token

&#x20;     ↓

Upload files sequentially

&#x20;     ↓

TUS upload completion

&#x20;     ↓

Generate recipient link

&#x20;     ↓

Display / share secure link

```



The current implementation creates the transfer before uploading the files.



\---



\# 10. Client-Side File Queue



The frontend supports multiple selected files.



The queue provides:



\* File selection.

\* Drag-and-drop.

\* File listing.

\* File sizes.

\* Removal of queued files.

\* Total-size calculation.

\* Upload progress.



The frontend currently enforces a total client-side limit of:



```text

5 GiB

```



This is a user-experience validation.



It is not the authoritative security boundary.



The backend must continue enforcing upload-size and quota restrictions independently.



\---



\# 11. TUS Upload Integration



The frontend uses the TUS resumable upload protocol through a custom browser implementation.



It does not currently use `tus-js-client`.



The process includes:



```text

POST

&#x20; ↓

Upload-Length

&#x20; ↓

Upload-Metadata

&#x20; ↓

TUS upload location

&#x20; ↓

PATCH chunks

&#x20; ↓

Upload-Offset

&#x20; ↓

Completion

```



The current chunk size is:



```text

4 MiB

```



Uploads are processed sequentially.



\---



\# 12. TUS Endpoint



The frontend constructs the TUS endpoint using:



```text

VITE\_API\_BASE\_URL

```



with a fallback of:



```text

http://localhost:8080

```



The intended endpoint is:



```text

/api/upload-stream

```



Therefore the default development endpoint is conceptually:



```text

http://localhost:8080/api/upload-stream

```



\---



\# 13. Current Upload Authentication Gap



The backend TUS endpoint currently requires authentication.



The current frontend TUS requests do not send the required JWT Authorization header.



This creates an integration gap.



Current situation:



```text

Frontend

&#x20;  │

&#x20;  │ TUS request without JWT

&#x20;  ▼

Backend

&#x20;  │

&#x20;  │ Authentication required

&#x20;  ▼

Request rejected

```



This must eventually be resolved through proper frontend authentication integration.



The correct architectural response is \*\*not\*\* to weaken or remove backend authentication.



\---



\# 14. Transfer Creation API Integration Gap



The current frontend contains a separate hardcoded API URL for transfer creation:



```text

http://localhost:5000/api/Transfer/create

```



However, the backend development environment currently runs on:



```text

http://localhost:8080

```



The TUS flow uses the configurable API base while transfer creation currently uses the hardcoded port 5000.



Therefore the frontend currently has an API-base inconsistency.



Conceptually:



```text

TUS

&#x20; → API\_BASE

&#x20; → default :8080



Transfer creation

&#x20; → hardcoded :5000

```



This is a known integration issue.



\---



\# 15. API Base Configuration



The frontend uses:



```text

VITE\_API\_BASE\_URL

```



for the configurable API base.



The application falls back to:



```text

http://localhost:8080

```



when the environment value is unavailable.



The frontend configuration must eventually use a single consistent API base strategy.



Environment configuration containing credentials or sensitive values must not be exposed or committed improperly.



\---



\# 16. Expiration Configuration



The frontend currently displays these expiration options:



```text

1 day

3 days

7 days

14 days

30 days

```



These correspond to the intended product choices.



The backend currently accepts a numeric expiration value and applies server-side validation.



The backend currently permits values within a 1–30 day range rather than enforcing only the exact five UI choices.



Therefore:



```text

Frontend contract

&#x20;   1 / 3 / 7 / 14 / 30



Backend validation

&#x20;   1–30 days

```



The backend remains authoritative.



\---



\# 17. Recipient Verification UI



The frontend currently provides a user interface option to enable recipient verification.



However, the complete recipient verification mechanism is not implemented end-to-end.



Current state:



```text

Verification checkbox

&#x20;      ↓

Transfer creation request

&#x20;      ↓

Backend stores RequireRecipientVerification

&#x20;      ↓

Complete recipient verification flow

&#x20;      ✗

```



Missing frontend functionality includes:



\* Recipient email entry where required.

\* OTP/code entry.

\* Verification request handling.

\* Verification-code validation.

\* Verification session/state.

\* Download gating based on verification.



Therefore the existing checkbox must not be interpreted as proof that recipient verification is operational.



\---



\# 18. DownloadPage.jsx Responsibilities



`DownloadPage.jsx` handles shared-transfer recipient access.



Current responsibilities include:



\* Reading the `token` route parameter.

\* Requesting transfer details.

\* Displaying loading state.

\* Displaying errors.

\* Showing sender message.

\* Showing available files.

\* Showing file sizes.

\* Showing expiration information.

\* Providing download links.



The current recipient flow is:



```text

Secure token

&#x20;   ↓

GET transfer details

&#x20;   ↓

Display file list

&#x20;   ↓

Direct download

```



\---



\# 19. Current Recipient Download Flow



The current implementation does not yet contain the full planned recipient security flow.



Current:



```text

/t/:token

&#x20;    ↓

GET /api/Transfer/details/{token}

&#x20;    ↓

Display files

&#x20;    ↓

GET /api/Transfer/download/{token}/{fileName}

```



Planned future flow may become:



```text

/t/:token

&#x20;    ↓

Transfer security evaluation

&#x20;    ↓

Recipient email

&#x20;    ↓

OTP verification

&#x20;    ↓

Optional password verification

&#x20;    ↓

Authorized download session

&#x20;    ↓

File download / signed URL

```



The future architecture must preserve transfer isolation.



\---



\# 20. Download URL Design



The current frontend builds download URLs using:



```text

token + encoded original filename

```



Conceptually:



```text

/api/Transfer/download/{token}/{encodedFileName}

```



This works with the current backend design but has limitations.



Potential future architecture:



```text

/api/Transfer/download/{token}/{fileId}

```



or a short-lived signed object-storage URL.



A file ID is preferable to relying on the original filename as the primary file identifier.



\---



\# 21. SEO



The frontend currently includes SEO-related metadata.



`App.jsx` contains metadata for:



```text

Title:

VASTRIX | Secure large-file transfer platform

```



and a product description/keywords.



`index.html` also contains title and metadata.



The current SEO implementation is primarily static/client-side metadata.



It is not currently a server-rendered Next.js SEO architecture.



\---



\# 22. Visual/UI Architecture



The frontend uses:



```text

Tailwind CSS

Framer Motion

Lucide React

```



The current visual direction includes:



\* Dark interface.

\* Violet/purple visual accents.

\* Animated UI.

\* Drag-and-drop upload experience.

\* File queue.

\* Progress indicators.

\* Responsive presentation.

\* Share-link interaction.



The application currently aims for a polished consumer-facing secure-file-transfer experience.



\---



\# 23. Tailwind Configuration



Tailwind styling is integrated through the Vite Tailwind plugin.



The Vite configuration includes:



```text

@vitejs/plugin-react

@tailwindcss/vite

```



The Tailwind configuration scans:



```text

index.html

src/\*\*/\*.{js,ts,jsx,tsx}

```



Custom animations include:



```text

pulse-slow

glow

```



No additional Tailwind plugins are currently required by the verified configuration.



\---



\# 24. CSS Architecture



The main global stylesheet is:



```text

src\\index.css

```



It imports Tailwind and provides global body styling.



The current application also contains:



```text

src\\App.css

```



The layout is primarily handled through Tailwind utility classes.



`App.css` is intentionally minimal.



\---



\# 25. Frontend State Management



The current application primarily uses React component state and hooks.



There is currently no verified dedicated global state-management framework such as:



```text

Redux

Zustand

MobX

```



The application therefore remains relatively lightweight.



As authentication, dashboard, transfer management, and recipient verification grow, state architecture may need to be reconsidered.



\---



\# 26. Authentication UI Status



The backend supports registration and login APIs.



However, the current frontend does not yet provide a complete authentication interface.



Current frontend status:



```text

Login UI        ❌

Registration UI ❌

JWT storage      ❌

Authenticated API client ❌

Protected routes ❌

User dashboard   ❌

```



This is an important integration area for future frontend development.



\---



\# 27. Dashboard Status



A full authenticated User dashboard is not currently implemented.



Planned dashboard capabilities include:



\* User transfer list.

\* Transfer status.

\* Expiration information.

\* File information.

\* Share-link management.

\* Transfer management.

\* Storage quota information.

\* Download activity where supported.



The dashboard should be developed after the authentication foundation is correctly integrated.



\---



\# 28. Admin UI Status



There is currently no complete frontend Admin dashboard.



The backend already contains an Admin health endpoint, but the frontend does not currently provide a corresponding administrative interface.



Future Admin UI should be protected by backend authorization and must not rely solely on hiding UI elements.



\---



\# 29. Current Frontend Feature Matrix



| Feature                         | Status      |

| ------------------------------- | ----------- |

| React application               | Implemented |

| Vite build system               | Implemented |

| React Router                    | Implemented |

| Landing page                    | Implemented |

| File selection                  | Implemented |

| Multiple-file queue             | Implemented |

| Drag and drop                   | Implemented |

| Client 5 GiB limit              | Implemented |

| Expiration UI                   | Implemented |

| Transfer creation UI            | Implemented |

| TUS upload UI                   | Implemented |

| Upload progress                 | Implemented |

| Secure-link generation          | Implemented |

| Download page                   | Implemented |

| Direct download links           | Implemented |

| Recipient verification checkbox | Partial     |

| Recipient OTP UI                | Missing     |

| Recipient OTP workflow          | Missing     |

| Transfer password UI            | Missing     |

| Maximum download UI             | Missing     |

| Login UI                        | Missing     |

| Registration UI                 | Missing     |

| JWT integration                 | Missing     |

| User dashboard                  | Missing     |

| Admin UI                        | Missing     |

| Consistent API base             | Partial     |

| Object-storage integration      | Missing     |



\---



\# 30. Frontend/Backend Contract



The frontend depends on backend APIs for authoritative operations.



The intended separation is:



```text

Frontend

&#x20;   │

&#x20;   ├── UI validation

&#x20;   ├── UX

&#x20;   ├── file selection

&#x20;   ├── progress

&#x20;   └── API requests

&#x20;            │

&#x20;            ▼

Backend

&#x20;   │

&#x20;   ├── authentication

&#x20;   ├── authorization

&#x20;   ├── ownership

&#x20;   ├── quota

&#x20;   ├── expiration

&#x20;   ├── security validation

&#x20;   └── storage operations

```



The frontend must never become the security authority.



\---



\# 31. Frontend Security Principles



The frontend should follow these principles:



\### 31.1 Never Trust Client Validation



Client validation improves UX but does not replace backend validation.



\### 31.2 Do Not Disable Backend Authentication



Frontend authentication failures must be fixed through correct token/session handling.



\### 31.3 Avoid Sensitive Data Exposure



Do not expose:



\* Passwords.

\* Password hashes.

\* SMTP credentials.

\* Database connection strings.

\* Private storage paths.

\* Internal server secrets.



\### 31.4 Use Secure Transfer Tokens Carefully



Transfer URLs provide access to transfer resources and should be treated as sensitive capabilities.



\### 31.5 Do Not Expose Physical Storage



The frontend should never receive:



```text

C:\\VASTRIX\_FileStore

```



or any equivalent internal filesystem path.



\---



\# 32. Current Frontend Integration Architecture



Current development architecture:



```text

┌───────────────────────────────┐

│       VASTRIX Frontend        │

│                               │

│ React + Vite                  │

│ React Router                  │

│ Tailwind                      │

│ Framer Motion                 │

└───────────────┬───────────────┘

&#x20;               │

&#x20;               │ HTTP / TUS

&#x20;               ▼

┌───────────────────────────────┐

│       VASTRIX Backend         │

│                               │

│ ASP.NET Core / .NET 9         │

│                               │

│ :8080 development API         │

└───────────────┬───────────────┘

&#x20;               │

&#x20;      ┌────────┴─────────┐

&#x20;      ▼                  ▼

&#x20;PostgreSQL        Local File Storage

&#x20;                   C:\\VASTRIX\_FileStore

```



\---



\# 33. Future Frontend Architecture



The intended future architecture is broader:



```text

&#x20;                        VASTRIX

&#x20;                           │

&#x20;                   React Frontend

&#x20;                           │

&#x20;            ┌──────────────┼──────────────┐

&#x20;            │              │              │

&#x20;        Auth UI        Transfer UI     Recipient UI

&#x20;            │              │              │

&#x20;            └──────────────┼──────────────┘

&#x20;                           │

&#x20;                        API

&#x20;                           │

&#x20;                   ASP.NET Core

&#x20;                           │

&#x20;            ┌──────────────┼──────────────┐

&#x20;            │              │              │

&#x20;       PostgreSQL       Object Storage   Workers

```



Potential future frontend areas:



```text

/auth/login

/auth/register

/dashboard

/transfer/new

/transfer/:id

/t/:token

/admin

```



These routes are architectural possibilities and are not all currently implemented.



\---



\# 34. Planned Authentication Architecture



Future authenticated frontend flow:



```text

Login

&#x20; ↓

Backend validates credentials

&#x20; ↓

JWT / authentication session

&#x20; ↓

Frontend authenticated state

&#x20; ↓

Authenticated API requests

&#x20; ↓

User-specific resources

```



The exact token storage strategy must be selected deliberately with security considerations.



Do not introduce insecure long-lived browser storage simply to make API calls work.



\---



\# 35. Planned Recipient Architecture



Recipient access should remain independent of sender account authentication.



Conceptually:



```text

Sender

&#x20; │

&#x20; └── creates transfer

&#x20;         │

&#x20;         ▼

&#x20;     Secure token

&#x20;         │

&#x20;         ▼

&#x20;     Recipient

&#x20;         │

&#x20;         ├── verification if required

&#x20;         ├── password if required

&#x20;         └── authorized download

```



A recipient should not automatically become an authenticated sender User.



\---



\# 36. Frontend Development Priorities



The frontend should eventually progress in an order that respects backend dependencies.



Suggested sequence:



```text

1\. Backend API contract stabilization

&#x20;       ↓

2\. Authentication UI and JWT integration

&#x20;       ↓

3\. Authenticated transfer creation

&#x20;       ↓

4\. Correct TUS authentication

&#x20;       ↓

5\. User dashboard

&#x20;       ↓

6\. Recipient verification UI/flow

&#x20;       ↓

7\. Transfer password UI/flow

&#x20;       ↓

8\. Download controls

&#x20;       ↓

9\. Admin UI

&#x20;       ↓

10\. Production UX/SEO refinement

```



The exact development order may change based on backend implementation decisions.



\---



\# 37. Current Known Integration Issues



\## Issue 1 — API Port Mismatch



Transfer creation currently references port `5000`, while the backend is running on `8080`.



Status:



```text

Open

```



\---



\## Issue 2 — Missing JWT on Transfer Creation



The backend requires authenticated access, but the current frontend transfer-creation request does not provide the JWT.



Status:



```text

Open

```



\---



\## Issue 3 — Missing JWT on TUS Upload



The backend TUS authorization requires authentication, while the current frontend TUS requests do not provide the JWT.



Status:



```text

Open

```



\---



\## Issue 4 — Recipient Verification Incomplete



The UI option exists, but the complete recipient verification process is not implemented.



Status:



```text

Open

```



\---



\## Issue 5 — Direct Filename-Based Download



The current download flow uses the original filename in the URL.



A future file-ID-based or signed-URL architecture should be evaluated.



Status:



```text

Future improvement

```



\---



\# 38. What the Frontend Must Not Assume



Future AI assistants must not assume:



\* The application uses Next.js.

\* The application uses TypeScript.

\* Authentication UI already exists.

\* JWT integration already exists.

\* TUS authentication already works from the frontend.

\* Recipient OTP is operational.

\* Password protection is operational.

\* Max-download controls are operational.

\* The dashboard exists.

\* The Admin UI exists.

\* The backend runs on port 5000.

\* Client-side validation is sufficient for security.



These assumptions would conflict with the verified current implementation.



\---



\# 39. Source of Truth



The authoritative frontend source is:



```text

C:\\Akhil\\VASTRIX\\frontend

```



Primary source files currently include:



```text

src\\main.jsx

src\\App.jsx

src\\DownloadPage.jsx

src\\index.css

src\\App.css

vite.config.js

tailwind.config.js

package.json

index.html

```



For detailed implementation behavior, inspect the actual source code.



Project memory should describe the source but must never override it.



\---



\# 40. Architecture Maintenance Rule



When frontend architecture changes materially, update the appropriate project-memory document.



Examples:



```text

New authentication architecture

&#x20;   → FRONTEND\_ARCHITECTURE.md



New storage integration

&#x20;   → STORAGE\_ARCHITECTURE.md



New complete system flow

&#x20;   → SYSTEM\_ARCHITECTURE.md



New implementation status

&#x20;   → 06\_FRONTEND/FRONTEND\_STATUS.md



Important architectural decision

&#x20;   → 08\_HISTORY/ARCHITECTURAL\_DECISIONS.md

```



Avoid copying entire source files into project memory.



The purpose of memory is to preserve:



\* Architecture.

\* Decisions.

\* Current state.

\* Important constraints.

\* Security boundaries.

\* Known gaps.

\* Development context.



\---



\# 41. Final Architecture Summary



The current VASTRIX frontend is a lightweight:



```text

React 19

\+

Vite 8

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



application.



Its current primary flow is:



```text

Landing page

&#x20;   ↓

Select files

&#x20;   ↓

Create transfer

&#x20;   ↓

TUS upload

&#x20;   ↓

Generate secure link

&#x20;   ↓

Recipient opens /t/:token

&#x20;   ↓

View files

&#x20;   ↓

Download

```



The frontend already provides a functional UI foundation, but the complete authenticated product architecture is not yet implemented.



The most important current integration work is to establish a correct authenticated frontend-to-backend contract without weakening the backend's security model.



\---



\# 42. Source Authority



The source-of-truth hierarchy is:



```text

Actual frontend source

&#x20;       ↓

Detailed technical documentation

&#x20;       ↓

PROJECT\_MEMORY

&#x20;       ↓

AI / human understanding

```



If this document conflicts with the actual source code, the actual source code is authoritative until the documentation is corrected.



