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
8. [Real-World Examples](#real-examples)
9. [Edge Cases & Testing Scenarios](#edge-cases)

---

## 🎯 System Overview {#system-overview}

### What We're Building

A Hybrid Document Management System with three main components:

1. **My Drive (Personal)** - Private workspace for each user
2. **Organization Drive** - Company-wide shared folders with RBAC (5 roles)
3. **Shared with Me** - View folders others have shared with you

### Key Design Decisions

- ✅ Personal folders CAN be shared with other users
- ✅ NO promotion from personal to organization folders
- ✅ Super Admin CANNOT see personal folders (privacy first)
- ✅ Two distinct folder types - PERSONAL and ORGANIZATION

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
- Cannot be shared like personal folders

**Think of it like:** Company filing cabinet - access depends on your job position

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
3. **SHARED:** Another user shared their personal folder with you

**Access Source Flow:**

```
Personal Folder:
  Create → CREATED (Owner) → Share → SHARED (Others)

Organization Folder:
  Create → CREATED (Creator gets owner) → Admin Assigns → ASSIGNED_RBAC
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
```

---

## 🏗️ Architecture Design {#architecture-design}

### Why This Architecture?

We're using **Enterprise Grade with Access Control Table** because:

- ✅ **Requirement Met:** Personal folder sharing with granular permissions
- ✅ **Scalability:** Easy to add new permission levels
- ✅ **Flexibility:** Grant/revoke access without touching folder structure
- ✅ **Audit:** Complete trail of who accessed what
- ✅ **Performance:** Efficient queries with proper indexing

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
└─────────────────────────────────────────────────────────┘
```

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

**Schema Overview:**
- Folder metadata (name, description)
- Folder type (PERSONAL/ORGANIZATION)
- Ownership information
- Hierarchy (parent-child relationships)
- Department association (for org folders)
- Soft delete support
- Timestamps

**Important Business Rules:**
- Organization folders MUST have a department
- Personal folders MUST NOT have a department
- Parent folder must be same type as child folder
- Soft delete preserves data for audit/recovery

---

### Model 2: FolderAccess ⭐ MOST IMPORTANT

**Purpose:** Control who can access which folder with what permissions

**Schema Overview:**
- Folder and user references
- Permission level (OWNER/CO_OWNER/EDITOR/VIEWER)
- Access source tracking (CREATED/ASSIGNED_RBAC/SHARED)
- Grant metadata (who granted, when)
- Expiry support
- Active status flag
- Timestamps

**Important Business Rules:**
- One user can have only ONE active access type per folder
- Unique compound index on: folderId + userId
- When access expires, isActive becomes false automatically
- Access can be revoked by setting isActive to false

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

**Purpose:** Track personal folder sharing with metadata

**Schema Overview:**
- Folder reference
- Sharer and recipient references
- Permission level granted
- Share message/notes
- Active status tracking
- Revocation metadata
- Timestamps

**Important Business Rules:**
- Can only share PERSONAL folders
- Cannot share with yourself
- Permission level cannot be OWNER (only original owner keeps OWNER)
- When share is revoked, corresponding FolderAccess is also deactivated
- Share message is optional but recommended for clarity

---

### Model 4: File

**Purpose:** Store uploaded files metadata

**Schema Overview:**
- File identification (name, original name)
- Parent folder reference
- Upload metadata (who uploaded, when)
- Storage location (S3 URL or local path)
- File properties (size, MIME type, extension)
- Soft delete support
- Timestamps

**Important Business Rules:**
- Files inherit access permissions from parent folder
- Soft delete preserves file metadata
- File size tracked in bytes for quota management
- Original filename preserved for download purposes

---

### Model 5: User (Reference - Already Exists)

**Purpose:** Store user information and role assignments

**Relevant Fields for DMS:**
- User identification (name, email)
- Role assignment (SUPER_ADMIN/ADMIN/DEPT_HEAD/FOLDER_MANAGER/FOLDER_USER)
- Department associations
- Account status

**Important Business Rules:**
- DEPT_HEAD: Assigned to ONE department
- ADMIN: Can be assigned to MULTIPLE departments
- Role determines access to organization folders
- isActive flag controls account access

---

### Model 6: Department (Reference - Already Exists)

**Purpose:** Store department information

**Schema Overview:**
- Department identification (name, code)
- Description
- Department head reference
- Active status

**Important Business Rules:**
- Each department has one head (DEPT_HEAD role)
- Department code must be unique
- Used for organization folder categorization
- isActive controls department visibility

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

### Access Control Logic Overview

#### For PERSONAL Folders:

**Step 1:** Check if user is the owner
- If YES → Full access (OWNER permissions)
- If NO → Go to Step 2

**Step 2:** Check FolderAccess table
- Look for active access record for this user
- Check if access has expired (compare expiresAt with current date)
- If found and valid → Grant access based on permission level
- If not found → DENY access

**Important:** Super Admin CANNOT access personal folders (privacy first)

#### For ORGANIZATION Folders:

**Step 1:** Check user role

**If SUPER_ADMIN:**
- Full access to all organization folders
- Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, MANAGE_USERS
- No department restrictions

**If ADMIN:**
- Check if user is assigned to this folder's department
- If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
- If NO → DENY

**If DEPT_HEAD:**
- Check if this is their assigned department
- If YES → Can VIEW, UPLOAD, DELETE, CREATE_FOLDER, ASSIGN_FOLDER_MANAGER
- If NO → DENY

**If FOLDER_MANAGER:**
- Check FolderAccess table for direct assignment
- Also check parent folders (inheritance)
- If found → Can VIEW, UPLOAD, DELETE, CREATE_SUBFOLDER
- If not found → DENY

**If FOLDER_USER:**
- Check FolderAccess table for direct assignment
- Also check parent folders (inheritance)
- If found → Can VIEW, UPLOAD
- If not found → DENY

---

## 🤝 Folder Sharing System {#folder-sharing-system}

### How Personal Folder Sharing Works

```
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
```

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

### Sharing Rules

**✅ CAN Share:**
- OWNER can share with anyone
- CO_OWNER can share with anyone
- Can share with multiple users
- Can set different permission levels for different users
- Can update existing share permissions

**❌ CANNOT Share:**
- EDITOR cannot share
- VIEWER cannot share
- FOLDER_MANAGER cannot share (they manage assigned folders only)
- Cannot share organization folders (only personal folders)
- Cannot share with yourself

### Sharing Workflow Steps

**Step 1: Initiate Share**
- User clicks "Share" on a personal folder
- System shows share dialog with user search

**Step 2: Select User and Permission**
- Search and select user to share with
- Choose permission level (VIEWER/EDITOR/CO_OWNER)
- Optional: Add share message

**Step 3: System Validation**
- Verify folder is PERSONAL type
- Verify current user can share (OWNER or CO_OWNER)
- Verify not sharing with self
- Check if already shared (update existing or create new)

**Step 4: Create Records**
- Create/Update FolderShare record
- Create/Update FolderAccess record
- Set accessSource = SHARED

**Step 5: Notification**
- Notify recipient about shared folder
- Include share message if provided

### Revoking Share

**Step 1: Initiate Revoke**
- Owner/CO_OWNER clicks "Revoke" on shared user

**Step 2: System Updates**
- Update FolderShare: Set isActive = false, add revokedAt, revokedBy
- Update FolderAccess: Set isActive = false

**Step 3: Access Removed**
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

## 👥 The 5 Roles Explained

### 🔴 Role 1: SUPER_ADMIN
**Who**: System owner, CTO, IT Head

**Powers**:
- ✅ Create/delete departments
- ✅ Access ALL organization folders in ALL departments
- ✅ Create/delete folders anywhere in organization drive
- ✅ Upload/delete files anywhere in organization drive
- ✅ Assign Admins to departments
- ✅ Assign Department Heads
- ✅ Assign Folder Managers and Folder Users
- ✅ Manage all users and roles

**Restrictions**:
- ❌ None for organization folders - bypasses all access checks
- ❌ **CANNOT access personal folders** (privacy respected)

**Example Scenario:**
```
Rajesh (Super Admin):
✅ Creates "Marketing" department
✅ Creates "Sales" department
✅ Makes Priya an Admin and assigns to Marketing + Sales
✅ Can access ANY file in ANY organization folder
✅ Can delete any organization folder
❌ Cannot see Sarah's personal "Draft Documents" folder (privacy)
```

---

### 🟠 Role 2: ADMIN
**Who**: Senior manager handling multiple departments

**Powers**:
- ✅ Access assigned departments (can be assigned to MULTIPLE departments)
- ✅ View all folders/files in assigned departments
- ✅ Create/delete root folders in assigned departments
- ✅ Upload/delete files in assigned departments
- ✅ Assign Folder Managers to folders in assigned departments
- ✅ Assign Folder Users to folders in assigned departments

**Restrictions**:
- ❌ Cannot create new departments
- ❌ Cannot access non-assigned departments
- ❌ Cannot assign other Admins (only Super Admin can)
- ❌ Cannot assign Department Heads
- ❌ Cannot access personal folders of any user

**Example Scenario:**
```
Priya (Admin assigned to Marketing + Sales departments):
✅ Can create root folders in Marketing department
✅ Can upload files to any folder in Sales department
✅ Can assign Rahul as Folder Manager to "Campaigns" folder
✅ Can delete files in Marketing or Sales
❌ Cannot access HR department (not assigned)
❌ Cannot create new departments
❌ Cannot make another user an Admin
❌ Cannot see anyone's personal folders
```

**Department Assignment:**
- Admin can be assigned to 1, 2, 3+ departments
- Super Admin controls these assignments
- Each assignment grants full access to that department's org folders

**Access Validation:**
```javascript
// For Admin accessing organization folder
if (user.role === 'ADMIN') {
  const folder = getFolder(folderId);
  if (user.assignedDepartments.includes(folder.departmentId)) {
    return ALLOW; // Admin has access to this department
  }
  return DENY; // Admin not assigned to this department
}
```

---

### 🟡 Role 3: DEPT_HEAD (Department Head)
**Who**: Head/Manager of ONE specific department

**Powers**:
- ✅ Full access to their assigned department
- ✅ View all folders/files in their department
- ✅ Create/delete root folders in their department
- ✅ Upload/delete files in their department
- ✅ Assign Folder Managers to folders in their department
- ✅ Assign Folder Users to folders in their department

**Restrictions**:
- ❌ Can only manage ONE department
- ❌ Cannot access other departments
- ❌ Cannot create new departments
- ❌ Cannot assign Admins or other Department Heads
- ❌ Cannot access personal folders

**Example Scenario:**
```
Sneha (Department Head of Marketing):
✅ Can create any root folder in Marketing department
✅ Can delete any file in Marketing department
✅ Can assign Rahul as Folder Manager to "Campaigns" folder
✅ Can upload files anywhere in Marketing
❌ Cannot see Sales department folders
❌ Cannot access HR department folders
❌ Cannot make anyone an Admin
❌ Cannot see personal folders
```

**Department Assignment:**
- Dept Head is assigned to exactly ONE department
- Cannot be changed without Super Admin
- Full control over their department (similar to Admin but single dept)

**Access Validation:**
```javascript
// For Dept Head accessing organization folder
if (user.role === 'DEPT_HEAD') {
  const folder = getFolder(folderId);
  if (user.assignedDepartment === folder.departmentId) {
    return ALLOW; // Dept Head manages this department
  }
  return DENY; // Not their department
}
```

---

### 🟢 Role 4: FOLDER_MANAGER
**Who**: Owner/Manager of specific folder(s) within a department

**Powers**:
- ✅ Full access to assigned folders
- ✅ View all files in assigned folders
- ✅ Upload/delete files in assigned folders
- ✅ Create subfolders inside assigned folders
- ✅ Delete subfolders inside assigned folders
- ✅ **Automatic access to ALL subfolders** (inheritance)

**Restrictions**:
- ❌ Cannot access folders they don't manage
- ❌ Cannot create root-level folders in department
- ❌ Cannot assign other Folder Managers or Folder Users
- ❌ Cannot access department level
- ❌ Cannot access personal folders

**Example Scenario:**
```
Rahul (Folder Manager assigned to "Campaign 2025" folder):
✅ Can upload files to "Campaign 2025"
✅ Can create "Design Assets" subfolder inside "Campaign 2025"
✅ Can delete any file in "Campaign 2025"
✅ Can delete "Design Assets" subfolder
✅ Automatically has access to all subfolders under "Campaign 2025"
❌ Cannot access "Social Media" folder (not assigned)
❌ Cannot create new root folder in Marketing department
❌ Cannot assign other users to folders
```

**Important - Automatic Subfolder Inheritance:**
```
Folder Manager assigned to: "Campaign 2025"
  │
  ├── Design Assets (✅ automatic access)
  │     ├── Logos (✅ automatic access)
  │     └── Banners (✅ automatic access)
  │
  └── Budget Files (✅ automatic access)

Folder Manager gets access to entire folder tree below assignment point!
```

**Access Validation:**
```javascript
// For Folder Manager accessing folder
if (user.role === 'FOLDER_MANAGER') {
  // Check direct assignment
  const hasDirectAccess = await FolderAccess.findOne({
    userId: user.id,
    folderId: folderId,
    accessSource: 'ASSIGNED_RBAC',
    isActive: true
  });
  
  if (hasDirectAccess) return ALLOW;
  
  // Check parent folder access (inheritance)
  const folder = await getFolder(folderId);
  if (folder.parentId) {
    return checkFolderManagerAccess(user, folder.parentId); // Recursive
  }
  
  return DENY;
}
```

---

### 🔵 Role 5: FOLDER_USER
**Who**: Team member who needs basic read and upload access

**Powers**:
- ✅ View folders they have access to
- ✅ View files in accessible folders
- ✅ Upload new files (basic contribution)
- ✅ **Automatic access to ALL subfolders** (inheritance)

**Restrictions**:
- ❌ Cannot delete any files or folders
- ❌ Cannot create new folders or subfolders
- ❌ Cannot manage or assign other users
- ❌ Cannot access folders not assigned to them
- ❌ Very limited access - read and upload only

**Example Scenario:**
```
Anjali (Folder User assigned to "Campaign 2025" folder):
✅ Can view files in "Campaign 2025"
✅ Can upload her work files to "Campaign 2025"
✅ Can view all subfolders under "Campaign 2025"
✅ Can upload to subfolders
❌ Cannot delete anything
❌ Cannot create subfolders
❌ Cannot give access to others
❌ Cannot access "Social Media" folder (not assigned)
```

**Automatic Subfolder Inheritance:**
```
Folder User assigned to: "Campaign 2025"
  │
  ├── Design Assets (✅ can view + upload)
  │     ├── Logos (✅ can view + upload)
  │     └── Banners (✅ can view + upload)
  │
  └── Budget Files (✅ can view + upload)

Folder User can contribute to entire folder tree!
```

**Access Validation:**
```javascript
// For Folder User accessing folder
if (user.role === 'FOLDER_USER') {
  // Check direct assignment or parent inheritance
  const hasAccess = await checkFolderAccess(user.id, folderId);
  
  if (hasAccess) {
    // Only allow VIEW and UPLOAD actions
    if (action === 'VIEW' || action === 'UPLOAD') {
      return ALLOW;
    }
    return DENY; // Cannot DELETE or CREATE_FOLDER
  }
  
  return DENY;
}
```

---

## 🎯 Role Permissions Matrix

| Role | Scope | View | Upload | Delete | Create Folder | Manage Users | Access Personal |
|------|-------|------|--------|--------|---------------|--------------|-----------------|
| **SUPER_ADMIN** | All Org Folders | ✅ | ✅ | ✅ | ✅ Root Level | ✅ All Roles | ❌ |
| **ADMIN** | Assigned Depts | ✅ | ✅ | ✅ | ✅ Root Level | ✅ FM/FU Only | ❌ |
| **DEPT_HEAD** | Own Dept | ✅ | ✅ | ✅ | ✅ Root Level | ✅ FM/FU Only | ❌ |
| **FOLDER_MANAGER** | Assigned Folders | ✅ | ✅ | ✅ | ✅ Subfolders | ❌ | ❌ |
| **FOLDER_USER** | Assigned Folders | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## 🔄 Access Control Implementation

### Central Access Function

```javascript
/**
 * Single source of truth for all access decisions
 * @param {Object} user - Current user object
 * @param {Object} resource - Folder/File being accessed
 * @param {String} action - Action to perform (VIEW, UPLOAD, DELETE, etc.)
 * @returns {Boolean} - true if allowed, false if denied
 */
async function canAccess(user, resource, action) {
  // GATE 1: Check folder type
  const folder = resource.type === 'FILE' 
    ? await getFolder(resource.folderId) 
    : resource;
  
  // PERSONAL FOLDER LOGIC
  if (folder.type === 'PERSONAL') {
    // Check ownership
    if (folder.ownerId.toString() === user.id.toString()) {
      return true; // Owner has full access
    }
    
    // Check if shared with user
    const access = await FolderAccess.findOne({
      folderId: folder.id,
      userId: user.id,
      accessSource: 'SHARED',
      isActive: true
    });
    
    if (!access) return false;
    
    // Check if access expired
    if (access.expiresAt && access.expiresAt < new Date()) {
      return false;
    }
    
    // Check permission level allows this action
    return permissionAllows(access.accessType, action);
  }
  
  // ORGANIZATION FOLDER LOGIC
  if (folder.type === 'ORGANIZATION') {
    // Gate 1: Super Admin bypass
    if (user.role === 'SUPER_ADMIN') {
      return true;
    }
    
    // Gate 2: Department-level access (Admin/Dept Head)
    if (user.role === 'ADMIN') {
      if (!user.assignedDepartments.includes(folder.departmentId)) {
        return false;
      }
      return roleAllows(user.role, action);
    }
    
    if (user.role === 'DEPT_HEAD') {
      if (user.assignedDepartment !== folder.departmentId) {
        return false;
      }
      return roleAllows(user.role, action);
    }
    
    // Gate 3: Folder-level access (Folder Manager/User)
    if (user.role === 'FOLDER_MANAGER' || user.role === 'FOLDER_USER') {
      const hasAccess = await userHasFolderAccess(user, folder.id);
      if (!hasAccess) return false;
      return roleAllows(user.role, action);
    }
  }
  
  // Default: Deny
  return false;
}

// Define what each role can do
const ROLE_PERMISSIONS = {
  SUPER_ADMIN: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'MANAGE_USERS'],
  ADMIN: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'ASSIGN_FOLDER_MANAGER'],
  DEPT_HEAD: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'ASSIGN_FOLDER_MANAGER'],
  FOLDER_MANAGER: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_SUBFOLDER'],
  FOLDER_USER: ['VIEW', 'UPLOAD']
};

