---
title: "Module 5: Search & Discovery"
date: "2025-01-15T13:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview

In this module, you'll implement search functionality for the Online Library using DynamoDB Global Secondary Indexes (GSI). You'll build efficient queries for title and author search, implement pagination, and create a user-friendly search interface with filters.

**Duration:** ~60 minutes

**Services Used:**
- Amazon DynamoDB (GSI for search)
- AWS Lambda (search logic)
- Amazon API Gateway (search endpoint)

---

## What You'll Learn

- Design DynamoDB GSI for efficient search
- Query GSI instead of expensive Scan operations
- Implement pagination with cursor-based navigation
- Build search UI with autocomplete
- Add filters and sorting capabilities

---

## Architecture for This Module

```
User → Search Form → API Gateway → Lambda (searchBooks)
                                    ↓
                              Query GSI1 (Title) or GSI2 (Author)
                                    ↓
                              Return filtered & paginated results
                                    ↓
User ← Search Results ← Response
```

---

## Step 1: Design DynamoDB GSI for Search

### 1.1 GSI Schema Design

**GSI1 - Title Search:**
- PK: `TITLE#{normalizedTitle}`
- SK: `BOOK#{bookId}`
- Allows querying books by title prefix

**GSI2 - Author Search:**
- PK: `AUTHOR#{normalizedAuthor}`
- SK: `BOOK#{bookId}`
- Allows querying books by author name

**Normalization strategy:**
- Lowercase all text
- Remove special characters
- Trim whitespace
- Store original values in attributes

### 1.2 Update CDK Stack

```typescript
// filepath: lib/online-library-cdk-stack.ts
// ...existing code...

// Add GSI for Title search
booksTable.addGlobalSecondaryIndex({
  indexName: 'TitleIndex',
  partitionKey: {
    name: 'titleNormalized',
    type: dynamodb.AttributeType.STRING,
  },
  sortKey: {
    name: 'bookId',
    type: dynamodb.AttributeType.STRING,
  },
  projectionType: dynamodb.ProjectionType.ALL,
});

// Add GSI for Author search
booksTable.addGlobalSecondaryIndex({
  indexName: 'AuthorIndex',
  partitionKey: {
    name: 'authorNormalized',
    type: dynamodb.AttributeType.STRING,
  },
  sortKey: {
    name: 'bookId',
    type: dynamodb.AttributeType.STRING,
  },
  projectionType: dynamodb.ProjectionType.ALL,
});
```

---

## Step 2: Create Search Lambda Function

### 2.1 Create searchBooks Lambda

Create `lambda/searchBooks/index.py`:

```python
# filepath: lambda/searchBooks/index.py
import json
import boto3
from boto3.dynamodb.conditions import Key, Attr
import os

dynamodb = boto3.resource('dynamodb')

BOOKS_TABLE = os.environ['BOOKS_TABLE']
table = dynamodb.Table(BOOKS_TABLE)

def normalize_text(text):
    """Normalize text for searching"""
    if not text:
        return ''
    return text.lower().strip()

def lambda_handler(event, context):
    """
    Search books by title and/or author
    """
    try:
        # Parse query parameters
        params = event.get('queryStringParameters', {}) or {}
        
        title_query = params.get('title', '').strip()
        author_query = params.get('author', '').strip()
        limit = int(params.get('limit', 20))
        next_token = params.get('nextToken')
        
        # Validate at least one search parameter
        if not title_query and not author_query:
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': 'Please provide title or author search term'
                })
            }
        
        results = []
        
        # Search by title
        if title_query and not author_query:
            title_normalized = normalize_text(title_query)
            
            response = table.query(
                IndexName='TitleIndex',
                KeyConditionExpression=Key('titleNormalized').begins_with(title_normalized),
                FilterExpression=Attr('status').eq('APPROVED'),
                Limit=limit,
                ExclusiveStartKey=json.loads(next_token) if next_token else None
            )
            results = response['Items']
            next_key = response.get('LastEvaluatedKey')
        
        # Search by author
        elif author_query and not title_query:
            author_normalized = normalize_text(author_query)
            
            response = table.query(
                IndexName='AuthorIndex',
                KeyConditionExpression=Key('authorNormalized').begins_with(author_normalized),
                FilterExpression=Attr('status').eq('APPROVED'),
                Limit=limit,
                ExclusiveStartKey=json.loads(next_token) if next_token else None
            )
            results = response['Items']
            next_key = response.get('LastEvaluatedKey')
        
        # Search by both title AND author
        else:
            # Query both indexes and find intersection
            title_normalized = normalize_text(title_query)
            author_normalized = normalize_text(author_query)
            
            # Get books matching title
            title_response = table.query(
                IndexName='TitleIndex',
                KeyConditionExpression=Key('titleNormalized').begins_with(title_normalized),
                FilterExpression=Attr('status').eq('APPROVED')
            )
            title_books = {book['bookId']: book for book in title_response['Items']}
            
            # Get books matching author
            author_response = table.query(
                IndexName='AuthorIndex',
                KeyConditionExpression=Key('authorNormalized').begins_with(author_normalized),
                FilterExpression=Attr('status').eq('APPROVED')
            )
            
            # Find intersection
            for book in author_response['Items']:
                if book['bookId'] in title_books:
                    results.append(book)
            
            # Apply pagination manually
            start = int(json.loads(next_token)['offset']) if next_token else 0
            end = start + limit
            results = results[start:end]
            next_key = {'offset': end} if end < len(results) else None
        
        # Format results
        formatted_results = []
        for book in results:
            formatted_results.append({
                'bookId': book['bookId'],
                'title': book['title'],
                'author': book['author'],
                'description': book.get('description', ''),
                'fileType': book['fileType'],
                'uploadTimestamp': book['uploadTimestamp'],
            })
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
            },
            'body': json.dumps({
                'results': formatted_results,
                'count': len(formatted_results),
                'nextToken': json.dumps(next_key) if next_key else None,
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': 'Search failed'
            })
        }
```

