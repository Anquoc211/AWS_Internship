---
title: "Day 48 - Admin Panel & Approval Workflow"
weight: 3
chapter: false
pre: "<b> 1.10.3. </b>"
---

**Date:** 2025-11-12 (Wednesday)  
**Status:** "Done"  

---

# **Lecture Notes**

## Admin Panel Architecture

### Role-Based Access Control (RBAC)

**Cognito Groups Implementation:**
- Admin users assigned to `Admins` group in Cognito User Pool
- JWT token contains `cognito:groups: ["Admins"]` claim
- Frontend checks user groups before rendering admin routes
- API Gateway validates JWT, Lambda validates group membership

**Authorization Flow:**
```
Admin Login → Cognito returns JWT with groups → 
Frontend checks groups → Show/Hide admin navigation →
Admin action → API validates JWT + groups → Lambda executes
```

### Admin Dashboard Features

**Core Functionality:**
1. **Pending Books Review**: List all books with status `PENDING`
2. **Approval/Rejection Actions**: Approve or reject with reason
3. **Book Management**: View all books, search, filter by status
4. **User Management**: View uploaders, activity logs
5. **Analytics**: Upload statistics, approval rates, storage usage

**Status Workflow:**
- `PENDING` → Initial upload status
- `VALIDATING` → Server-side MIME validation in progress
- `APPROVED` → Admin approved, book publicly accessible
- `REJECTED` → Admin rejected with reason
- `TAKEDOWN` → Content removed due to copyright/violation

## Approval Workflow Implementation

### Backend Flow (Lambda)

**approveBook Lambda:**
```
1. Validate JWT and check Admins group
2. Verify book exists and status is PENDING
3. Copy file from uploads/ to public/books/
4. Update DynamoDB: status=APPROVED, approverID, approvalTimestamp
5. Delete original file from uploads/
6. Return success response
```

**rejectBook Lambda:**
```
1. Validate JWT and check Admins group
2. Update DynamoDB: status=REJECTED, reason, rejectionTimestamp
3. Optional: Move file to quarantine/ instead of deletion
4. Send notification to uploader (future enhancement)
```

### Frontend Admin Interface

**Pending Books List:**
- Display: thumbnail, title, author, uploader, upload date
- Actions: Preview (download temp), Approve, Reject
- Filters: by date, by uploader, by file type
- Pagination for large datasets

**Approval Modal:**
- Confirm action with book details
- Optional: add approval notes
- Loading state during processing
- Success/error notifications

**Rejection Modal:**
- Required rejection reason selection/textarea
- Predefined reasons: copyright, inappropriate, quality, other
- Custom message field
- Confirmation before submission

## Key Insights

- Admin panel is high-privilege area - strict access control essential
- Audit logging critical for accountability (who approved/rejected what and when)
- Soft delete (quarantine) better than hard delete for legal compliance
- Real-time status updates improve admin workflow efficiency
- Clear rejection reasons help uploaders improve future submissions

---

# **Tasks Completed**

1. **Admin Route Protection**
   - Created admin-only route wrapper checking Cognito groups
   - Implemented unauthorized access handling (redirect to dashboard)
   - Added admin navigation menu (visible only to admins)
   - Built "Access Denied" page for non-admin users

2. **Pending Books Interface**
   - Created pending books list component with DataTable
   - Implemented book preview modal (display metadata and cover)
   - Added bulk selection for batch operations
   - Built status filter (PENDING, VALIDATING, ALL)

3. **Approval System**
   - Created approval confirmation modal
   - Implemented `approveBook` API integration
   - Added optimistic UI updates (instant status change)
   - Built success/error toast notifications

4. **Rejection System**
   - Created rejection modal with reason selection
   - Built rejection reason dropdown (predefined + custom)
   - Implemented `rejectBook` API integration
   - Added rejection history view for uploaders

5. **Admin Dashboard**
   - Created dashboard overview with statistics cards
   - Built charts: uploads over time, approval rates, status distribution
   - Implemented recent activity feed
   - Added quick action buttons (pending count badge)

6. **Audit Logging Display**
   - Created audit log viewer showing all actions
   - Implemented filtering by action type, date range, admin
   - Added export audit logs to CSV functionality
   - Built detailed action view with metadata

---

# **Challenges & Solutions**

**Challenge:** Real-time status updates without polling  
**Solution:** Implemented WebSocket connection for admin panel to receive instant status changes from backend

**Challenge:** Large list of pending books causing performance issues  
**Solution:** Implemented virtual scrolling and server-side pagination with cursor-based navigation

**Challenge:** Accidental approvals/rejections  
**Solution:** Added confirmation modals with 2-second countdown before action enabled

**Challenge:** Admin actions not reflected immediately  
**Solution:** Implemented optimistic UI updates with rollback on error, plus cache invalidation