function roleAllows(role, action) {
  return ROLE_PERMISSIONS[role]?.includes(action) || false;
}

// Personal folder sharing permissions
const SHARE_PERMISSIONS = {
  OWNER: ['VIEW', 'UPLOAD', 'DELETE', 'SHARE', 'MANAGE'],
  CO_OWNER: ['VIEW', 'UPLOAD', 'DELETE', 'SHARE'],
  EDITOR: ['VIEW', 'UPLOAD', 'DELETE'],
  VIEWER: ['VIEW']
};

function permissionAllows(accessType, action) {
  return SHARE_PERMISSIONS[accessType]?.includes(action) || false;
}

// Check folder access with inheritance
async function userHasFolderAccess(user, folderId) {
  // Check direct assignment
  const directAccess = await FolderAccess.findOne({
    userId: user.id,
    folderId: folderId,
    accessSource: 'ASSIGNED_RBAC',
    isActive: true
  });
  
  if (directAccess) return true;
  
  // Check parent folder access (inheritance)
  const folder = await Folder.findById(folderId);
  if (folder && folder.parentId) {
    return userHasFolderAccess(user, folder.parentId); // Recursive
  }
  
  return false;
}
```

---

## 📊 Department-Based Access Examples

### Organization Structure

```
Company
├── Marketing Department (Dept Head: Sneha)
│   ├── Campaigns Folder
│   │   ├── Q1-2024 (subfolder)
│   │   └── Q2-2024 (subfolder)
│   └── Analytics Folder
│
├── Engineering Department (Dept Head: Tom)
│   ├── Projects Folder
│   │   ├── Project-A (subfolder)
│   │   └── Project-B (subfolder)
│   └── Documentation Folder
│
└── HR Department (Dept Head: Rachel)
    ├── Recruitment Folder
    └── Policies Folder
