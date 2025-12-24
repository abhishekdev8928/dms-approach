# 🏢 DMS Hybrid System - Complete Design Documentation

## Document Management System with Personal Drive + Organization Drive + Sharing

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Core Concepts](#core-concepts)
3. [Architecture Design](#architecture-design)
4. [Database Models](#database-models)
5. [Access Control Logic](#access-control-logic)
6. [Folder Sharing System](#folder-sharing-system)
7. [RBAC for Organization Drive](#rbac-organization)
8. [API Design Strategy](#api-design-strategy)
9. [Implementation Roadmap](#implementation-roadmap)
10. [Security & Privacy Rules](#security-rules)
11. [Real-World Examples](#real-examples)
12. [Edge Cases & Testing Scenarios](#edge-cases)

---

## 🎯 System Overview {#system-overview}

### What We're Building

A Hybrid Document Management System with three main components:

1. **My Drive (Personal)** - Private workspace for each user
2. **Organization Drive** - Company-wide shared folders with RBAC (5 roles)
3. **Shared with Me** - View folders others have shared with you

### Key Design Decisions

✅ Personal folders CAN be shared with other users  
✅ Organization folders CAN ALSO be shared if needed (special feature)  
✅ NO promotion from personal to organization folders  
✅ Super Admin CANNOT see personal folders (privacy first)  
✅ Two distinct folder types - PERSONAL and ORGANIZATION

### System Architecture Diagram

```
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
│  │                    │         │       │ • Can also be    │   │
│  │                    │         │       │   shared         │   │
│  │                    │         │       │   (special)      │   │
│  └────────────────────┘         │       └──────────────────┘   │
│                                 │                              │
│                    ┌────────────▼──────────┐                   │
│                    │  SHARING SYSTEM       │                   │
│                    ├───────────────────────┤                   │
│                    │                       │                   │
│                    │ • Share personal      │                   │
│                    │   folders             │                   │
│                    │                       │                   │
│                    │ • Share organization  │                   │
│                    │   folders (special)   │                   │
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
```

---

## 🧩 Core Concepts {#core-concepts}

### Concept 1: Two Types of Folders

Every folder in your system is ONE of these two types:

#### PERSONAL Folder

- Created in "My Drive"
- Completely private by default
- You are the owner
- Only you can see it initially
- You can share it with specific people
- Cannot be converted to organization folder

**Think of it like:** Your personal diary - only you can read it unless you show it to someone

#### ORGANIZATION Folder

- Created in "Organization Drive"
- Belongs to a department
- Access controlled by user roles (RBAC)
- Visible to people based on their job role
- **Can ALSO be shared with specific users (special feature)**

**Think of it like:** Company filing cabinet - access depends on your job position, but can also be selectively shared

---

### Concept 2: Access Control Layers

We use a two-layer access control system:

#### Layer 1: Folder Storage

- Stores the folder metadata (name, type, owner, department)
- Defines what type of folder it is
- Links to parent folder (for hierarchy)

#### Layer 2: Access Control

- Defines WHO can access WHAT folder
- Stores permission level (Owner/Co-Owner/Editor/Viewer)
- Tracks HOW access was granted (Created/Assigned/Shared)
- Handles expiry and revocation

**Why two layers?**

- **Flexibility:** Easy to add/remove access without touching folder data
- **Audit:** Complete history of who accessed what and when
- **Performance:** Efficient queries for "what folders can user X see?"
- **Sharing:** Simple mechanism to grant/revoke access

---

### Concept 3: Access Sources

Every folder access comes from one of three sources:

1. **CREATED:** User created the folder (becomes OWNER)
2. **ASSIGNED_RBAC:** Admin/Dept Head assigned folder access in organization
3. **SHARED:** Another user shared their folder with you (personal or organization)

#### Access Source Flow:

```
Personal Folder:
  Create → CREATED (Owner) → Share → SHARED (Others)

Organization Folder:
  Create → CREATED (Creator gets owner) → Admin Assigns → ASSIGNED_RBAC
  OR
  Create → CREATED → Owner Shares → SHARED (special case)
```

---

### Concept 4: Permission Levels

Four levels of access for any folder:

```
┌──────────────────────────────────────────────────┐
│ OWNER                                            │
│ ├─ View files & folders                          │
│ ├─ Upload/Edit/Delete files                      │
│ ├─ Create/Delete subfolders                      │
│ ├─ Share folder with others                      │
│ └─ Manage all permissions                        │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ CO_OWNER                                         │
│ ├─ View files & folders                          │
│ ├─ Upload/Edit/Delete files                      │
│ ├─ Create/Delete subfolders                      │
│ ├─ Share folder with others                      │
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
```

---

## 🏗️ Architecture Design {#architecture-design}

### Why This Architecture?

We're using **Approach 3: Enterprise Grade with Access Control Table** because:

✅ **Requirement Met:** Both personal and organization folder sharing with granular permissions  
✅ **Scalability:** Easy to add new permission levels  
✅ **Flexibility:** Grant/revoke access without touching folder structure  
✅ **Audit:** Complete trail of who accessed what  
✅ **Performance:** Efficient queries with proper indexing

---

### Three-Layer Architecture

```
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
│  • Special: Organization folder sharing                │
└─────────────────────────────────────────────────────────┘
```

---

### System Flow Diagram

```
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
```

---

### Collections Overview

```
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
```

---

## 💾 Database Models {#database-models}

### Model 1: Folder

**Purpose:** Store all folders (both personal and organization)

**Important Business Rules:**

- Organization folders MUST have a department
- Personal folders MUST NOT have a department
- Parent folder must be same type as child folder
- Soft delete preserves data for audit/recovery

---

### Model 2: FolderAccess ⭐ MOST IMPORTANT

**Purpose:** Control who can access which folder with what permissions

**Important Business Rules:**

- One user can have only ONE active access type per folder
- Unique compound index on: `folderId + userId`
- When access expires, `isActive` becomes false automatically
- Access can be revoked by setting `isActive` to false

**Permission Matrix:**

```
┌─────────────┬──────┬────────┬────────┬───────┬─────────┐
│ Access Type │ VIEW │ UPLOAD │ DELETE │ SHARE │ MANAGE  │
├─────────────┼──────┼────────┼────────┼───────┼─────────┤
│ OWNER       │  ✅  │   ✅   │   ✅   │  ✅   │   ✅    │
│ CO_OWNER    │  ✅  │   ✅   │   ✅   │  ✅   │   ❌    │
│ EDITOR      │  ✅  │   ✅   │   ✅   │  ❌   │   ❌    │
│ VIEWER      │  ✅  │   ❌   │   ❌   │  ❌   │   ❌    │
└─────────────┴──────┴────────┴────────┴───────┴─────────┘
```

---

### Model 3: FolderShare

**Purpose:** Track folder sharing with metadata (for both personal and organization folders)

**Important Business Rules:**

- Can share PERSONAL folders
- Can also share ORGANIZATION folders (special feature)
- Cannot share with yourself
- Permission level cannot be OWNER (only original owner keeps OWNER)
- When share is revoked, corresponding FolderAccess is also deactivated
- Share message is optional but recommended for clarity

---

### Model 4: File

**Purpose:** Store uploaded files metadata

**Important Business Rules:**

- Files inherit access permissions from parent folder
- Soft delete preserves file metadata
- File size tracked in bytes for quota management
- Original filename preserved for download purposes

---

### Model 5: User (Reference - Already Exists)

**Purpose:** Store user information and role assignments

**Important Business Rules:**

- DEPT_HEAD: Assigned to ONE department
- ADMIN: Can be assigned to MULTIPLE departments
- Role determines access to organization folders
- `isActive` flag controls account access

---

### Model 6: Department (Reference - Already Exists)

**Purpose:** Store department information

**Important Business Rules:**

- Each department has one head (DEPT_HEAD role)
- Department code must be unique
- Used for organization folder categorization
- `isActive` controls department visibility

---

## 🔐 Access Control Logic {#access-control-logic}

### Decision Flow for Access Check

```
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
```

---

### Access Control Logic Overview

#### For PERSONAL Folders:

**Step 1:** Check if user is the owner
- If YES → Full access (OWNER permissions)
- If NO → Go to Step 2

**Step 2:** Check FolderAccess table
- Look for active access record for this user
- Check if access has expired (compare `expiresAt` with current date)
- If found and valid → Grant access based on permission level
- If not found → DENY access

**Important:** Super Admin CANNOT access personal folders (privacy first)

---

#### For ORGANIZATION Folders:

**Step 1:** Check user role

- **If SUPER_ADMIN:**
  - Full access to all organization folders
  - Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, MANAGE_USERS
  - No department restrictions

- **If ADMIN:**
  - Check if user is assigned to this folder's department
  - If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
  - If NO → Check FolderAccess for shared access

- **If DEPT_HEAD:**
  - Check if this is their assigned department
  - If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
  - If NO → Check FolderAccess for shared access

- **If FOLDER_MANAGER:**
  - Check FolderAccess table for direct assignment
  - Also check parent folders (inheritance)
  - If found → Can VIEW, UPLOAD, DELETE, CREATE_SUBFOLDER

- **If FOLDER_USER:**
  - Check FolderAccess table for direct assignment
  - Also check parent folders (inheritance)
  - If found → Can VIEW, UPLOAD

**Step 2:** Check for shared access (special case)
- Organization folders can also be shared
- Check FolderAccess table for SHARED access source
- Grant permissions based on share level

---

### Role Permissions Matrix

```
┌───────────────┬──────┬────────┬────────┬──────────────┬────────────┐
│ Role          │ VIEW │ UPLOAD │ DELETE │ CREATE FOLDER│ MANAGE     │
├───────────────┼──────┼────────┼────────┼──────────────┼────────────┤
│ SUPER_ADMIN   │  ✅  │   ✅   │   ✅   │      ✅      │     ✅     │
│ ADMIN         │  ✅  │   ✅   │   ✅   │      ✅      │     ❌     │
│ DEPT_HEAD     │  ✅  │   ✅   │   ✅   │      ✅      │     ❌     │
│ FOLDER_MANAGER│  ✅  │   ✅   │   ✅   │   Subfolder  │     ❌     │
│ FOLDER_USER   │  ✅  │   ✅   │   ❌   │      ❌      │     ❌     │
└───────────────┴──────┴────────┴────────┴──────────────┴────────────┘
```

---

## 🤝 Folder Sharing System {#folder-sharing-system}

### How Folder Sharing Works

```
┌─────────────────────────────────────────────────┐
│            FOLDER SHARING WORKFLOW              │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. User A (Owner) shares folder with User B   │
│     ↓                                           │
│  2. System verifies:                            │
│     • User A has permission to share            │
│       (OWNER/CO_OWNER)                          │
│     • User B is not User A (can't share with    │
│       self)                                     │
│     • For ORG folders: additional checks        │
│     ↓                                           │
│  3. Create FolderShare record                   │
│     • Store share metadata                      │
│     • Store share message                       │
│     ↓                                           │
│  4. Create FolderAccess record for User B       │
│     • Grant permission level                    │
│     • Set accessSource = SHARED                 │
│     ↓                                           │
│  5. User B can now access folder                │
│     • Folder appears in "Shared with Me"       │
│     • Access level based on permission given    │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

### Share Permission Levels

```
┌─────────────┬──────┬────────┬────────┬───────┬─────────┐
│ Permission  │ VIEW │ UPLOAD │ DELETE │ SHARE │ MANAGE  │
├─────────────┼──────┼────────┼────────┼───────┼─────────┤
│ VIEWER      │  ✅  │   ❌   │   ❌   │  ❌   │   ❌    │
│ EDITOR      │  ✅  │   ✅   │   ✅   │  ❌   │   ❌    │
│ CO_OWNER    │  ✅  │   ✅   │   ✅   │  ✅   │   ❌    │
│ OWNER       │  ✅  │   ✅   │   ✅   │  ✅   │   ✅    │
└─────────────┴──────┴────────┴────────┴───────┴─────────┘
```

---

### Sharing Rules

#### ✅ CAN Share:

- OWNER can share with anyone
- CO_OWNER can share with anyone
- Can share PERSONAL folders
- **Can share ORGANIZATION folders (special feature)**
- Can share with multiple users
- Can set different permission levels for different users
- Can update existing share permissions

#### ❌ CANNOT Share:

- EDITOR cannot share
- VIEWER cannot share
- FOLDER_MANAGER cannot share organization folders (unless they're OWNER/CO_OWNER)
- Cannot share with yourself

---

### Sharing Workflow Steps

**Step 1:** Initiate Share
- User clicks "Share" on a folder
- System shows share dialog with user search

**Step 2:** Select User and Permission
- Search and select user to share with
- Choose permission level (VIEWER/EDITOR/CO_OWNER)
- Optional: Add share message

**Step 3:** System Validation
- Verify current user can share (OWNER or CO_OWNER)
- Verify not sharing with self
- Check if already shared (update existing or create new)
- For organization folders: verify department-level permissions

**Step 4:** Create Records
- Create/Update FolderShare record
- Create/Update FolderAccess record
- Set `accessSource = SHARED`

**Step 5:** Notification
- Notify recipient about shared folder
- Include share message if provided

---

### Revoking Share

**Step 1:** Initiate Revoke
- Owner/CO_OWNER clicks "Revoke" on shared user

**Step 2:** System Updates
- Update FolderShare: Set `isActive = false`, add `revokedAt`, `revokedBy`
- Update FolderAccess: Set `isActive = false`

**Step 3:** Access Removed
- User can no longer access the folder
- Folder removed from their "Shared with Me" view

---

## 🏢 RBAC for Organization Drive {#rbac-organization}

### Role Hierarchy

```
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
```

---

### Role Descriptions

#### SUPER_ADMIN:
- **Scope:** Entire organization
- **Access:** All organization folders across all departments
- **Permissions:** Full control - VIEW, UPLOAD, DELETE, CREATE_FOLDER, MANAGE_USERS
- **Cannot Access:** Personal folders of users
- **Special Powers:** System-wide configuration, user role management

#### ADMIN:
- **Scope:** Assigned departments
- **Access:** All organization folders in assigned departments
- **Permissions:** VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
- **Assignment:** Can be assigned to multiple departments
- **Limitations:** Cannot access departments they're not assigned to

#### DEPT_HEAD:
- **Scope:** Single department (their own)
- **Access:** All organization folders in their department
- **Permissions:** VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
- **Assignment:** Assigned to one department only
- **Limitations:** Cannot access other departments

#### FOLDER_MANAGER:
- **Scope:** Specifically assigned folders
- **Access:** Only folders explicitly assigned by ADMIN/DEPT_HEAD
- **Permissions:** VIEW, UPLOAD, DELETE, CREATE_SUBFOLDER
- **Inheritance:** Access to subfolders automatically
- **Limitations:** Cannot create top-level folders

#### FOLDER_USER:
- **Scope:** Specifically assigned folders
- **Access:** Only folders explicitly assigned by ADMIN/DEPT_HEAD
- **Permissions:** VIEW, UPLOAD (read and add only)
- **Inheritance:** Access to subfolders automatically
- **Limitations:** Cannot delete or create folders

---

### Department-Based Access

**Organization Structure:**

```
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
```

**Access Examples:**

1. **Super Admin** → Access to ALL folders in ALL departments

2. **Admin (assigned to Marketing + Engineering)**
   - → Access to Marketing folders
   - → Access to Engineering folders
   - → NO access to HR folders

3. **Marketing Dept Head**
   - → Access to ALL Marketing folders
   - → NO access to Engineering or HR folders

4. **Folder Manager (assigned to "Campaigns Folder")**
   - → Access to Campaigns Folder
   - → Access to Q1-2024 and Q2-2024 (inheritance)
   - → NO access to Analytics Folder

5. **Folder User (assigned to "Project-A")**
   - → Can VIEW and UPLOAD to Project-A
   - → Cannot DELETE or CREATE folders
   - → NO access to Project-B

---

### Folder Assignment Flow

```
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
│     • accessType = FOLDER_MANAGER or EDITOR     │
│     • accessSource = ASSIGNED_RBAC              │
│     • grantedBy = current admin/dept_head       │
│     ↓                                           │
│  5. User now has access                         │
│     • Can access assigned folder                │
│     • Inherits access to all subfolders         │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

### Access Inheritance in Organization Folders

```
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
```
