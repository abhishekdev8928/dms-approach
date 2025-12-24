# 🏢 DMS Hybrid System
## Document Management System with Personal Drive, Organization Drive & Secure Sharing

---

## 📌 Overview

This repository contains the **complete design documentation** for a **Hybrid Document Management System (DMS)** supporting:

- **Personal Drive (My Drive)** – Private workspace owned by each user
- **Organization Drive** – Department-based shared workspace with RBAC
- **Shared with Me** – Controlled access to folders shared by others

The system is designed with **privacy-first principles**, **enterprise-grade access control**, and **full auditability**.

---

## 📋 Table of Contents

1. System Overview  
2. Core Concepts  
3. Architecture Design  
4. Database Models  
5. Access Control Logic  
6. Folder Sharing System  
7. RBAC for Organization Drive  
8. API Design Strategy  
9. Implementation Roadmap  
10. Security & Privacy Rules  
11. Real-World Examples  
12. Edge Cases & Testing Scenarios  
13. Summary  

---

## 🎯 System Overview

### Components

- **My Drive**
  - Private by default
  - Owner-controlled sharing
  - Super Admin has no access

- **Organization Drive**
  - Department-based
  - Role-Based Access Control (RBAC)
  - Managed by Admins and Department Heads

- **Shared with Me**
  - Read-only listing of folders shared with the user
  - Permission-driven visibility

### Design Principles

- Privacy-first for personal data
- Explicit permissions over implicit access
- Separation of storage and access control
- Full audit trail for compliance

---

## 🧩 Core Concepts

### Folder Types

**PERSONAL**
- Created in My Drive
- Owned by creator
- Shareable
- Not visible to admins

**ORGANIZATION**
- Created in Organization Drive
- Department-owned
- Access via RBAC
- Not shareable like personal folders

---

## 🏗️ Architecture Design

### Three-Layer Architecture

1. **Storage Layer**
   - Folders
   - Files

2. **Access Control Layer**
   - Folder access rules
   - Sharing metadata

3. **Business Logic Layer**
   - RBAC enforcement
   - Sharing validation
   - Inheritance logic

---

## 💾 Database Models

### Folder
- Stores both personal and organization folders
- Supports hierarchy
- Soft delete enabled

### FolderAccess
- Central access-control table
- One active access per user per folder
- Supports inheritance and expiry

### FolderShare
- Tracks personal folder sharing
- Stores permission level and metadata

### File
- Metadata-only
- Inherits folder permissions
- Soft delete supported

---

## 🔐 Access Control Logic

### Personal Folders
- Owner always has full access
- Others only via explicit share
- Super Admin explicitly denied

### Organization Folders
- Access derived from role + department
- Explicit folder assignments override inheritance

---

## 🤝 Folder Sharing System

- Only PERSONAL folders can be shared
- Only OWNER or CO_OWNER can share
- Permissions: VIEWER, EDITOR, CO_OWNER
- Revocation immediately removes access

---

## 🏢 RBAC for Organization Drive

### Roles

- SUPER_ADMIN
- ADMIN
- DEPT_HEAD
- FOLDER_MANAGER
- FOLDER_USER

### Key Rules

- Department-based isolation
- Downward permission inheritance
- No upward inheritance
- Explicit assignment overrides role-based access

---

## 🔌 API Design Strategy

### Core Areas

- Folder Management
- File Management
- Sharing
- Access Control
- Drive Views

### API Versioning

- `/api/v1/`
- Stateless REST APIs
- JWT-secured

---

## 🗺️ Implementation Roadmap

1. Foundation & Models
2. Personal Drive
3. Organization Drive (RBAC)
4. Drive Views
5. File Management
6. Security & Auditing
7. Testing & Optimization

---

## 🔒 Security & Privacy Rules

- Super Admin cannot access personal folders
- All actions audited
- Encryption at rest and in transit
- Soft delete with retention policy

---

## 💡 Real-World Use Cases

- Marketing collaboration
- Engineering documentation
- HR confidential records
- Cross-department projects

---

## 🧪 Edge Cases & Testing Scenarios

- Permission inheritance conflicts
- Share chains
- Role demotion
- Circular folder prevention
- Concurrent permission changes

---

## 📝 Summary

This DMS Hybrid System delivers:

- ✅ Private personal workspace
- ✅ Secure organization-wide collaboration
- ✅ Granular sharing with full control
- ✅ Strong privacy guarantees
- ✅ Scalable, enterprise-ready design

**End of Documentation**