```

### Access Scenarios

**1. Super Admin (Mike):**
```
✅ Access to ALL folders in ALL departments
✅ Marketing/Campaigns: Full access
✅ Engineering/Projects: Full access
✅ HR/Recruitment: Full access
✅ Can create folders in any department
✅ Can assign any role to any user
```

**2. Admin (Lisa - assigned to Marketing + Engineering):**
```
✅ Marketing/Campaigns: Full access
✅ Marketing/Analytics: Full access
✅ Engineering/Projects: Full access
✅ Engineering/Documentation: Full access
❌ HR/Recruitment: NO access (not assigned to HR)
❌ HR/Policies: NO access (not assigned to HR)
✅ Can assign Folder Managers in Marketing and Engineering
❌ Cannot assign Folder Managers in HR
```

**3. Dept Head (Sneha - Marketing):**
```
✅ Marketing/Campaigns: Full access
✅ Marketing/Analytics: Full access
✅ Can create any folder in Marketing
✅ Can assign Folder Managers in Marketing
❌ Engineering/Projects: NO access (different department)
❌ HR/Recruitment: NO access (different department)
```

**4. Folder Manager (Rahul - assigned to "Campaigns"):**
```
✅ Campaigns Folder: Full access
✅ Q1-2024: Automatic access (inheritance)
✅ Q2-2024: Automatic access (inheritance)
✅ Can create subfolders under Campaigns
✅ Can delete files in any subfolder
❌ Analytics Folder: NO access (not assigned)
❌ Cannot create root folder in Marketing
```

**5. Folder User (Anjali - assigned to "Project-A"):**
```
✅ Project-A: Can view and upload
✅ Any subfolders under Project-A: Can view and upload (inheritance)
❌ Project-B: NO access (not assigned)
❌ Cannot delete any files
❌ Cannot create folders
```

---

## 🔄 Folder Assignment Flow

### How Admin/Dept Head Assigns Folder Access

```
┌─────────────────────────────────────────────────┐
│         ADMIN/DEPT_HEAD ASSIGNS FOLDER          │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Admin/Dept Head selects organization folder │
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

