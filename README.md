

# 🏢 DMS Hybrid System - Complete Design Documentation ## Document Management System with Personal Drive + Organization Drive + Sharing --- ## 📋 Table of Contents 1. [System Overview](#system-overview) 2. [Core Concepts](#core-concepts) 3. [Architecture Design](#architecture-design) 4. [Databas

pasted

remove all model code like we needd this field and give me the completed documentation some line are missing

14:56
🏢 DMS Hybrid System - Complete Design Documentation
Document Management System with Personal Drive + Organization Drive + Sharing
📋 Table of Contents
System Overview
Core Concepts
Architecture Design
Database Models
Access Control Logic
Folder Sharing System
RBAC for Organization Drive
API Design Strategy
Implementation Roadmap
Security & Privacy Rules
Real-World Examples
Edge Cases & Testing Scenarios
🎯 System Overview {#system-overview}
What We're Building
A Hybrid Document Management System with three main components:

My Drive (Personal) - Private workspace for each user
Organization Drive - Company-wide shared folders with RBAC (5 roles)
Shared with Me - View folders others have shared with you
Key Design Decisions
✅ Personal folders CAN be shared with other users
✅ NO promotion from personal to organization folders
✅ Super Admin CANNOT see personal folders (privacy first)
✅ Two distinct folder types - PERSONAL and ORGANIZATION

System Architecture Diagram
┌─────────────────────────────────────────────────────────────────┐
│                      DMS HYBRID SYSTEM                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────────┐                 ┌──────────────────┐   │
│  │  PERSONAL DRIVE    │                 │ ORGANIZATION     │   │
│  │  (My Drive)        │                 │ DRIVE            │   │
│  ├────────────────────┤                 ├──────────────────┤   │
│  │                    │                 │                  │   │
│  │ • Private by       │                 │ • Shared by      │   │
│  │   default          │                 │   default        │   │
│  │                    │                 │                  │   │
│  │ • Owner = Creator  │                 │ • Access via     │   │
│  │                    │                 │   RBAC           │   │
│  │ • Can share with   │◄────────┐       │                  │   │
│  │   specific users   │         │       │ • 5 Role system  │   │
│  │                    │         │       │                  │   │
│  │ • Super Admin      │         │       │ • Department     │   │
│  │   NO access        │         │       │   based          │   │
│  │                    │         │       │                  │   │
│  └────────────────────┘         │       └──────────────────┘   │
│                                 │                              │
│                    ┌────────────▼──────────┐                   │
│                    │  SHARING SYSTEM       │                   │
│                    ├───────────────────────┤                   │
│                    │                       │                   │
│                    │ • Share personal      │                   │
│                    │   folders             │                   │
│                    │                       │                   │
│                    │ • Grant permissions:  │                   │
│                    │   - VIEWER            │                   │
│                    │   - EDITOR            │                   │
│                    │   - CO_OWNER          │                   │
│                    │                       │                   │
│                    │ • Revoke access       │                   │
│                    │                       │                   │
│                    └───────────────────────┘                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
🧩 Core Concepts {#core-concepts}
Concept 1: Two Types of Folders
Every folder in your system is ONE of these two types:

PERSONAL Folder
Created in "My Drive"
Completely private by default
You are the owner
Only you can see it initially
You can share it with specific people
Cannot be converted to organization folder
Think of it like: Your personal diary - only you can read it unless you show it to someone

ORGANIZATION Folder
Created in "Organization Drive"
Belongs to a department
Access controlled by user roles (RBAC)
Visible to people based on their job role
Cannot be shared like personal folders
Think of it like: Company filing cabinet - access depends on your job position

Concept 2: Access Control Layers
We use a two-layer access control system:

Layer 1: Folder Storage
Stores the folder metadata (name, type, owner, department)
Defines what type of folder it is
Links to parent folder (for hierarchy)
Layer 2: Access Control
Defines WHO can access WHAT folder
Stores permission level (Owner/Co-Owner/Editor/Viewer)
Tracks HOW access was granted (Created/Assigned/Shared)
Handles expiry and revocation
Why two layers?

Flexibility: Easy to add/remove access without touching folder data
Audit: Complete history of who accessed what and when
Performance: Efficient queries for "what folders can user X see?"
Sharing: Simple mechanism to grant/revoke access
Concept 3: Access Sources
Every folder access comes from one of three sources:

CREATED: User created the folder (becomes OWNER)
ASSIGNED_RBAC: Admin/Dept Head assigned folder access in organization
SHARED: Another user shared their personal folder with you
Access Source Flow:

Personal Folder:
  Create → CREATED (Owner) → Share → SHARED (Others)

Organization Folder:
  Create → CREATED (Creator gets owner) → Admin Assigns → ASSIGNED_RBAC
Concept 4: Permission Levels
Four levels of access for any folder:

┌──────────────────────────────────────────────────┐
│ OWNER                                            │
│ ├─ View files & folders                          │
│ ├─ Upload/Edit/Delete files                      │
│ ├─ Create/Delete subfolders                      │
│ ├─ Share folder with others (personal only)      │
│ └─ Manage all permissions                        │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ CO_OWNER                                         │
│ ├─ View files & folders                          │
│ ├─ Upload/Edit/Delete files                      │
│ ├─ Create/Delete subfolders                      │
│ ├─ Share folder with others (personal only)      │
│ └─ Cannot manage owner permissions               │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ EDITOR                                           │
│ ├─ View files & folders                          │
│ ├─ Upload/Edit/Delete files                      │
│ ├─ Create/Delete subfolders                      │
│ └─ Cannot share                                  │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ VIEWER                                           │
│ ├─ View files & folders                          │
│ ├─ Download files                                │
│ └─ Cannot modify anything                        │
└──────────────────────────────────────────────────┘
🏗️ Architecture Design {#architecture-design}
Why This Architecture?
We're using Approach 3: Enterprise Grade with Access Control Table because:

✅ Requirement Met: Personal folder sharing with granular permissions
✅ Scalability: Easy to add new permission levels
✅ Flexibility: Grant/revoke access without touching folder structure
✅ Audit: Complete trail of who accessed what
✅ Performance: Efficient queries with proper indexing

Three-Layer Architecture
┌─────────────────────────────────────────────────────────┐
│                    LAYER 1: STORAGE                     │
│                                                         │
│  Folder Model (type: PERSONAL/ORGANIZATION)            │
│  File Model                                            │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                 LAYER 2: ACCESS CONTROL                 │
│                                                         │
│  FolderAccess Model (who can access what)              │
│  FolderShare Model (sharing metadata)                  │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   LAYER 3: BUSINESS LOGIC               │
│                                                         │
│  • RBAC for Organization folders                       │
│  • Ownership check for Personal folders                │
│  • Share permission validation                         │
└─────────────────────────────────────────────────────────┘
System Flow Diagram
┌──────────────────────────────────────────────────────┐
│                   USER INTERFACE                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐    │
│  │  My Drive  │  │   Org      │  │  Shared    │    │
│  │            │  │   Drive    │  │  with Me   │    │
│  └────────────┘  └────────────┘  └────────────┘    │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│              API LAYER (Express Routes)              │
│  • Authentication Middleware                         │
│  • Authorization Checks                              │
│  • Request Validation                                │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│           BUSINESS LOGIC LAYER (Services)            │
│                                                      │
│  ┌────────────────┐         ┌──────────────────┐   │
│  │ Access Control │         │  Sharing Logic   │   │
│  │    Service     │◄───────►│     Service      │   │
│  └────────────────┘         └──────────────────┘   │
│          ▲                           ▲              │
│          │                           │              │
│  ┌───────┴───────┐         ┌────────┴─────────┐   │
│  │  RBAC Logic   │         │  Folder Service  │   │
│  │   for Org     │         │                  │   │
│  └───────────────┘         └──────────────────┘   │
│                                                      │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│            DATA LAYER (MongoDB Models)               │
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │   Folder    │  │ FolderAccess │  │ FolderShare│ │
│  │   Model     │  │    Model     │  │   Model    │ │
│  └─────────────┘  └──────────────┘  └────────────┘ │
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │    File     │  │     User     │  │ Department │ │
│  │   Model     │  │    Model     │  │   Model    │ │
│  └─────────────┘  └──────────────┘  └────────────┘ │
│                                                      │
└──────────────────────────────────────────────────────┘
Collections Overview
┌──────────────────────────────────────────────────────┐
│                    COLLECTIONS                       │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. folders                                          │
│     ├── Stores all folders (personal + organization)│
│     ├── Type distinguishes personal vs organization │
│     └── Owner tracks creator                         │
│                                                      │
│  2. folderAccess ⭐ KEY TABLE                        │
│     ├── Who has access to which folder              │
│     ├── What level of access (OWNER/EDITOR/VIEWER)  │
│     └── How they got access (CREATED/ASSIGNED/...)  │
│                                                      │
│  3. folderShares                                     │
│     ├── Tracks sharing invitations                  │
│     ├── Share metadata (message, expiry)            │
│     └── Share history                               │
│                                                      │
│  4. files                                            │
│     ├── Stores all uploaded files                   │
│     ├── Links to parent folder                      │
│     └── Uploaded by which user                      │
│                                                      │
│  5. users (existing)                                 │
│     ├── User information                            │
│     ├── Role assignment                             │
│     └── Department assignment                       │
│                                                      │
│  6. departments (existing)                           │
│     ├── Department information                      │
│     └── Department head                             │
│                                                      │
└──────────────────────────────────────────────────────┘
💾 Database Models {#database-models}
Model 1: Folder
Purpose: Store all folders (both personal and organization)

Schema Overview:

Folder metadata (name, description)
Folder type (PERSONAL/ORGANIZATION)
Ownership information
Hierarchy (parent-child relationships)
Department association (for org folders)
Soft delete support
Timestamps
Important Business Rules:

Organization folders MUST have a department
Personal folders MUST NOT have a department
Parent folder must be same type as child folder
Soft delete preserves data for audit/recovery
Model 2: FolderAccess ⭐ MOST IMPORTANT
Purpose: Control who can access which folder with what permissions

Schema Overview:

Folder and user references
Permission level (OWNER/CO_OWNER/EDITOR/VIEWER)
Access source tracking (CREATED/ASSIGNED_RBAC/SHARED)
Grant metadata (who granted, when)
Expiry support
Active status flag
Timestamps
Important Business Rules:

One user can have only ONE active access type per folder
Unique compound index on: folderId + userId
When access expires, isActive becomes false automatically
Access can be revoked by setting isActive to false
Permission Matrix:

┌─────────────┬──────┬────────┬────────┬───────┬─────────┐
│ Access Type │ VIEW │ UPLOAD │ DELETE │ SHARE │ MANAGE  │
├─────────────┼──────┼────────┼────────┼───────┼─────────┤
│ OWNER       │  ✅  │   ✅   │   ✅   │  ✅   │   ✅    │
│ CO_OWNER    │  ✅  │   ✅   │   ✅   │  ✅   │   ❌    │
│ EDITOR      │  ✅  │   ✅   │   ✅   │  ❌   │   ❌    │
│ VIEWER      │  ✅  │   ❌   │   ❌   │  ❌   │   ❌    │
└─────────────┴──────┴────────┴────────┴───────┴─────────┘
Model 3: FolderShare
Purpose: Track personal folder sharing with metadata

Schema Overview:

Folder reference
Sharer and recipient references
Permission level granted
Share message/notes
Active status tracking
Revocation metadata
Timestamps
Important Business Rules:

Can only share PERSONAL folders
Cannot share with yourself
Permission level cannot be OWNER (only original owner keeps OWNER)
When share is revoked, corresponding FolderAccess is also deactivated
Share message is optional but recommended for clarity
Model 4: File
Purpose: Store uploaded files metadata

Schema Overview:

File identification (name, original name)
Parent folder reference
Upload metadata (who uploaded, when)
Storage location (S3 URL or local path)
File properties (size, MIME type, extension)
Soft delete support
Timestamps
Important Business Rules:

Files inherit access permissions from parent folder
Soft delete preserves file metadata
File size tracked in bytes for quota management
Original filename preserved for download purposes
Model 5: User (Reference - Already Exists)
Purpose: Store user information and role assignments

Relevant Fields for DMS:

User identification (name, email)
Role assignment (SUPER_ADMIN/ADMIN/DEPT_HEAD/FOLDER_MANAGER/FOLDER_USER)
Department associations
Account status
Important Business Rules:

DEPT_HEAD: Assigned to ONE department
ADMIN: Can be assigned to MULTIPLE departments
Role determines access to organization folders
isActive flag controls account access
Model 6: Department (Reference - Already Exists)
Purpose: Store department information

Schema Overview:

Department identification (name, code)
Description
Department head reference
Active status
Important Business Rules:

Each department has one head (DEPT_HEAD role)
Department code must be unique
Used for organization folder categorization
isActive controls department visibility
🔐 Access Control Logic {#access-control-logic}
Decision Flow for Access Check
┌─────────────────────────────────────────────────────┐
│  User wants to perform ACTION on FOLDER             │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
          ┌──────────────────┐
          │ What folder type?│
          └────────┬─────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   PERSONAL              ORGANIZATION
        │                     │
        ▼                     ▼
┌───────────────┐    ┌────────────────┐
│ Is user owner?│    │  Apply RBAC    │
└───────┬───────┘    │  (Role Based)  │
        │            └────────┬───────┘
       YES                    │
        │                     │
        ▼                     ▼
   ┌─────────┐          ┌──────────┐
   │  ALLOW  │          │  Check   │
   │  (OWNER)│          │   Role   │
   └─────────┘          └────┬─────┘
        ▲                    │
        │              ┌─────┴─────┐
        │              │           │
        │         SUPER_ADMIN   OTHERS
        │              │           │
        │              ▼           ▼
        │         ┌────────┐  ┌──────────────┐
        │         │ ALLOW  │  │ Check Dept + │
        │         └────────┘  │ Assignment   │
        │                     └──────┬───────┘
        │                            │
┌───────┴─────────┐                  │
│ Check FolderAccess│                │
│ table for share  │                 │
└───────┬─────────┘                  │
        │                            │
   ┌────┴────┐                  ┌────┴────┐
  YES       NO                 YES       NO
   │         │                  │         │
   ▼         ▼                  ▼         ▼
┌──────┐ ┌──────┐          ┌──────┐ ┌──────┐
│ALLOW │ │DENY  │          │ALLOW │ │DENY  │
└──────┘ └──────┘          └──────┘ └──────┘
Access Control Logic Overview
For PERSONAL Folders:

Step 1: Check if user is the owner

If YES → Full access (OWNER permissions)
If NO → Go to Step 2
Step 2: Check FolderAccess table

Look for active access record for this user
Check if access has expired (compare expiresAt with current date)
If found and valid → Grant access based on permission level
If not found → DENY access
Important: Super Admin CANNOT access personal folders (privacy first)

For ORGANIZATION Folders:

Step 1: Check user role

If SUPER_ADMIN:

Full access to all organization folders
Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, MANAGE_USERS
No department restrictions
If ADMIN:

Check if user is assigned to this folder's department
If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
If NO → DENY
If DEPT_HEAD:

Check if this is their assigned department
If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
If NO → DENY
If FOLDER_MANAGER:

Check FolderAccess table for direct assignment
Also check parent folders (inheritance)
If found → Can VIEW, UPLOAD, DELETE, CREATE_SUBFOLDER
If not found → DENY
If FOLDER_USER:

Check FolderAccess table for direct assignment
Also check parent folders (inheritance)
If found → Can VIEW, UPLOAD
If not found → DENY
Role Permissions Matrix
┌───────────────┬──────┬────────┬────────┬──────────────┬────────────┐
│ Role          │ VIEW │ UPLOAD │ DELETE │ CREATE FOLDER│ MANAGE     │
├───────────────┼──────┼────────┼────────┼──────────────┼────────────┤
│ SUPER_ADMIN   │  ✅  │   ✅   │   ✅   │      ✅      │     ✅     │
│ ADMIN         │  ✅  │   ✅   │   ✅   │      ✅      │     ❌     │
│ DEPT_HEAD     │  ✅  │   ✅   │   ✅   │      ✅      │     ❌     │
│ FOLDER_MANAGER│  ✅  │   ✅   │   ✅   │   Subfolder  │     ❌     │
│ FOLDER_USER   │  ✅  │   ✅   │   ❌   │      ❌      │     ❌     │
└───────────────┴──────┴────────┴────────┴──────────────┴────────────┘
🤝 Folder Sharing System {#folder-sharing-system}
How Personal Folder Sharing Works
┌─────────────────────────────────────────────────┐
│            FOLDER SHARING WORKFLOW              │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. User A (Owner) shares "My Docs" with User B│
│     ↓                                           │
│  2. System verifies:                            │
│     • Folder is PERSONAL type                   │
│     • User A has permission to share (OWNER/CO_OWNER)│
│     • User B is not User A (can't share with self)│
│     ↓                                           │
│  3. Create FolderShare record                   │
│     • Store share metadata                      │
│     • Store share message                       │
│     ↓                                           │
│  4. Create FolderAccess record for User B       │
│     • Grant permission level                    │
│     • Set accessSource = SHARED                 │
│     ↓                                           │
│  5. User B can now access "My Docs"            │
│     • Folder appears in "Shared with Me"       │
│     • Access level based on permission given    │
│                                                 │
└─────────────────────────────────────────────────┘
Share Permission Levels
┌─────────────┬──────┬────────┬────────┬───────┬─────────┐
│ Permission  │ VIEW │ UPLOAD │ DELETE │ SHARE │ MANAGE  │
├─────────────┼──────┼────────┼────────┼───────┼─────────┤
│ VIEWER      │  ✅  │   ❌   │   ❌   │  ❌   │   ❌    │
│ EDITOR      │  ✅  │   ✅   │   ✅   │  ❌   │   ❌    │
│ CO_OWNER    │  ✅  │   ✅   │   ✅   │  ✅   │   ❌    │
│ OWNER       │  ✅  │   ✅   │   ✅   │  ✅   │   ✅    │
└─────────────┴──────┴────────┴────────┴───────┴─────────┘
Sharing Rules
✅ CAN Share:

OWNER can share with anyone
CO_OWNER can share with anyone
Can share with multiple users
Can set different permission levels for different users
Can update existing share permissions
❌ CANNOT Share:

EDITOR cannot share
VIEWER cannot share
FOLDER_MANAGER cannot share (they manage assigned folders only)
Cannot share organization folders (only personal folders)
Cannot share with yourself
Sharing Workflow Steps
Step 1: Initiate Share

User clicks "Share" on a personal folder
System shows share dialog with user search
Step 2: Select User and Permission

Search and select user to share with
Choose permission level (VIEWER/EDITOR/CO_OWNER)
Optional: Add share message
Step 3: System Validation

Verify folder is PERSONAL type
Verify current user can share (OWNER or CO_OWNER)
Verify not sharing with self
Check if already shared (update existing or create new)
Step 4: Create Records

Create/Update FolderShare record
Create/Update FolderAccess record
Set accessSource = SHARED
Step 5: Notification

Notify recipient about shared folder
Include share message if provided
Revoking Share
Step 1: Initiate Revoke

Owner/CO_OWNER clicks "Revoke" on shared user
Step 2: System Updates

Update FolderShare: Set isActive = false, add revokedAt, revokedBy
Update FolderAccess: Set isActive = false
Step 3: Access Removed

User can no longer access the folder
Folder removed from their "Shared with Me" view
🏢 RBAC for Organization Drive {#rbac-organization}
Role Hierarchy
                  SUPER_ADMIN
                      │
          ┌───────────┴───────────┐
          │                       │
        ADMIN              DEPT_HEAD
          │                       │
          └───────────┬───────────┘
                      │
              FOLDER_MANAGER
                      │
                 FOLDER_USER
Role Descriptions
SUPER_ADMIN:

Scope: Entire organization
Access: All organization folders across all departments
Permissions: Full control - VIEW, UPLOAD, DELETE, CREATE_FOLDER, MANAGE_USERS
Cannot Access: Personal folders of users
Special Powers: System-wide configuration, user role management
ADMIN:

Scope: Assigned departments
Access: All organization folders in assigned departments
Permissions: VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
Assignment: Can be assigned to multiple departments
Limitations: Cannot access departments they're not assigned to
DEPT_HEAD:

Scope: Single department (their own)
Access: All organization folders in their department
Permissions: VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
Assignment: Assigned to one department only
Limitations: Cannot access other departments
FOLDER_MANAGER:

Scope: Specifically assigned folders
Access: Only folders explicitly assigned by ADMIN/DEPT_HEAD
Permissions: VIEW, UPLOAD, DELETE, CREATE_SUBFOLDER
Inheritance: Access to subfolders automatically
Limitations: Cannot create top-level folders
FOLDER_USER:

Scope: Specifically assigned folders
Access: Only folders explicitly assigned by ADMIN/DEPT_HEAD
Permissions: VIEW, UPLOAD (read and add only)
Inheritance: Access to subfolders automatically
Limitations: Cannot delete or create folders
Department-Based Access
Organization Structure:

Company
├── Marketing Department
│   ├── Campaigns Folder
│   │   ├── Q1-2024 (subfolder)
│   │   └── Q2-2024 (subfolder)
│   └── Analytics Folder
│
├── Engineering Department
│   ├── Projects Folder
│   │   ├── Project-A (subfolder)
│   │   └── Project-B (subfolder)
│   └── Documentation Folder
│
└── HR Department
    ├── Recruitment Folder
    └── Policies Folder

Access Examples:

1. Super Admin → Access to ALL folders in ALL departments

2. Admin (assigned to Marketing + Engineering)
   → Access to Marketing folders
   → Access to Engineering folders
   → NO access to HR folders

3. Marketing Dept Head
   → Access to ALL Marketing folders
   → NO access to Engineering or HR folders

4. Folder Manager (assigned to "Campaigns Folder")
   → Access to Campaigns Folder
   → Access to Q1-2024 and Q2-2024 (inheritance)
   → NO access to Analytics Folder

5. Folder User (assigned to "Project-A")
   → Can VIEW and UPLOAD to Project-A
   → Cannot DELETE or CREATE folders
   → NO access to Project-B
Folder Assignment Flow
┌─────────────────────────────────────────────────┐
│         ADMIN/DEPT_HEAD ASSIGNS FOLDER          │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Select organization folder                  │
│     ↓                                           │
│  2. Choose user to assign                       │
│     ↓                                           │
│  3. Select role for assignment:                 │
│     • FOLDER_MANAGER (full folder control)      │
│     • FOLDER_USER (view + upload only)          │
│     ↓                                           │
│  4. Create FolderAccess record:                 │
│     • folderId = selected folder                │
│     • userId = selected user                    │
Continue

14:59
│     • accessType = FOLDER_MANAGER or EDITOR/VIEWER│
│     • accessSource = ASSIGNED_RBAC              │
│     • grantedBy = current admin/dept_head       │
│     ↓                                           │
│  5. User now has access                         │
│     • Can access assigned folder                │
│     • Inherits access to all subfolders         │
│                                                 │
└─────────────────────────────────────────────────┘


### Access Inheritance in Organization Folders
Parent Folder: "Engineering Department"
├── Child Folder: "Projects"
│   ├── Grandchild: "Project-A"
│   └── Grandchild: "Project-B"

If Folder Manager is assigned to "Projects":
✅ Can access "Projects"
✅ Can access "Project-A" (inherited)
✅ Can access "Project-B" (inherited)
✅ Can create new subfolder under "Projects"
❌ Cannot access "Engineering Department" (parent)

If Folder User is assigned to "Project-A":
✅ Can access "Project-A"
❌ Cannot access "Projects" (parent)
❌ Cannot access "Project-B" (sibling)


---

## 🔌 API Design Strategy {#api-design-strategy}

### API Architecture Overview
┌──────────────────────────────────────────────────┐
│                API STRUCTURE                     │
├──────────────────────────────────────────────────┤
│                                                  │
│  /api/v1/                                        │
│    ├── folders/                                  │
│    │   ├── Personal folder operations            │
│    │   ├── Organization folder operations        │
│    │   └── Common folder operations              │
│    │                                             │
│    ├── shares/                                   │
│    │   ├── Share personal folders                │
│    │   └── Manage shares                         │
│    │                                             │
│    ├── files/                                    │
│    │   ├── Upload files                          │
│    │   ├── Download files                        │
│    │   └── File operations                       │
│    │                                             │
│    ├── access/                                   │
│    │   ├── Grant folder access (RBAC)            │
│    │   └── Revoke folder access                  │
│    │                                             │
│    └── drive/                                    │
│        ├── My Drive view                         │
│        ├── Organization Drive view               │
│        └── Shared with Me view                   │
│                                                  │
└──────────────────────────────────────────────────┘


### Core API Endpoints

#### Folder Management APIs
Personal Folders: POST /api/v1/folders/personal - Create personal folder GET /api/v1/folders/personal/:id - Get personal folder details PUT /api/v1/folders/personal/:id - Update personal folder DELETE /api/v1/folders/personal/:id - Delete personal folder GET /api/v1/folders/personal - List my personal folders

Organization Folders: POST /api/v1/folders/organization - Create org folder (requires dept) GET /api/v1/folders/organization/:id - Get org folder details PUT /api/v1/folders/organization/:id - Update org folder DELETE /api/v1/folders/organization/:id - Delete org folder GET /api/v1/folders/organization - List accessible org folders

Common: GET /api/v1/folders/:id/contents - List folder contents (files + subfolders) POST /api/v1/folders/:id/move - Move folder to different parent GET /api/v1/folders/:id/breadcrumb - Get folder path hierarchy


#### Sharing APIs
POST /api/v1/shares/folder/:id - Share personal folder GET /api/v1/shares/folder/:id - List who folder is shared with PUT /api/v1/shares/:shareId - Update share permissions DELETE /api/v1/shares/:shareId - Revoke share GET /api/v1/shares/received - List folders shared with me GET /api/v1/shares/given - List folders I've shared


#### File Management APIs
POST /api/v1/files/upload - Upload file to folder GET /api/v1/files/:id - Get file details GET /api/v1/files/:id/download - Download file DELETE /api/v1/files/:id - Delete file PUT /api/v1/files/:id - Update file metadata POST /api/v1/files/:id/move - Move file to different folder


#### Access Control APIs (RBAC)
POST /api/v1/access/assign - Assign user to org folder DELETE /api/v1/access/:accessId - Remove user access GET /api/v1/access/folder/:id - List users with access GET /api/v1/access/user/:userId - List folders user has access to PUT /api/v1/access/:accessId - Update access level


#### Drive View APIs
GET    /api/v1/drive/personal                - My Drive view
GET    /api/v1/drive/organization            - Organization Drive view
GET    /api/v1/drive/shared                  - Shared with Me view
GET    /api/v1/drive/recent                  - Recently accessed folders/files
GET    /api/v1/drive/search                  - Search across accessible folders


### API Request/Response Examples

#### Example 1: Create Personal Folder

**Request:**
```http
POST /api/v1/folders/personal
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "My Projects",
  "description": "Personal project documentation",
  "parentId": null
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "folderId": "67abc123def456",
    "name": "My Projects",
    "folderType": "PERSONAL",
    "ownerId": "user123",
    "parentId": null,
    "description": "Personal project documentation",
    "createdAt": "2024-12-24T10:30:00Z"
  }
}
```

#### Example 2: Share Personal Folder

**Request:**
```http
POST /api/v1/shares/folder/67abc123def456
Authorization: Bearer <token>
Content-Type: application/json

{
  "sharedWithUserId": "user456",
  "permissionLevel": "EDITOR",
  "shareMessage": "Please review these project docs"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "shareId": "share789",
    "folderId": "67abc123def456",
    "sharedBy": "user123",
    "sharedWith": "user456",
    "permissionLevel": "EDITOR",
    "shareMessage": "Please review these project docs",
    "createdAt": "2024-12-24T11:00:00Z"
  }
}
```

#### Example 3: Assign Folder Manager (Organization)

**Request:**
```http
POST /api/v1/access/assign
Authorization: Bearer <token>
Content-Type: application/json

{
  "folderId": "org_folder_123",
  "userId": "user789",
  "accessType": "FOLDER_MANAGER"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "accessId": "access456",
    "folderId": "org_folder_123",
    "userId": "user789",
    "accessType": "FOLDER_MANAGER",
    "accessSource": "ASSIGNED_RBAC",
    "grantedBy": "admin123",
    "grantedAt": "2024-12-24T11:30:00Z"
  }
}
```

### Middleware Stack
Request Flow:

Authentication Middleware └─ Verify JWT token └─ Extract user info └─ Check if user is active
Authorization Middleware └─ Check folder access permissions └─ Verify action allowed for user role └─ Apply RBAC rules for org folders
Validation Middleware └─ Validate request body └─ Check required fields └─ Sanitize input
Business Logic └─ Execute controller function └─ Apply business rules └─ Return response
Error Handler └─ Catch and format errors └─ Log errors └─ Return appropriate status codes

---

## 🗺️ Implementation Roadmap {#implementation-roadmap}

### Phase 1: Foundation (Week 1-2)

**Goal:** Set up database models and basic folder structure

**Tasks:**
1. Create database models
   - Folder model
   - FolderAccess model
   - FolderShare model
   - File model

2. Set up indexes
   - Folder: folderType, ownerId, parentId, departmentId
   - FolderAccess: folderId + userId (unique compound)
   - FolderShare: folderId, sharedWith, isActive
   - File: folderId, uploadedBy

3. Write model validation
   - Folder type constraints
   - Department requirements
   - Parent-child type matching

4. Create seed data
   - Test users with different roles
   - Test departments
   - Sample folder structures

**Deliverables:**
- ✅ All models implemented
- ✅ Database indexes created
- ✅ Validation rules working
- ✅ Seed data script ready

---

### Phase 2: Personal Drive (Week 3-4)

**Goal:** Implement My Drive functionality with sharing

**Tasks:**
1. Personal folder CRUD
   - Create personal folder API
   - List user's folders API
   - Update folder API
   - Delete folder API (soft delete)

2. File operations in personal folders
   - Upload file API
   - Download file API
   - List files in folder
   - Delete file API

3. Personal folder sharing
   - Share folder API
   - List shared folders API
   - Update share permissions
   - Revoke share API

4. Access control for personal folders
   - Owner check middleware
   - Share permission check
   - File access through folder permissions

**Deliverables:**
- ✅ Personal folder CRUD working
- ✅ File upload/download working
- ✅ Sharing system functional
- ✅ Access control enforced

---

### Phase 3: Organization Drive - RBAC (Week 5-6)

**Goal:** Implement organization folders with role-based access

**Tasks:**
1. Organization folder CRUD
   - Create org folder with department
   - List accessible org folders by role
   - Update org folder
   - Delete org folder

2. RBAC implementation
   - Super Admin access logic
   - Admin multi-department access
   - Dept Head single department access
   - Folder Manager assignment logic
   - Folder User view-only logic

3. Folder assignment system
   - Assign Folder Manager API
   - Assign Folder User API
   - List assigned users
   - Remove assignments

4. Access inheritance
   - Subfolder access propagation
   - Parent access checking
   - Recursive permission queries

**Deliverables:**
- ✅ Org folder structure working
- ✅ RBAC fully implemented
- ✅ Assignment system functional
- ✅ Inheritance working correctly

---

### Phase 4: Drive Views (Week 7)

**Goal:** Create three main drive views

**Tasks:**
1. My Drive view
   - List personal folders
   - Show folder hierarchy
   - Display folder metadata
   - Search personal folders

2. Organization Drive view
   - List accessible org folders by role
   - Filter by department
   - Show role-based permissions
   - Search org folders

3. Shared with Me view
   - List folders shared with current user
   - Show who shared
   - Display permission level
   - Sort by recent shares

4. Common features
   - Recently accessed
   - Favorites/starred folders
   - Breadcrumb navigation
   - Folder statistics

**Deliverables:**
- ✅ All three views working
- ✅ Navigation smooth
- ✅ Search functional
- ✅ UI responsive

---

### Phase 5: File Management (Week 8)

**Goal:** Complete file operations across all folder types

**Tasks:**
1. File upload enhancements
   - Multiple file upload
   - File type validation
   - Size limit enforcement
   - Upload progress tracking

2. File operations
   - Move file between folders
   - Copy file
   - Rename file
   - Version history (optional)

3. File download
   - Direct download
   - Bulk download (zip)
   - Download tracking
   - Bandwidth optimization

4. File preview
   - Image preview
   - PDF preview
   - Document preview
   - Video/audio preview

**Deliverables:**
- ✅ Complete file management
- ✅ Preview system working
- ✅ Download optimized
- ✅ Move/copy functional

---

### Phase 6: Security & Audit (Week 9)

**Goal:** Implement security measures and audit logging

**Tasks:**
1. Security enhancements
   - Rate limiting on uploads
   - Malware scanning
   - XSS prevention
   - CSRF protection

2. Audit logging
   - Track all folder accesses
   - Log all file operations
   - Record permission changes
   - Monitor share activities

3. Privacy controls
   - Ensure Super Admin can't see personal folders
   - Data encryption at rest
   - Secure file URLs
   - Session management

4. Compliance
   - Data retention policies
   - GDPR compliance
   - Access logs for audits
   - Data export functionality

**Deliverables:**
- ✅ Security measures active
- ✅ Audit logs comprehensive
- ✅ Privacy enforced
- ✅ Compliance ready

---

### Phase 7: Testing & Optimization (Week 10)

**Goal:** Comprehensive testing and performance optimization

**Tasks:**
1. Unit testing
   - Model validation tests
   - Access control logic tests
   - RBAC rule tests
   - Sharing logic tests

2. Integration testing
   - API endpoint tests
   - End-to-end workflows
   - Role-based access scenarios
   - Share permission scenarios

3. Performance optimization
   - Query optimization
   - Index usage analysis
   - Caching strategy
   - Load testing

4. Bug fixes and refinement
   - Fix identified issues
   - Edge case handling
   - Error message improvements
   - UI/UX refinements

**Deliverables:**
- ✅ 90%+ test coverage
- ✅ All critical paths tested
- ✅ Performance benchmarks met
- ✅ Production-ready code

---

## 🔒 Security & Privacy Rules {#security-rules}

### Privacy Principles

**1. Personal Folder Privacy**
RULE: Super Admin CANNOT access personal folders

Why?

Personal folders are PRIVATE by default
Only owner controls access
Sharing is explicit and user-controlled
No backdoor access, even for admins
Exception: Legal/compliance requirements with proper authorization


**2. Organization Folder Access**
RULE: Access strictly by role and department

Why?

Prevents unauthorized data access
Maintains departmental boundaries
Enforces least privilege principle
Clear audit trail

**3. Share Permission Control**
RULE: Only OWNER and CO_OWNER can share

Why?

Prevents unauthorized sharing
Maintains control over private data
Reduces security risks
Clear permission hierarchy

### Security Rules

**1. Authentication Requirements**
ALL endpoints require:

Valid JWT token
Active user account
Session not expired
IP whitelist (optional)

**2. Authorization Checks**
Before ANY folder operation:

Verify user has access
Check permission level
Validate action allowed
Log access attempt

**3. File Upload Security**
Validations:

File size limits (e.g., max 100MB)
Allowed MIME types
Filename sanitization
Virus scanning
Storage quota checks

**4. SQL Injection Prevention**
Use parameterized queries
Validate all inputs
Sanitize user data
Use ORM (Mongoose) properly

**5. XSS Prevention**
Escape HTML in file names
Sanitize folder descriptions
Validate share messages
Content Security Policy headers

### Data Protection

**1. Encryption**
At Rest:

Database encryption enabled
File storage encryption (S3 SSE or local encryption)
Backup encryption
In Transit:

HTTPS only
TLS 1.2+ required
Secure WebSocket connections

**2. Access Logs**
Log ALL:

Folder access attempts
File downloads
Permission changes
Share activities
Failed access attempts
Retention: 90 days minimum


**3. Data Deletion**
Soft Delete:

Mark isDeleted = true
Set deletedAt timestamp
Keep deletedBy user reference
Retain for 30 days
Hard Delete:

After retention period
Permanent file deletion
Remove from backups
Audit log entry

### Compliance Rules

**1. Data Access Rights**
Users can:

Request all their data
Export personal folders
Review access logs
Delete their account

**2. Data Retention**
Personal Folders:

Kept while account active
Deleted 30 days after account deletion
Backups purged after 90 days
Organization Folders:

Company retention policy applies
Minimum 7 years for compliance
Deletion requires admin approval

**3. Audit Requirements**
Track and log:

Who accessed what
When access occurred
What actions were performed
Permission changes
Share activities

---

## 💡 Real-World Examples {#real-examples}

### Example 1: Marketing Team Collaboration

**Scenario:**
Marketing department needs to collaborate on campaign materials. Some assets are personal drafts, others are official department assets.

**Setup:**

**Users:**
- Sarah (Dept Head - Marketing)
- John (Folder Manager)
- Alice (Folder User)
- Bob (Freelancer - external)

**Organization Structure:**
Organization Drive
└── Marketing Department
├── Q4 Campaigns (org folder)
│   ├── Social Media Assets
│   └── Email Templates
└── Brand Guidelines (org folder)


**Personal Work:**
Sarah's My Drive
└── Campaign Drafts (personal folder)
├── Draft-1.psd
└── Draft-2.psd


**Workflow:**

1. **Sarah creates campaign draft in her personal drive**
   - Creates "Campaign Drafts" folder
   - Uploads draft designs
   - Folder is completely private

2. **Sarah shares draft with John for feedback**
   - Shares "Campaign Drafts" with John
   - Grants EDITOR permission
   - John can now view and add comments
   - Folder appears in John's "Shared with Me"

3. **John reviews and approves**
   - Accesses folder from "Shared with Me"
   - Reviews files
   - Uploads revised versions

4. **Sarah shares with Bob (freelancer)**
   - Shares same folder with Bob
   - Grants VIEWER permission only
   - Bob can view but not modify

5. **Final assets moved to organization**
   - Sarah uploads final assets to "Q4 Campaigns"
   - John (Folder Manager) organizes them
   - Alice (Folder User) can view and download
   - Bob cannot access organization folders

**Access Summary:**
Campaign Drafts (Personal):

Sarah: OWNER
John: EDITOR (shared)
Bob: VIEWER (shared)
Alice: NO ACCESS
Super Admin: NO ACCESS (privacy)
Q4 Campaigns (Organization):

Sarah: Full access (Dept Head)
John: Full access (Folder Manager assigned)
Alice: View + Upload (Folder User assigned)
Bob: NO ACCESS (not in organization)
Super Admin: Full access

---

### Example 2: Engineering Documentation

**Scenario:**
Engineering team manages project documentation with strict access control.

**Setup:**

**Users:**
- Mike (Super Admin)
- Lisa (Admin - Engineering + HR)
- Tom (Dept Head - Engineering)
- Emma (Folder Manager - Project A)
- David (Folder User - Project A)

**Organization Structure:**
Organization Drive
├── Engineering Department
│   ├── Project A Documentation
│   │   ├── Requirements
│   │   ├── Design Docs
│   │   └── API Specs
│   ├── Project B Documentation
│   └── Internal Tools
└── HR Department
└── Employee Handbook


**Workflow:**

1. **Tom creates Project A folder structure**
   - As Dept Head, creates "Project A Documentation"
   - Creates subfolders: Requirements, Design Docs, API Specs
   - All inherit Engineering department access

2. **Tom assigns Emma as Folder Manager**
   - Emma gets full control of Project A folder
   - Can manage files and subfolders
   - Access automatically includes all subfolders

3. **Emma assigns David as Folder User**
   - David gets view + upload access
   - Can contribute documents
   - Cannot delete or reorganize

4. **Access scenarios:**

   **Mike (Super Admin):**
   - Can access ALL organization folders
   - Project A: ✅ Full access
   - Project B: ✅ Full access
   - HR Handbook: ✅ Full access

   **Lisa (Admin - Engineering + HR):**
   - Can access assigned departments
   - Project A: ✅ Full access
   - Project B: ✅ Full access
   - HR Handbook: ✅ Full access
   - Other departments: ❌ No access

   **Tom (Dept Head - Engineering):**
   - Can access only Engineering department
   - Project A: ✅ Full access
   - Project B: ✅ Full access
   - HR Handbook: ❌ No access (different dept)

   **Emma (Folder Manager - Project A):**
   - Can access only assigned folders
   - Project A: ✅ Full access (assigned)
   - Requirements: ✅ Full access (inheritance)
   - Design Docs: ✅ Full access (inheritance)
   - Project B: ❌ No access (not assigned)

   **David (Folder User - Project A):**
   - Can access only assigned folders
   - Project A: ✅ View + Upload
   - Requirements: ✅ View + Upload (inheritance)
   - Design Docs: ✅ View + Upload (inheritance)
   - Project B: ❌ No access
   - Cannot delete files: ❌

5. **Emma creates personal backup**
   - Creates personal folder "My Project Notes"
   - Uploads personal copies for reference
   - Completely private, even Tom cannot see
   - Emma can share with David if needed

---

### Example 3: HR Confidential Documents

**Scenario:**
HR needs to manage sensitive employee documents with strict confidentiality.

**Setup:**

**Users:**
- Rachel (Dept Head - HR)
- Kevin (Folder Manager - Recruitment)
- Nina (Folder User - Payroll)

**Organization Structure:**
Organization Drive
└── HR Department
├── Recruitment
│   ├── Candidate Profiles
│   └── Interview Notes
├── Payroll
│   └── Salary Data
└── Policies


**Workflow:**

1. **Rachel creates HR folder structure**
   - Creates department folders
   - Sets up sensitive areas: Recruitment, Payroll

2. **Kevin assigned to Recruitment only**
   - Gets Folder Manager access to Recruitment
   - Can manage candidate files
   - CANNOT access Payroll (different folder)

3. **Nina assigned to Payroll only**
   - Gets Folder User access to Payroll
   - Can view and upload salary documents
   - CANNOT access Recruitment (different folder)

4. **Access isolation:**
Recruitment folder:

Rachel: ✅ Full access (Dept Head)
Kevin: ✅ Full access (Folder Manager)
Nina: ❌ No access (not assigned)
Payroll folder:

Rachel: ✅ Full access (Dept Head)
Kevin: ❌ No access (not assigned)
Nina: ✅ View + Upload (Folder User)

5. **Rachel's personal review notes**
   - Creates personal folder "Employee Reviews 2024"
   - Writes confidential review notes
   - Even Super Admin cannot access
   - Can selectively share specific reviews with Kevin

---

### Example 4: Cross-Department Project

**Scenario:**
Marketing and Engineering collaborate on product launch.

**Setup:**

**Users:**
- Sarah (Dept Head - Marketing)
- Tom (Dept Head - Engineering)
- Lisa (Admin - Marketing + Engineering)
- John (Folder Manager - Marketing)
- Emma (Folder Manager - Engineering)

**Approach 1: Organization Folder in One Department**
Organization Drive
└── Marketing Department
└── Product Launch 2024
├── Marketing Materials
└── Technical Specs


**Problem:** Tom (Engineering Dept Head) cannot access Marketing folders

**Solution:** Lisa (Admin assigned to both departments) can access
- Lisa creates folder in Marketing
- Lisa assigns Emma (Engineering) as Folder Manager
- Now Emma can contribute technical specs
- Tom still cannot access (not his department)

---

**Approach 2: Personal Folder with Sharing**
Sarah's My Drive
└── Product Launch Collaboration
├── Joint Planning
└── Shared Resources


**Solution:**
- Sarah creates personal folder
- Shares with Tom (CO_OWNER)
- Shares with John (EDITOR)
- Shares with Emma (EDITOR)
- Everyone can collaborate
- When finalized, upload to respective departments

**Access:**
Product Launch Collaboration (Personal):

Sarah: OWNER
Tom: CO_OWNER (can share further)
John: EDITOR (can modify)
Emma: EDITOR (can modify)

**Benefits:**
- Cross-department collaboration
- Flexible access control
- Easy to add/remove collaborators
- Final assets moved to official locations

---

## 🧪 Edge Cases & Testing Scenarios {#edge-cases}

### Edge Case 1: Nested Folder Access

**Scenario:** User has access to parent folder but child folder is explicitly denied

**Setup:**
Parent Folder (Engineering)
└── Child Folder (Confidential)


**Test Cases:**

1. **User assigned to Parent:**
   - Should access parent: ✅
   - Should access child (inheritance): ✅

2. **User access revoked from Child:**
   - Should access parent: ✅
   - Should access child: ❌ (explicit deny)

3. **User reassigned to Child directly:**
   - Should access parent: ❌ (not assigned)
   - Should access child: ✅ (direct assignment)

**Expected Behavior:**
- Inheritance flows downward (parent → child)
- Explicit assignment overrides inheritance
- No upward inheritance (child → parent)

---

### Edge Case 2: Share Chain

**Scenario:** User A shares with User B, User B wants to share with User C

**Setup:**
User A (Owner) → User B (CO_OWNER) → User C (?)


**Test Cases:**

1. **B tries to share with C (B is CO_OWNER):**
   - Should succeed: ✅
   - C gets access with permission set by B

2. **B tries to share with C (B is EDITOR):**
   - Should fail: ❌
   - Only OWNER and CO_OWNER can share

3. **C tries to share with D:**
   - Depends on C's permission level
   - If CO_OWNER: ✅ Can share
   - If EDITOR/VIEWER: ❌ Cannot share

**Expected Behavior:**
- Share chains are allowed for CO_OWNERs
- Permission level controls sharing ability
- Original owner always maintains control

---

### Edge Case 3: Department Transfer

**Scenario:** Dept Head moves to different department

**Setup:**
Initial:

Tom is Dept Head of Engineering
Tom has access to all Engineering folders
Change:

Tom promoted to Dept Head of Marketing

**Test Cases:**

1. **Before department change:**
   - Tom accesses Engineering folders: ✅

2. **After department change:**
   - Tom accesses Engineering folders: ❌ (wrong dept)
   - Tom accesses Marketing folders: ✅ (new dept)

3. **Explicit folder assignments:**
   - If Tom was explicitly assigned as Folder Manager to Engineering folder
   - Assignment should remain: ✅
   - Access source: ASSIGNED_RBAC (not automatic dept access)

**Expected Behavior:**
- Role-based access follows department assignment
- Explicit folder assignments remain unless revoked
- Department transfer doesn't remove explicit assignments

---

### Edge Case 4: Circular Folder Structure

**Scenario:** Prevent folder from being its own parent or ancestor

**Test Cases:**

1. **Direct circular reference:**
Folder A
└── Move to parent: Folder A

   - Should fail: ❌
   - Error: "Folder cannot be its own parent"

2. **Indirect circular reference:**
Folder A
└── Folder B
└── Folder C
└── Move Folder A here

   - Should fail: ❌
   - Error: "Circular reference detected"

3. **Valid move:**
Folder A → Folder D (unrelated)

   - Should succeed: ✅

**Expected Behavior:**
- Validate parent chain before move
- Prevent any circular references
- Check all ancestors recursively

---

### Edge Case 5: Simultaneous Share and Revoke

**Scenario:** Owner shares folder, then immediately revokes while recipient is accessing

**Test Cases:**

1. **Share created, recipient hasn't accessed yet:**
   - Share: ✅ Created
   - Revoke: ✅ Access removed
   - Recipient tries to access: ❌ No access

2. **Recipient actively using shared folder:**
   - Recipient viewing files
   - Owner revokes share
   - Current session: ✅ Can continue (cached)
   - Next request: ❌ Access denied
   - Folder removed from "Shared with Me"

3. **Share updated instead of revoked:**
   - Owner changes EDITOR → VIEWER
   - Recipient trying to upload: ❌ Permission denied
   - Recipient viewing: ✅ Still works

**Expected Behavior:**
- Revocation takes effect on next request
- Current session may complete
- Clear error messages to user
- Remove from shared views immediately

---

### Edge Case 6: File Upload During Permission Change

**Scenario:** User uploading file when their permission changes

**Test Cases:**

1. **Upload started with EDITOR permission:**
   - Permission changed to VIEWER mid-upload
   - Upload completes: ❌ Fail at finalization
   - Error: "Permission changed during upload"

2. **Upload started with VIEWER permission:**
   - Should fail immediately: ❌
   - Error: "Insufficient permissions"

3. **Folder deleted during upload:**
   - Upload should fail: ❌
   - Error: "Folder no longer exists"

**Expected Behavior:**
- Check permissions at upload start
- Verify permissions at upload completion
- Handle gracefully with clear errors
- Don't leave partial uploads

---

### Edge Case 7: Soft Delete and Undelete

**Scenario:** Folder deleted, then user tries to access or undelete

**Test Cases:**

1. **Owner deletes folder:**
   - Folder marked: isDeleted = true
   - Owner can see in trash: ✅
   - Others cannot see: ❌

2. **Shared users access deleted folder:**
   - All shared users lose access
   - Removed from "Shared with Me"
   - Error if direct access: "Folder not found"

3. **Owner restores folder:**
   - Set isDeleted = false
   - All shares reactivate: ✅
   - Shared users can access again

4. **Permanent delete (30 days passed):**
   - Hard delete folder
   - Delete all files
   - Remove all shares
   - Remove all access records
   - Cannot undelete: ❌

**Expected Behavior:**
- Soft delete preserves structure
- Shares preserved during soft delete
- Hard delete is
now create a documenattion readme.md so i can downlaod it correclt give me readme.md format so i can downlaod it directly

15:03
permanent

Audit log retained even after hard delete
Edge Case 8: Role Demotion
Scenario: Admin demoted to Folder User

Setup:

Initial:
- User is ADMIN
- Has access to multiple departments

Change:
- User demoted to FOLDER_USER
Test Cases:

Before demotion:
Access all folders in assigned departments: ✅
After demotion:
Access department folders: ❌ (role changed)
Only assigned folders accessible: ✅
Explicit assignments:
If explicitly assigned as Folder Manager before demotion
Assignment remains: ✅ (not revoked)
Try to assign others:
Should fail: ❌
Error: "Insufficient permissions"
Expected Behavior:

Role change revokes automatic access
Explicit assignments remain
Cannot perform actions above new role level
Audit log tracks demotion
Testing Checklist
Unit Tests:

 Folder model validation
 Access control logic
 RBAC permission checking
 Share permission validation
 File access through folder permissions
 Inheritance logic
 Soft delete functionality
Integration Tests:

 Create personal folder and share
 Create org folder and assign users
 Upload file and check access
 Revoke share and verify access removed
 Department-based access control
 Role-based access scenarios
 Move folder and verify permissions
End-to-End Tests:

 Complete sharing workflow
 Complete folder assignment workflow
 File upload and download flow
 Search across all drives
 Recently accessed functionality
 Breadcrumb navigation
 Bulk operations
Performance Tests:

 Large folder hierarchy (1000+ folders)
 Many shares (100+ users)
 Concurrent uploads
 Search performance
 Query optimization
 Index usage
Security Tests:

 Unauthorized access attempts
 SQL injection attempts
 XSS prevention
 CSRF protection
 File upload malware scanning
 Rate limiting
 Session hijacking prevention
📝 Summary
This DMS Hybrid System provides:

✅ Personal Drive - Private workspace with selective sharing
✅ Organization Drive - Department-based with 5-role RBAC
✅ Flexible Sharing - Granular permissions (Owner/Co-Owner/Editor/Viewer)
✅ Strong Privacy - Super Admin cannot access personal folders
✅ Clear Access Control - Two-layer system (storage + access)
✅ Audit Trail - Complete tracking of all actions
✅ Scalable Architecture - Supports growth and new features

Key Principles:

Privacy first for personal folders
Role-based access for organization folders
Explicit permissions over implicit
Audit everything
Fail secure
End of Documentation