### 2.2 Add Lambda to CDK

```typescript
// filepath: lib/online-library-cdk-stack.ts
// ...existing code...

const searchBooksFunction = new lambda.Function(this, 'SearchBooks', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.lambda_handler',
  code: lambda.Code.fromAsset('lambda/searchBooks'),
  environment: {
    BOOKS_TABLE: booksTable.tableName,
  },
  timeout: cdk.Duration.seconds(30),
});

// Grant permissions
booksTable.grantReadData(searchBooksFunction);

// Add API route
httpApi.addRoutes({
  path: '/search',
  methods: [apigateway.HttpMethod.GET],
  integration: new integrations.HttpLambdaIntegration(
    'SearchBooksIntegration',
    searchBooksFunction
  ),
  authorizer: jwtAuthorizer,
});
```

---

## Step 3: Build Search Interface

### 3.1 Create Search API Client

Create `src/lib/api/search.ts`:

```typescript
// filepath: src/lib/api/search.ts
import { fetchAuthSession } from 'aws-amplify/auth';

export interface SearchResult {
  bookId: string;
  title: string;
  author: string;
  description?: string;
  fileType: string;
  uploadTimestamp: string;
}

export interface SearchResponse {
  results: SearchResult[];
  count: number;
  nextToken?: string;
}

export async function searchBooks(
  title?: string,
  author?: string,
  nextToken?: string
): Promise<SearchResponse> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const params = new URLSearchParams();
  if (title) params.append('title', title);
  if (author) params.append('author', author);
  if (nextToken) params.append('nextToken', nextToken);

  const response = await fetch(
    `${process.env.NEXT_PUBLIC_API_URL}/search?${params}`,
    {
      headers: {
        'Authorization': `Bearer ${idToken}`,
      },
    }
  );

  if (!response.ok) {
    throw new Error('Search failed');
  }

  return response.json();
}
```

### 3.2 Create Search Component

Create `src/components/search/SearchBar.tsx`:

```typescript
// filepath: src/components/search/SearchBar.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';

export function SearchBar() {
  const router = useRouter();
  const searchParams = useSearchParams();
  
  const [titleQuery, setTitleQuery] = useState(searchParams.get('title') || '');
  const [authorQuery, setAuthorQuery] = useState(searchParams.get('author') || '');

  const debouncedSearch = useDebouncedCallback((title: string, author: string) => {
    const params = new URLSearchParams();
    if (title) params.set('title', title);
    if (author) params.set('author', author);
    
    router.push(`/search?${params.toString()}`);
  }, 300);

  const handleTitleChange = (value: string) => {
    setTitleQuery(value);
    debouncedSearch(value, authorQuery);
  };

  const handleAuthorChange = (value: string) => {
    setAuthorQuery(value);
    debouncedSearch(titleQuery, value);
  };

  return (
    <div className="flex gap-4 p-4 bg-white rounded-lg shadow">
      <div className="flex-1">
        <label className="block text-sm font-medium mb-2">
          Search by Title
        </label>
        <input
          type="text"
          value={titleQuery}
          onChange={(e) => handleTitleChange(e.target.value)}
          placeholder="Enter book title..."
          className="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500"
        />
      </div>

      <div className="flex-1">
        <label className="block text-sm font-medium mb-2">
          Search by Author
        </label>
        <input
          type="text"
          value={authorQuery}
          onChange={(e) => handleAuthorChange(e.target.value)}
          placeholder="Enter author name..."
          className="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500"
        />
      </div>
    </div>
  );
}
```