### Access Inheritance Visualization

```
Parent Folder: "Engineering Department"
├── Child: "Projects" (Folder Manager assigned here)
│   ├── Grandchild: "Project-A" (✅ automatic access)
│   └── Grandchild: "Project-B" (✅ automatic access)

If Folder Manager is assigned to "Projects":
✅ Can access "Projects"
✅ Can access "Project-A" (inherited)
✅ Can access "Project-B" (inherited)
✅ Can create new subfolder under "Projects"
❌ Cannot access "Engineering Department" (parent - no upward inheritance)

If Folder User is assigned to "Project-A":
✅ Can access "Project-A"
✅ Can access any subfolders under "Project-A" (inherited)
❌ Cannot access "Projects" (parent)
❌ Cannot access "Project-B" (sibling)
```

**Key Rule:** Inheritance flows DOWN the folder tree, never UP.

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
- Bob (Freelancer - external consultant)

**Organization Structure:**
```
Organization Drive
└── Marketing Department
    ├── Q4 Campaigns (org folder)
    │   ├── Social Media Assets
    │   └── Email Templates
    └── Brand Guidelines (org folder)
```

**Personal Work:**
```
Sarah's My Drive
└── Campaign Drafts (personal folder)
    ├── Draft-1.psd
    └── Draft-2.psd
```

