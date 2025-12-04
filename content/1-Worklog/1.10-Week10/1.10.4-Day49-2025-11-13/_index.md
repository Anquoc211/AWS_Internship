---
title: "Day 49 - Reader Interface & Search Implementation"
weight: 4
chapter: false
pre: "<b> 1.10.4. </b>"
---

**Date:** 2025-11-13 (Thursday)  
**Status:** "Done"  

---

# **Lecture Notes**

## Book Reading Experience

### CloudFront Signed URLs for Content Delivery

**Security Strategy:**
- Books stored in S3 `public/books/` but **not publicly accessible**
- CloudFront configured with **Origin Access Control (OAC)**
- S3 bucket policy allows only CloudFront to access objects
- Lambda generates **signed GET URLs** with short TTL (15 minutes)
- Users access content through signed URLs only

**Signed URL Flow:**
```
User clicks "Read" → Frontend calls /books/{id}/read API → 
Lambda validates user authentication → Lambda checks book status (APPROVED) →
Lambda generates CloudFront signed URL → Returns URL to frontend →
Frontend loads PDF/ePub via signed URL
```

### PDF/ePub Rendering

**Implementation Options:**
- **react-pdf**: Render PDF in browser with canvas
- **PDF.js**: Mozilla's PDF viewer (more features)
- **epub.js**: ePub reader with customization
- **iframe**: Simple but less control

**Reader Features:**
- Page navigation (previous/next)
- Zoom in/out
- Fullscreen mode
- Bookmark support (store in DynamoDB)
- Reading progress tracking

## Search & Discovery

### DynamoDB GSI-Based Search

**Search Strategy:**
- Create **Global Secondary Indexes (GSI)** for searchable fields
- GSI1: `PK=TITLE#{normalizedTitle}, SK=BOOK#{bookId}`
- GSI2: `PK=AUTHOR#{normalizedAuthor}, SK=BOOK#{bookId}`
- Query GSI instead of expensive Scan operations

**Search Flow:**
```
User types search query → Frontend calls /search API with query params →
Lambda determines which GSI(s) to query →
Query GSI1 (title) and/or GSI2 (author) →
Intersection of results if multiple params →
BatchGetItem to fetch full book metadata →
Return results to frontend
```

### Frontend Search Implementation

**Search Components:**
- Search bar with autocomplete
- Filter panel (by author, genre, upload date)
- Sort options (newest, oldest, popular)
- Results grid/list view toggle
- Pagination or infinite scroll

**Search Optimization:**
- Debounce search input (wait 300ms after typing)
- Cache recent search results (5 minutes)
- Show loading skeleton during search
- Display "No results" with suggestions

## Key Insights

- Signed URLs essential for content security - never expose S3 directly
- Short TTL on signed URLs prevents sharing/abuse
- GSI queries much cheaper and faster than Scan
- Normalized fields (lowercase, no special chars) improve search accuracy
- Client-side caching reduces API calls and improves UX

---

# **Tasks Completed**

1. **Book Detail Page**
   - Created book detail view with metadata display
   - Implemented cover image loading from S3
   - Added "Read Now" button (only for APPROVED books)
   - Built related books section

2. **PDF Reader Integration**
   - Integrated react-pdf library for PDF rendering
   - Created reader component with navigation controls
   - Implemented zoom controls (fit-width, fit-page, custom zoom)
   - Added fullscreen mode toggle

3. **ePub Reader Integration**
   - Integrated epub.js library for ePub rendering
   - Created ePub reader with page turn animations
   - Implemented text selection and note-taking
   - Added adjustable font size and theme

4. **Signed URL Implementation**
   - Created `getReadUrl` API client method
   - Implemented signed URL fetching on "Read" click
   - Added URL expiration handling (re-fetch on expire)
   - Built loading state while fetching URL

5. **Search Functionality**
   - Created search bar component with autocomplete
   - Implemented search API integration (`/search` endpoint)
   - Built search results page with grid/list view
   - Added filter sidebar (author, genre, status)

6. **Search Optimization**
   - Implemented debounced search input (300ms delay)
   - Added client-side result caching with React Query
   - Created search history (stored in localStorage)
   - Built "Recent Searches" dropdown

7. **Discovery Features**
   - Created "Browse Books" page with all approved books
   - Implemented category/genre filtering
   - Added sort options (newest, title A-Z, author A-Z)
   - Built "Recommended for You" section (future: ML-based)

---

# **Challenges & Solutions**

**Challenge:** PDF rendering performance issues with large files  
**Solution:** Implemented lazy loading of pages (render visible pages only) and used Web Workers for PDF parsing

**Challenge:** Signed URLs expiring during reading session  
**Solution:** Implemented auto-refresh mechanism that requests new URL 2 minutes before expiration

**Challenge:** Search returning too many results causing slow UI  
**Solution:** Implemented server-side pagination with cursor-based navigation and virtual scrolling

**Challenge:** ePub text selection interfering with page turns  
**Solution:** Added touch/click handlers with gesture detection to differentiate between selection and navigation