### 3.3 Create Search Results Page

Create `src/app/search/page.tsx`:

```typescript
// filepath: src/app/search/page.tsx
'use client';

import { useState, useEffect } from 'react';
import { useSearchParams } from 'next/navigation';
import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import { SearchBar } from '@/components/search/SearchBar';
import { searchBooks, SearchResult } from '@/lib/api/search';
import Link from 'next/link';

export default function SearchPage() {
  const searchParams = useSearchParams();
  const [results, setResults] = useState<SearchResult[]>([]);
  const [loading, setLoading] = useState(false);
  const [nextToken, setNextToken] = useState<string | null>(null);

  useEffect(() => {
    const title = searchParams.get('title');
    const author = searchParams.get('author');

    if (title || author) {
      performSearch(title || undefined, author || undefined);
    }
  }, [searchParams]);

  const performSearch = async (
    title?: string,
    author?: string,
    token?: string
  ) => {
    setLoading(true);
    try {
      const response = await searchBooks(title, author, token);
      setResults(token ? [...results, ...response.results] : response.results);
      setNextToken(response.nextToken || null);
    } catch (error) {
      console.error('Search error:', error);
    } finally {
      setLoading(false);
    }
  };

  const loadMore = () => {
    const title = searchParams.get('title');
    const author = searchParams.get('author');
    if (nextToken) {
      performSearch(title || undefined, author || undefined, nextToken);
    }
  };

  return (
    <ProtectedRoute>
      <div className="max-w-6xl mx-auto p-6">
        <h1 className="text-3xl font-bold mb-6">Search Books</h1>

        <SearchBar />

        <div className="mt-8">
          {loading && results.length === 0 && (
            <div className="text-center py-8">Searching...</div>
          )}

          {!loading && results.length === 0 && (
            <div className="text-center py-8 text-gray-600">
              No results found. Try different search terms.
            </div>
          )}

          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {results.map((book) => (
              <Link
                key={book.bookId}
                href={`/read/${book.bookId}`}
                className="border rounded-lg p-4 hover:shadow-lg transition"
              >
                <h3 className="font-semibold text-lg mb-2">{book.title}</h3>
                <p className="text-gray-600 mb-2">by {book.author}</p>
                {book.description && (
                  <p className="text-sm text-gray-500 line-clamp-3">
                    {book.description}
                  </p>
                )}
                <div className="mt-4 text-sm text-blue-600">Read →</div>
              </Link>
            ))}
          </div>

          {nextToken && (
            <div className="text-center mt-8">
              <button
                onClick={loadMore}
                disabled={loading}
                className="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:bg-gray-300"
              >
                {loading ? 'Loading...' : 'Load More'}
              </button>
            </div>
          )}
        </div>
      </div>
    </ProtectedRoute>
  );
}
```

---

## Verification & Testing

### Checklist

- DynamoDB GSI created for title and author
- Search Lambda queries GSI efficiently
- Search by title returns correct results
- Search by author returns correct results
- Combined search (title AND author) works
- Pagination with nextToken functional
- Search UI debounces input
- Load more button loads additional results

### Common Issues & Solutions

**Issue:** Search returns too many results  
**Solution:** Implement more strict begins_with matching or add additional filters

**Issue:** Slow search performance  
**Solution:** Ensure GSI is properly configured and using Query not Scan

**Issue:** Pagination not working correctly  
**Solution:** Verify LastEvaluatedKey is properly encoded/decoded

---

## Clean Up

```bash
cdk destroy
```

---

## Next Steps

Congratulations! You've completed Module 5. You now have:
- Efficient search using DynamoDB GSI
- Search by title and author with prefix matching
- Pagination for large result sets
- User-friendly search interface with debouncing

**Continue to:** [Module 6: Deployment & Operations](../5.6-module6/)

---

## Additional Resources

- [DynamoDB GSI Documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
- [DynamoDB Query Operations](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html)
- [Pagination Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)