**Workflow:**

1. **Sarah creates campaign draft in her personal drive**
   - Creates "Campaign Drafts" folder (type: PERSONAL)
   - Uploads draft designs
   - Folder is completely private
   - Even Super Admin cannot see it

2. **Sarah shares draft with John for feedback**
   - Shares "Campaign Drafts" with John
   - Grants EDITOR permission
   - John can now view and add comments
   - Folder appears in John's "Shared with Me"
   - FolderShare and FolderAccess records created

3. **John reviews and approves**
   - Accesses folder from "Shared with Me"
   - Reviews files
   - Uploads revised versions (EDITOR permission)

4. **Sarah shares with Bob (freelancer)**
   - Shares same folder with Bob
   - Grants VIEWER permission only
   - Bob can view but not modify
   - Bob does not have access to organization folders

5. **Sarah assigns John as Folder Manager for Q4 Campaigns**
   - Goes to Organization Drive
   - Selects "Q4 Campaigns" folder
   - Assigns John as FOLDER_MANAGER
   - Creates FolderAccess with accessSource = ASSIGNED_RBAC
   - John now has full control of Q4 Campaigns and all subfolders

6. **Sarah assigns Alice as Folder User**
   - Assigns Alice to "Q4 Campaigns"
   - Alice can view and upload
   - Alice cannot delete or create folders

