---
title: "Day 47 - Book Upload Flow & Presigned URLs"
weight: 2
chapter: false
pre: "<b> 1.10.2. </b>"
---

**Date:** 2025-11-11 (Tuesday)  
**Status:** "Done"  

---

# **Lecture Notes**

## Upload Flow Architecture

### S3 Presigned URL Strategy

**Why Presigned URLs?**
- Direct upload to S3 bypasses API Gateway payload limits (10MB)
- Reduces Lambda execution time and cost
- Better user experience with progress tracking
- Serverless architecture scales automatically

**Upload Flow:**
```
User → Request Upload URL (API) → Lambda generates Presigned PUT URL → 
User uploads directly to S3 → S3 Event Notification → Lambda updates DynamoDB
```

### Security Considerations

**Presigned URL Configuration:**
- Short expiration time (15 minutes for upload)
- Restrict file size via `content-length-range` condition
- Limit to specific MIME types (application/pdf, application/epub+zip)
- Upload to temporary `uploads/` prefix, not public folder

**Validation Strategy:**
- Client-side: check file size and type before requesting URL
- Server-side: Lambda validates metadata before generating URL
- S3 Event: post-upload Lambda validates actual file MIME type using magic bytes
- Status tracking: PENDING → VALIDATING → APPROVED/REJECTED

## Frontend Upload Implementation

### Multi-Step Upload Process

**Step 1: File Selection & Validation**
- Drag-and-drop or file picker interface
- Client-side validation (size ≤50MB, type PDF/ePub)
- Preview file metadata (name, size, type)

**Step 2: Metadata Collection**
- Form fields: title, author, description, cover image
- Genre/category selection
- Optional: ISBN, publication year, language

**Step 3: Request Upload URL**
- POST to `/upload` API endpoint with metadata
- Receive presigned URL and book ID
- Store book ID for status tracking

**Step 4: Direct S3 Upload**
- PUT request to presigned URL with file content
- Track upload progress with XMLHttpRequest or fetch
- Show progress bar and estimated time remaining

**Step 5: Status Monitoring**
- Poll `/books/{id}/status` endpoint
- Display current status (PENDING, VALIDATING, APPROVED, REJECTED)
- Show rejection reason if applicable

## Key Insights

- Presigned URLs are time-bound credentials - handle expiration gracefully
- Always validate on both client and server to prevent malicious uploads
- Progress tracking significantly improves user experience for large files
- S3 Event Notifications enable async validation without polling
- Clear status messages help users understand upload workflow

---

# **Tasks Completed**

1. **Upload UI Components**
   - Created file upload component with drag-and-drop
   - Built metadata form with validation (React Hook Form + Zod)
   - Implemented upload progress indicator with percentage
   - Added file type and size validation

2. **API Integration**
   - Created API client methods for upload flow
   - Implemented `createUploadUrl` API call with metadata
   - Built direct S3 upload with progress tracking
   - Added error handling for network failures

3. **Status Tracking**
   - Created status polling mechanism with intervals
   - Built status badge component (PENDING, APPROVED, REJECTED)
   - Implemented real-time status updates
   - Added notification system for status changes

4. **User Experience**
   - Designed upload wizard with multi-step form
   - Added file preview before upload
   - Implemented success/error notifications
   - Created "My Uploads" page showing all user uploads

5. **Error Handling**
   - Handled presigned URL expiration
   - Implemented retry logic for failed uploads
   - Added validation error messages
   - Created fallback UI for upload failures

---

# **Challenges & Solutions**

**Challenge:** Large file uploads timing out  
**Solution:** Implemented chunked upload with retry logic for failed chunks

**Challenge:** Presigned URL expiring during slow uploads  
**Solution:** Request new URL if upload takes longer than 10 minutes

**Challenge:** User closing browser during upload  
**Solution:** Added localStorage tracking to resume uploads and show warning before page unload

**Challenge:** Progress tracking not accurate  
**Solution:** Used XMLHttpRequest upload.onprogress event for accurate byte-level tracking
