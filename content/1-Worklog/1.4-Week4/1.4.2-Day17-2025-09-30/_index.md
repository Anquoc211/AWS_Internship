---
title: "Day 17 - S3 Advanced Features"
weight: 2
chapter: false
pre: "<b> 1.4.2. </b>"
---

**Date:** 2025-09-30 (Thứ Ba)  
**Status:** "Done"  

---

# **Lecture Notes**

## Amazon S3 Static Website Hosting

Host static websites (HTML, CSS, JS, images) directly from S3.

### Key Capabilities

- **Simple setup:** A few steps to enable static website hosting on a bucket.
- **Low cost:** Pay standard S3 storage and data transfer; no separate web server charges.
- **Elastic scaling:** Automatically handles traffic spikes.
- **CDN integration:** Easily front with Amazon CloudFront for global performance.

**Static Website Configuration:**

```json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "error.html"
  }
}
```

---

## Cross-Origin Resource Sharing (CORS)

CORS allows web resources (fonts, JavaScript, etc.) on one domain to request resources from another domain.

### Configuring CORS on S3

- **Define policies:** Specify which origins are permitted to access a bucket's content.
- **Control methods:** Allow specific HTTP methods (GET, PUT, POST, etc.).
- **Security posture:** Prevent unauthorized cross-origin access.

**CORS Configuration Example:**

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedOrigins": ["https://example.com"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

---

## Performance and Object Key Design

Object key naming can significantly affect S3 performance:

- **Randomized prefixes:** Distribute keys across partitions for higher parallelism.
- **Avoid sequential prefixes:** Don't use monotonically increasing prefixes (e.g., timestamps) for high-throughput workloads.
- **Parallel access:** Structure keys to enable concurrent reads/writes.

**Key Design Best Practices:**

```
❌ Bad:  2025-09-30-file1.jpg, 2025-09-30-file2.jpg
Good: a1b2/2025-09-30-file1.jpg, c3d4/2025-09-30-file2.jpg
```

---

## S3 Glacier – Long-Term Archival

S3 Glacier classes are optimized for ultra–low-cost long-term storage.

### Retrieval Options

- **Expedited / Fast:** Minutes; highest cost.
- **Standard:** 3–5 hours; balanced cost.
- **Bulk:** 5–12 hours; lowest cost for large restores.

### Glacier Deep Archive

The lowest-cost class for multi-year retention, with ~12-hour retrieval times.

---

# **Hands-On Labs**

## Lab 57 – Amazon S3 & CloudFront (Part 2)

5. Configure Public Objects → *57-5*  
6. Test Website → *57-6*  
7. Block All Public Access → *57-7.1*  
8. Configure CloudFront → *57-7.2*  
9. Test CloudFront → *57-7.3*  
10. Bucket Versioning → *57-8*