7. **Final assets moved to organization**
   - Sarah uploads final assets to "Q4 Campaigns"
   - John (Folder Manager) organizes them into subfolders
   - Alice (Folder User) can view and download
   - Bob cannot access organization folders

**Access Summary:**

**Campaign Drafts (Personal Folder):**
- Sarah: OWNER (created it)
- John: EDITOR (shared by Sarah)
- Bob: VIEWER (shared by Sarah)
- Alice: NO ACCESS
- Super Admin: NO ACCESS (privacy)

**Q4 Campaigns (Organization Folder):**
- Sarah: Full access (Dept Head)
- John: Full access (Folder Manager assigned)
- Alice: View + Upload (Folder User assigned)
- Bob: NO ACCESS (not in organization)
- Super Admin: Full access (role-based)

---

### Example 2: Engineering Documentation with Strict Access

**Scenario:**
Engineering team manages project documentation with strict access control based on roles.

**Setup:**

**Users:**
- Mike (Super Admin)
- Lisa (Admin - Engineering + HR)
- Tom (Dept Head - Engineering)
- Emma (Folder Manager - Project A)
- David (Folder User - Project A)
- Sarah (Folder User - Project B)

**Organization Structure:**
```
Organization Drive
├── Engineering Department
│   ├── Project A Documentation
│   │   ├── Requirements
│   │   ├── Design Docs
│   │   └── API Specs
│   ├── Project B Documentation
│   │   ├── Architecture
│   │   └── Code Review
│   └── Internal Tools
│
└── HR Department
    └── Employee Handbook
```

**Workflow:**

1. **Tom creates folder structure**
   - As Dept Head of Engineering
   - Creates "Project A Documentation" root folder
   - Creates subfolders: Requirements, Design Docs, API Specs
   - All inherit Engineering department access

2. **Tom assigns Emma as Folder Manager**
   - Selects "Project A Documentation"
   - Assigns Emma as FOLDER_MANAGER
   - Emma gets full control of Project A folder
   - Emma automatically gets access to all subfolders (inheritance)

3. **Emma assigns David as Folder User**
   - Emma selects "Project A Documentation"
   - Assigns David as FOLDER_USER
   - David gets view + upload access
   - David can contribute documents but cannot delete

4. **Tom assigns Sarah to Project B**
   - Assigns Sarah as FOLDER_USER to "Project B Documentation"
   - Sarah can only access Project B, not Project A

**Access Validation Scenarios:**

**Mike (Super Admin):**
```
Action: Access "Project A/Requirements" folder
✅ ALLOWED
Reason: Super Admin bypasses all checks
Can also: Create, Delete, Upload anywhere

Action: Access Sarah's personal folder
❌ DENIED
Reason: Personal folders respect privacy even for Super Admin
```

**Lisa (Admin - Engineering + HR):**
```
Action: Upload file to "Project A/Design Docs"
✅ ALLOWED
Reason: Admin assigned to Engineering department

Action: Create folder in HR/Employee Handbook
✅ ALLOWED
Reason: Admin assigned to HR department

Action: Access Marketing folder
❌ DENIED
Reason: Not assigned to Marketing department
```

**Tom (Dept Head - Engineering):**
```
Action: Delete "Project B/Architecture" folder
✅ ALLOWED
Reason: Dept Head of Engineering

Action: Assign Folder Manager to "Project A"
✅ ALLOWED
Reason: Dept Heads can assign FM/FU in their department

Action: Access HR/Employee Handbook
❌ DENIED
Reason: Not his department
```

**Emma (Folder Manager - Project A):**
```
Action: Create subfolder under "API Specs"
✅ ALLOWED
Reason: Folder Manager can create subfolders

Action: Delete file in "Requirements" folder
✅ ALLOWED
Reason: Has access to all subfolders (inheritance)

Action: Access "Project B" folder
❌ DENIED
Reason: Not assigned to Project B

Action: Create root folder in Engineering
❌ DENIED
Reason: Folder Managers cannot create root folders
```

