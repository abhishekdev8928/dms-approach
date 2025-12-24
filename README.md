# 🏢 DMS Hybrid System - Complete Design Documentation

## Document Management System with Personal Drive + Organization Drive + Sharing

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Design](#architecture-design)
3. [Mongoose Schema Models](#mongoose-schema)
4. [Access Control Logic](#access-control-logic)
5. [Folder Sharing System](#folder-sharing-system)
6. [API Endpoints](#api-endpoints)
7. [Implementation Guide](#implementation-guide)
8. [Feature Comparison](#feature-comparison)
9. [Security Rules](#security-rules)
10. [Testing Scenarios](#testing-scenarios)

---

## 🎯 System Overview {#system-overview}

### What We're Building

A **Hybrid Document Management System** with three main components:

1. **Organization Drive** - Company-wide shared folders with RBAC (5 roles)
2. **My Drive** - Personal private space for each user
3. **Shared Folders** - Personal folders that users can share with others

### Key Design Decisions (Based on Your Requirements)

✅ **Personal folders CAN be shared** with other users  
✅ **NO promotion** from personal to organization folders  
✅ **Super Admin CANNOT see** personal folders (privacy first)  
✅ **Same storage limits** for both personal and organization drives

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

## 🏗️ Architecture Design {#architecture-design}

### Approach: Single Table with Access Control Table

We're using **Approach 3** (Enterprise Grade) because:
- ✅ You need personal folder sharing
- ✅ Flexible permission system
- ✅ Easy to grant/revoke access
- ✅ Audit trail built-in

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

### Collections Overview

```
folders
├── Stores all folders (personal + organization)
├── Type distinguishes personal vs organization
└── Owner tracks creator

folderAccess
├── Who has access to which folder
├── What level of access (OWNER/EDITOR/VIEWER)
└── How they got access (CREATED/ASSIGNED/SHARED)

folderShares
├── Tracks sharing invitations
├── Share metadata (message, expiry)
└── Share history

files
├── Stores all uploaded files
├── Links to parent folder
└── Uploaded by which user
```

---

## 💾 Mongoose Schema Models {#mongoose-schema}

### 1. Folder Model

```javascript
const mongoose = require('mongoose');

const folderSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Folder name is required'],
    trim: true,
    maxlength: [255, 'Folder name cannot exceed 255 characters']
  },
  
  folderType: {
    type: String,
    enum: {
      values: ['PERSONAL', 'ORGANIZATION'],
      message: 'Folder type must be either PERSONAL or ORGANIZATION'
    },
    required: [true, 'Folder type is required']
  },
  
  departmentId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department',
    default: null,
    // Validation: Required for ORGANIZATION, must be null for PERSONAL
    validate: {
      validator: function(value) {
        if (this.folderType === 'ORGANIZATION') {
          return value != null;
        }
        if (this.folderType === 'PERSONAL') {
          return value == null;
        }
        return true;
      },
      message: 'Organization folders must have departmentId, Personal folders must not'
    }
  },
  
  ownerId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Owner ID is required']
  },
  
  parentId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    default: null
  },
  
  // Additional metadata
  description: {
    type: String,
    maxlength: [500, 'Description cannot exceed 500 characters'],
    default: ''
  },
  
  isDeleted: {
    type: Boolean,
    default: false
  },
  
  deletedAt: {
    type: Date,
    default: null
  }
  
}, {
  timestamps: true  // Automatically adds createdAt and updatedAt
});

// Indexes for performance
folderSchema.index({ folderType: 1 });
folderSchema.index({ ownerId: 1 });
folderSchema.index({ departmentId: 1 });
folderSchema.index({ parentId: 1 });
folderSchema.index({ folderType: 1, ownerId: 1 });
folderSchema.index({ folderType: 1, departmentId: 1 });

// Virtual for checking if folder is root level
folderSchema.virtual('isRoot').get(function() {
  return this.parentId == null;
});

// Method to check if folder is personal
folderSchema.methods.isPersonal = function() {
  return this.folderType === 'PERSONAL';
};

// Method to check if folder is organization
folderSchema.methods.isOrganization = function() {
  return this.folderType === 'ORGANIZATION';
};

// Static method to find all personal folders of a user
folderSchema.statics.findPersonalFolders = function(userId) {
  return this.find({
    folderType: 'PERSONAL',
    ownerId: userId,
    isDeleted: false
  });
};

// Static method to find all organization folders in a department
folderSchema.statics.findDepartmentFolders = function(departmentId) {
  return this.find({
    folderType: 'ORGANIZATION',
    departmentId: departmentId,
    isDeleted: false
  });
};

module.exports = mongoose.model('Folder', folderSchema);
```

---

### 2. FolderAccess Model

```javascript
const mongoose = require('mongoose');

const folderAccessSchema = new mongoose.Schema({
  folderId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: [true, 'Folder ID is required']
  },
  
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'User ID is required']
  },
  
  accessType: {
    type: String,
    enum: {
      values: ['OWNER', 'CO_OWNER', 'EDITOR', 'VIEWER'],
      message: 'Access type must be OWNER, CO_OWNER, EDITOR, or VIEWER'
    },
    required: [true, 'Access type is required']
  },
  
  accessSource: {
    type: String,
    enum: {
      values: ['CREATED', 'ASSIGNED_RBAC', 'SHARED'],
      message: 'Access source must be CREATED, ASSIGNED_RBAC, or SHARED'
    },
    required: [true, 'Access source is required']
  },
  
  grantedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    default: null
  },
  
  grantedAt: {
    type: Date,
    default: Date.now
  },
  
  expiresAt: {
    type: Date,
    default: null  // null means no expiry
  },
  
  isActive: {
    type: Boolean,
    default: true
  }
  
}, {
  timestamps: true
});

// Compound index to ensure one user has only one access type per folder
folderAccessSchema.index({ folderId: 1, userId: 1 }, { unique: true });

// Additional indexes
folderAccessSchema.index({ userId: 1 });
folderAccessSchema.index({ folderId: 1, isActive: 1 });
folderAccessSchema.index({ expiresAt: 1 });

// Method to check if access has expired
folderAccessSchema.methods.hasExpired = function() {
  if (!this.expiresAt) return false;
  return new Date() > this.expiresAt;
};

// Method to check if user can perform specific action
folderAccessSchema.methods.canPerform = function(action) {
  const permissions = {
    OWNER: ['VIEW', 'UPLOAD', 'DELETE', 'SHARE', 'MANAGE'],
    CO_OWNER: ['VIEW', 'UPLOAD', 'DELETE', 'SHARE'],
    EDITOR: ['VIEW', 'UPLOAD', 'DELETE'],
    VIEWER: ['VIEW']
  };
  
  return permissions[this.accessType]?.includes(action) || false;
};

// Static method to find all folders a user has access to
folderAccessSchema.statics.findUserAccess = function(userId) {
  return this.find({
    userId: userId,
    isActive: true,
    $or: [
      { expiresAt: null },
      { expiresAt: { $gt: new Date() } }
    ]
  }).populate('folderId');
};

// Static method to find all users who have access to a folder
folderAccessSchema.statics.findFolderUsers = function(folderId) {
  return this.find({
    folderId: folderId,
    isActive: true
  }).populate('userId');
};

// Static method to grant access
folderAccessSchema.statics.grantAccess = async function(folderId, userId, accessType, grantedBy, accessSource = 'SHARED') {
  return this.findOneAndUpdate(
    { folderId, userId },
    {
      folderId,
      userId,
      accessType,
      accessSource,
      grantedBy,
      grantedAt: new Date(),
      isActive: true
    },
    { upsert: true, new: true }
  );
};

// Static method to revoke access
folderAccessSchema.statics.revokeAccess = async function(folderId, userId) {
  return this.findOneAndUpdate(
    { folderId, userId },
    { isActive: false },
    { new: true }
  );
};

module.exports = mongoose.model('FolderAccess', folderAccessSchema);
```

---

### 3. FolderShare Model

```javascript
const mongoose = require('mongoose');

const folderShareSchema = new mongoose.Schema({
  folderId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: [true, 'Folder ID is required']
  },
  
  sharedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Shared by user ID is required']
  },
  
  sharedWith: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Shared with user ID is required']
  },
  
  permissionLevel: {
    type: String,
    enum: {
      values: ['VIEWER', 'EDITOR', 'CO_OWNER'],
      message: 'Permission level must be VIEWER, EDITOR, or CO_OWNER'
    },
    required: [true, 'Permission level is required']
  },
  
  shareMessage: {
    type: String,
    maxlength: [500, 'Share message cannot exceed 500 characters'],
    default: ''
  },
  
  isActive: {
    type: Boolean,
    default: true
  },
  
  revokedAt: {
    type: Date,
    default: null
  },
  
  revokedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    default: null
  }
  
}, {
  timestamps: true  // createdAt, updatedAt
});

// Indexes
folderShareSchema.index({ folderId: 1 });
folderShareSchema.index({ sharedBy: 1 });
folderShareSchema.index({ sharedWith: 1 });
folderShareSchema.index({ isActive: 1 });
folderShareSchema.index({ folderId: 1, sharedWith: 1 });

// Method to revoke share
folderShareSchema.methods.revoke = function(revokedBy) {
  this.isActive = false;
  this.revokedAt = new Date();
  this.revokedBy = revokedBy;
  return this.save();
};

// Static method to find active shares for a folder
folderShareSchema.statics.findActiveShares = function(folderId) {
  return this.find({
    folderId: folderId,
    isActive: true
  }).populate('sharedWith', 'name email');
};

// Static method to find folders shared with a user
folderShareSchema.statics.findSharedWithUser = function(userId) {
  return this.find({
    sharedWith: userId,
    isActive: true
  }).populate('folderId sharedBy');
};

// Static method to check if folder is shared with specific user
folderShareSchema.statics.isSharedWith = async function(folderId, userId) {
  const share = await this.findOne({
    folderId: folderId,
    sharedWith: userId,
    isActive: true
  });
  return share != null;
};

module.exports = mongoose.model('FolderShare', folderShareSchema);
```

---

### 4. File Model

```javascript
const mongoose = require('mongoose');

const fileSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'File name is required'],
    trim: true,
    maxlength: [255, 'File name cannot exceed 255 characters']
  },
  
  folderId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: [true, 'Folder ID is required']
  },
  
  uploadedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Uploader ID is required']
  },
  
  fileUrl: {
    type: String,
    required: [true, 'File URL is required']
  },
  
  fileSize: {
    type: Number,
    required: [true, 'File size is required'],
    min: [0, 'File size cannot be negative']
  },
  
  mimeType: {
    type: String,
    required: [true, 'MIME type is required']
  },
  
  // Additional metadata
  originalName: {
    type: String,
    required: true
  },
  
  fileExtension: {
    type: String,
    required: true
  },
  
  isDeleted: {
    type: Boolean,
    default: false
  },
  
  deletedAt: {
    type: Date,
    default: null
  },
  
  deletedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    default: null
  }
  
}, {
  timestamps: true
});

// Indexes
fileSchema.index({ folderId: 1 });
fileSchema.index({ uploadedBy: 1 });
fileSchema.index({ folderId: 1, isDeleted: 1 });
fileSchema.index({ mimeType: 1 });

// Virtual for file size in MB
fileSchema.virtual('fileSizeMB').get(function() {
  return (this.fileSize / (1024 * 1024)).toFixed(2);
});

// Method to soft delete file
fileSchema.methods.softDelete = function(userId) {
  this.isDeleted = true;
  this.deletedAt = new Date();
  this.deletedBy = userId;
  return this.save();
};

// Static method to find files in folder
fileSchema.statics.findByFolder = function(folderId) {
  return this.find({
    folderId: folderId,
    isDeleted: false
  }).sort({ createdAt: -1 });
};

// Static method to calculate total size of files in folder
fileSchema.statics.calculateFolderSize = async function(folderId) {
  const result = await this.aggregate([
    { $match: { folderId: mongoose.Types.ObjectId(folderId), isDeleted: false } },
    { $group: { _id: null, totalSize: { $sum: '$fileSize' } } }
  ]);
  return result[0]?.totalSize || 0;
};

module.exports = mongoose.model('File', fileSchema);
```

---

## 🔐 Access Control Logic {#access-control-logic}

### Decision Flow Chart

```
User wants to access Folder X
         ↓
┌────────────────────────┐
│ What is folder type?   │
└────────┬───────────────┘
         ↓
    ┌────┴────┐
    │         │
PERSONAL  ORGANIZATION
    │         │
    ↓         ↓
┌────────┐  ┌──────────────┐
│ Is user│  │ Check RBAC   │
│ owner? │  │ permissions  │
└───┬────┘  └──────┬───────┘
    │              │
   YES            ┌┴─────────────┐
    │             │ Super Admin? │
    ↓             └┬─────────────┘
┌────────┐        │
│ ALLOW  │       YES
└────────┘        │
    ↑             ↓
    │        ┌────────┐
    │        │ ALLOW  │
    │        └────────┘
    │             ↑
┌───┴──────┐      │
│ Is user  │      │
│ shared   │      │
│ with?    │      │
└───┬──────┘      │
   YES            │
    │             │
    ↓             │
┌────────────┐    │
│ Check      │    │
│ permission │    │
│ level      │────┘
└────────────┘
    │
   NO
    ↓
┌────────┐
│ DENY   │
└────────┘
```

### Core Access Control Function

```javascript
/**
 * Main function to check if user can access a folder
 * @param {Object} user - Current user object with role
 * @param {Object} folder - Folder object to check access
 * @param {String} action - Action to perform (VIEW, UPLOAD, DELETE, etc.)
 * @returns {Boolean} - true if allowed, false if denied
 */
async function canAccessFolder(user, folder, action) {
  
  // ============================================
  // STEP 1: Check if it's a PERSONAL folder
  // ============================================
  if (folder.folderType === 'PERSONAL') {
    
    // Owner has full access
    if (folder.ownerId.toString() === user._id.toString()) {
      return true;
    }
    
    // Check if folder is shared with this user
    const access = await FolderAccess.findOne({
      folderId: folder._id,
      userId: user._id,
      isActive: true
    });
    
    if (access && !access.hasExpired()) {
      return access.canPerform(action);
    }
    
    // Super Admin CANNOT access personal folders (per requirement)
    return false;
  }
  
  // ============================================
  // STEP 2: It's an ORGANIZATION folder - Apply RBAC
  // ============================================
  if (folder.folderType === 'ORGANIZATION') {
    return canAccessOrganizationFolder(user, folder, action);
  }
  
  // Default: Deny
  return false;
}

/**
 * RBAC logic for organization folders
 */
async function canAccessOrganizationFolder(user, folder, action) {
  
  // Super Admin - Full access to all organization folders
  if (user.role === 'SUPER_ADMIN') {
    return true;
  }
  
  // Admin - Check department assignment
  if (user.role === 'ADMIN') {
    if (!user.assignedDepartments?.includes(folder.departmentId.toString())) {
      return false;
    }
    return roleCanPerformAction(user.role, action);
  }
  
  // Department Head - Check if it's their department
  if (user.role === 'DEPT_HEAD') {
    if (user.assignedDepartment?.toString() !== folder.departmentId.toString()) {
      return false;
    }
    return roleCanPerformAction(user.role, action);
  }
  
  // Folder Manager/User - Check folder assignment
  if (user.role === 'FOLDER_MANAGER' || user.role === 'FOLDER_USER') {
    const hasAccess = await userHasFolderAccess(user, folder);
    if (!hasAccess) {
      return false;
    }
    return roleCanPerformAction(user.role, action);
  }
  
  return false;
}

/**
 * Check if user has access to folder (including parent folders)
 */
async function userHasFolderAccess(user, folder) {
  // Check direct assignment
  const access = await FolderAccess.findOne({
    folderId: folder._id,
    userId: user._id,
    isActive: true,
    accessSource: 'ASSIGNED_RBAC'
  });
  
  if (access) return true;
  
  // Check parent folder recursively
  if (folder.parentId) {
    const parentFolder = await Folder.findById(folder.parentId);
    if (parentFolder) {
      return userHasFolderAccess(user, parentFolder);
    }
  }
  
  return false;
}

/**
 * Define what each role can do
 */
function roleCanPerformAction(role, action) {
  const rolePermissions = {
    SUPER_ADMIN: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'MANAGE_USERS', 'SHARE'],
    ADMIN: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'ASSIGN_FOLDER_MANAGER'],
    DEPT_HEAD: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_FOLDER', 'ASSIGN_FOLDER_MANAGER'],
    FOLDER_MANAGER: ['VIEW', 'UPLOAD', 'DELETE', 'CREATE_SUBFOLDER'],
    FOLDER_USER: ['VIEW', 'UPLOAD']
  };
  
  return rolePermissions[role]?.includes(action) || false;
}

module.exports = {
  canAccessFolder,
  canAccessOrganizationFolder,
  userHasFolderAccess,
  roleCanPerformAction
};
```

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
│  2. Create FolderShare record                   │
│     ↓                                           │
│  3. Create FolderAccess record for User B       │
│     ↓                                           │
│  4. User B can now access "My Docs"            │
│     ↓                                           │
│  5. User B's permissions based on share level   │
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
- Owner can share with anyone
- CO_OWNER can share with anyone
- Can share multiple times (multiple users)
- Can set different permission levels for different users

**❌ CANNOT Share:**
- EDITOR cannot share
- VIEWER cannot share
- FOLDER_MANAGER cannot share (they manage assigned folders only)
- Cannot share organization folders (only personal folders)

### Share Implementation Logic

```javascript
/**
 * Share a personal folder with another user
 */
async function shareFolder(folderId, sharedBy, sharedWith, permissionLevel, shareMessage = '') {
  // 1. Get the folder
  const folder = await Folder.findById(folderId);
  
  if (!folder) {
    throw new Error('Folder not found');
  }
  
  // 2. Verify it's a personal folder
  if (folder.folderType !== 'PERSONAL') {
    throw new Error('Only personal folders can be shared');
  }
  
  // 3. Verify the user has permission to share
  const canShare = await canUserShareFolder(sharedBy, folder);
  if (!canShare) {
    throw new Error('You do not have permission to share this folder');
  }
  
  // 4. Cannot share with yourself
  if (sharedBy.toString() === sharedWith.toString()) {
    throw new Error('Cannot share folder with yourself');
  }
  
  // 5. Check if already shared
  const existingShare = await FolderShare.findOne({
    folderId: folderId,
    sharedWith: sharedWith,
    isActive: true
  });
  
  if (existingShare) {
    // Update existing share
    existingShare.permissionLevel = permissionLevel;
    existingShare.shareMessage = shareMessage;
    await existingShare.save();
  } else {
    // Create new share
    await FolderShare.create({
      folderId,
      sharedBy,
      sharedWith,
      permissionLevel,
      shareMessage
    });
  }
  
  // 6. Grant access in FolderAccess table
  await FolderAccess.grantAccess(
    folderId,
    sharedWith,
    permissionLevel,
    sharedBy,
    'SHARED'
  );
  
  return { success: true, message: 'Folder shared successfully' };
}

/**
 * Check if user can share a folder
 */
async function canUserShareFolder(userId, folder) {
  // Owner can always share
  if (folder.ownerId.toString() === userId.toString()) {
    return true;
  }
  
  // Check if user has CO_OWNER access
  const access = await FolderAccess.findOne({
    folderId: folder._id,
    userId: userId,
    accessType: 'CO_OWNER',
    isActive: true
  });
  
  return access != null;
}

/**
 * Revoke folder share
 */
async function revokeShare(folderId, sharedWith, revokedBy) {
  // 1. Find the share