**David (Folder User - Project A):**
```
Action: View files in "Design Docs"
✅ ALLOWED
Reason: Subfolder inheritance from Project A assignment

Action: Upload document to "API Specs"
✅ ALLOWED
Reason: Folder Users can upload

Action: Delete file in "Requirements"
❌ DENIED
Reason: Folder Users cannot delete

Action: Create subfolder under "Design Docs"
❌ DENIED
Reason: Folder Users cannot create folders
```

**Sarah (Folder User - Project B):**
```
Action: View files in "Project B/Architecture"
✅ ALLOWED
Reason: Assigned to Project B with inheritance

Action: Access "Project A/Requirements"
❌ DENIED
Reason: Not assigned to Project A (different folder)
```

5. **Emma creates personal backup**
   - Creates personal folder "My Project Notes" (type: PERSONAL)
   - Uploads personal copies for reference
   - Completely private, even Tom cannot see
   - Emma can share with David if needed using sharing system

---

### Example 3: HR Confidential Documents

**Scenario:**
HR needs to manage sensitive employee documents with strict confidentiality and folder-level access control.

**Setup:**

**Users:**
- Mike (Super Admin)
- Rachel (Dept Head - HR)
- Kevin (Folder Manager - Recruitment)
- Nina (Folder User - Payroll)
- John (Folder Manager - Marketing, not HR)

**Organization Structure:**
```
Organization Drive
└── HR Department
    ├── Recruitment
    │   ├── Candidate Profiles
    │   └── Interview Notes
    ├── Payroll
    │   ├── Salary Data
    │   └── Tax Documents
    └── Policies
        └── Employee Handbook
```

**Workflow:**

1. **Rachel creates HR folder structure**
   - As Dept Head of HR
   - Creates department folders
   - Sets up sensitive areas: Recruitment, Payroll, Policies

2. **Kevin assigned to Recruitment only**
   - Rachel assigns Kevin as FOLDER_MANAGER to "Recruitment" folder
   - Kevin gets full access to Recruitment
   - Kevin can manage candidate files
   - Kevin automatically gets access to:
     - ✅ Candidate Profiles (subfolder)
     - ✅ Interview Notes (subfolder)
   - Kevin CANNOT access:
     - ❌ Payroll (different folder, not assigned)
     - ❌ Policies (different folder, not assigned)

3. **Nina assigned to Payroll only**
   - Rachel assigns Nina as FOLDER_USER to "Payroll" folder
   - Nina can view and upload salary documents
   - Nina automatically gets access to:
     - ✅ Salary Data (subfolder)
     - ✅ Tax Documents (subfolder)
   - Nina CANNOT access:
     - ❌ Recruitment (different folder, not assigned)
     - ❌ Policies (different folder, not assigned)
   - Nina CANNOT delete anything (Folder User restriction)

**Access Isolation Examples:**

**Kevin tries to access Payroll:**
```javascript
// Kevin (Folder Manager - Recruitment) tries to view Payroll
Action: View "HR/Payroll" folder
❌ DENIED

Validation Flow:
1. User role: FOLDER_MANAGER
2. Check folder assignment: Kevin assigned to "Recruitment"
3. Check if "Payroll" is under "Recruitment": NO
4. Check parent inheritance: NO
5. Result: DENY ACCESS

Reason: Folder Managers only access assigned folders
```

**Nina tries to delete salary document:**
```javascript
// Nina (Folder User - Payroll) tries to delete file
Action: Delete "Salary Data/john-salary.pdf"
❌ DENIED

Validation Flow:
1. User role: FOLDER_USER
2. Check folder access: Nina assigned to "Payroll" ✓
3. Check subfolder inheritance: "Salary Data" under "Payroll" ✓
4. Check action permission: FOLDER_USER cannot DELETE
5. Result: DENY ACTION

Reason: Folder Users can only VIEW and UPLOAD
```

**Rachel's full access:**
```javascript
// Rachel (Dept Head - HR) accesses any HR folder
Action: View "Recruitment/Candidate Profiles"
✅ ALLOWED

Action: Delete "Payroll/Tax Documents/old-file.pdf"
✅ ALLOWED

Action: Create new root folder "Benefits"
✅ ALLOWED

Reason: Dept Head has full access to entire department
```

**Mike (Super Admin) accesses HR:**
```javascript
// Mike (Super Admin) accesses HR folders
Action: View "HR/Payroll/Salary Data"
✅ ALLOWED

Action: Upload to "HR/Recruitment"
✅ ALLOWED

Reason: Super Admin bypasses all department and folder restrictions
```

**John (Folder Manager - Marketing) tries HR:**
```javascript
// John (Folder Manager in Marketing) tries to access HR
Action: View "HR/Policies"
❌ DENIED

Validation Flow:
1. User role: FOLDER_MANAGER
2. Check department: HR department, John is Marketing
3. Check if ADMIN or DEPT_HEAD: NO
4. Check folder assignment: John not assigned to any HR folder
5. Result: DENY ACCESS

Reason: Folder Managers from other departments have no access
```

4. **Rachel's personal review notes**
   - Creates personal folder "Employee Reviews 2024" (type: PERSONAL)
   - Writes confidential review notes
   - Even Super Admin Mike cannot access (privacy rule)
   - Rachel can selectively share specific reviews:
     - Share "Performance Review - Kevin" with Kevin (VIEWER)
     - Keep other reviews completely private

**Access Summary Table:**

| User | Role | Recruitment | Payroll | Policies | Actions |
|------|------|-------------|---------|----------|---------|
| Mike | Super Admin | ✅ Full | ✅ Full | ✅ Full | All |
| Rachel | Dept Head (HR) | ✅ Full | ✅ Full | ✅ Full | All |
| Kevin | Folder Manager | ✅ Full | ❌ None | ❌ None | View/Upload/Delete/Create Sub |
| Nina | Folder User | ❌ None | ✅ Limited | ❌ None | View/Upload only |
| John | Folder Manager (Mkt) | ❌ None | ❌ None | ❌ None | No access |

---

### Example 4: Cross-Department Project

**Scenario:**
Marketing and Engineering need to collaborate on product launch, but they're in different departments.

**Setup:**

**Users:**
- Sarah (Dept Head - Marketing)
- Tom (Dept Head - Engineering)
- Lisa (Admin - Marketing + Engineering)
- John (Folder Manager - Marketing)
- Emma (Folder Manager - Engineering)

**Challenge:** How to enable cross-department collaboration?

**Solution 1: Use Admin with Multiple Department Access**

```
Organization Drive
└── Marketing Department
    └── Product Launch 2024
        ├── Marketing Materials (Marketing team)
        └── Technical Specs (Engineering team)
```

**Setup:**
1. Lisa (Admin assigned to both Marketing and Engineering)
2. Lisa creates "Product Launch 2024" folder in Marketing
3. Lisa assigns John (Marketing) as Folder Manager
4. Lisa assigns Emma (Engineering) as Folder Manager to same folder
5. Both teams can now collaborate

**Access:**
- Sarah: ✅ Full access (Dept Head - Marketing)
- Tom: ❌ Cannot access (Dept Head - Engineering, different dept)
- Lisa: ✅ Full access (Admin - both departments)
- John: ✅ Full access (Folder Manager assigned)
- Emma: ✅ Full access (Folder Manager assigned)

**Limitation:** Tom (Engineering Dept Head) cannot access because folder is in Marketing department.

---

**Solution 2: Personal Folder with Sharing (Recommended for Cross-Dept)**

```
Sarah's My Drive
└── Product Launch Collaboration (PERSONAL folder)
    ├── Joint Planning Docs
    └── Shared Resources
```

**Setup:**
1. Sarah creates personal folder "Product Launch Collaboration"
2. Sarah shares with Tom (CO_OWNER) - Tom can share further
3. Sarah shares with John (EDITOR) - can modify
4. Sarah shares with Emma (EDITOR) - can modify
5. Everyone collaborates in personal folder
6. When finalized, upload to respective department org folders

**Access via Sharing:**
- Sarah: OWNER (created it)
- Tom: CO_OWNER (shared, can reshare)
- John: EDITOR (shared, can modify)
- Emma: EDITOR (shared, can modify)
- Lisa: Not shared (even as Admin, must be explicitly shared)

**Benefits:**
- True cross-department collaboration
- Flexible access control
- Easy to add/remove collaborators
- Final assets can be moved to official dept folders

**Workflow:**
```
1. Collaborate in Sarah's personal "Product Launch Collaboration"
   └── Everyone contributes (shared access)

2. Finalize materials
   └── Sarah reviews and approves

3. Upload to Organization Drive:
   ├── Marketing materials → Marketing/Product Launch 2024
   └── Technical docs → Engineering/Product Documentation
```

---

## 🧪 Edge Cases & Testing Scenarios {#edge-cases}

### Edge Case 1: Nested Folder Access with Explicit Denial

**Scenario:** User has access to parent folder but tries to access sibling folder not assigned to them.

**Setup:**
```
Engineering Department
├── Projects (root)
│   ├── Project A
│   │   └── Requirements
│   └── Project B
│       └── Architecture
```

**Test Cases:**

**Test 1: Folder Manager assigned to Project A**
```
User: Emma (Folder Manager)
Assignment: Project A

Attempt: Access "Project A/Requirements"
Expected: ✅ ALLOWED (inheritance from Project A)

Attempt: Access "Project B"
Expected: ❌ DENIED (sibling folder, not in inheritance chain)

Attempt: Access "Projects" (parent)
Expected: ❌ DENIED (no upward inheritance)
```

**Test 2: Folder User assigned to Project B/Architecture**
```
User: David (Folder User)
Assignment: Project B/Architecture

Attempt: Access "Project B/Architecture"
Expected: ✅ ALLOWED (direct assignment)

Attempt: Access "Project B" (parent)
