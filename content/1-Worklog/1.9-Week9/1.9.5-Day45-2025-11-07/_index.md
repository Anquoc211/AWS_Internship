---
title: "Day 45 - NLP Integration & Final Project"
weight: 5
chapter: false
pre: "<b> 1.9.5. </b>"
---

**Date:** 2025-11-07 (Friday)  
**Status:** "Done"  

---

# **Lecture Notes**

## Week 9 Recap

### Concepts Covered This Week

- **Day 41**: NLP fundamentals revision - tokenization, preprocessing pipelines
- **Day 42**: Advanced text classification - ensemble methods, imbalanced data
- **Day 43**: Production sentiment analysis - aspect-based, emotion detection, APIs
- **Day 44**: Advanced NER - custom training, entity linking, relation extraction
- **Day 45**: Integration & complete NLP systems

### Key Learnings

- Building production-ready NLP systems requires more than good models
- Integration of multiple NLP components creates powerful applications
- Monitoring, evaluation, and continuous improvement are essential
- Domain adaptation crucial for real-world performance

## Integrating NLP Components

### Multi-Component Pipeline

```
Text Input → Preprocessing → Classification → NER → Sentiment → Output
```

**Benefits of Integration:**
- Single API for multiple NLP tasks
- Shared preprocessing reduces computation
- Consistent error handling across components
- Easier maintenance and updates

### Design Patterns

**Pipeline Pattern**
- Sequential processing of text through multiple stages
- Each stage can be independently developed and tested
- Easy to add/remove stages

**Microservices Pattern**
- Each NLP task as separate service
- Scalable and independently deployable
- Communication via REST API or message queues

**Monolithic Pattern**
- All NLP tasks in single application
- Simpler deployment
- Suitable for smaller applications

## Building Complete NLP Applications

### Requirements Analysis

**Functional Requirements:**
- What NLP tasks are needed?
- What accuracy is acceptable?
- Real-time or batch processing?
- Input/output formats?

**Non-Functional Requirements:**
- Performance (latency, throughput)
- Scalability (concurrent users, data volume)
- Reliability (uptime, error rates)
- Security (data privacy, authentication)

### System Architecture

**Layers:**
1. **API Layer**: REST/GraphQL endpoints, request validation
2. **Business Logic Layer**: NLP pipeline orchestration
3. **Processing Layer**: Individual NLP components
4. **Data Layer**: Model storage, caching, logging

## Best Practices

### Code Organization

- Separate concerns: preprocessing, models, API, utilities
- Configuration management (environment variables, config files)
- Dependency injection for flexibility
- Clear naming conventions

### Testing Strategy

- Unit tests for individual components
- Integration tests for pipelines
- End-to-end tests for complete workflows
- Performance/load testing

### Deployment

- Containerization (Docker) for consistency
- CI/CD pipelines for automated deployment
- Blue-green or canary deployments for safety
- Health checks and monitoring

## Key Insights

- Start simple, iterate based on real usage
- Log everything for debugging and improvement
- Plan for failure - graceful degradation
- User feedback loop essential for improvement
- Documentation as important as code

---

# **Hands-On Labs**

## Lab 1: Multi-Component NLP Pipeline

```python
import spacy
from textblob import TextBlob
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
import pickle

class ComprehensiveNLPPipeline:
    """Integrated NLP pipeline with multiple components"""
    
    def __init__(self):
        # Load models
        self.nlp = spacy.load("en_core_web_sm")
        
        # Initialize components
        self.vectorizer = TfidfVectorizer(max_features=100)
        self.classifier = MultinomialNB()
        
        # Train simple classifier (in production, load pre-trained)
        sample_texts = ["tech product review", "political news article", "sports game summary"] * 10
        sample_labels = ["technology", "politics", "sports"] * 10
        X = self.vectorizer.fit_transform(sample_texts)
        self.classifier.fit(X, sample_labels)
    
    def preprocess(self, text):
        """Clean and normalize text"""
        # Lowercase
        text = text.lower()
        # Remove extra whitespace
        text = ' '.join(text.split())
        return text
    
    def extract_entities(self, text):
        """Extract named entities"""
        doc = self.nlp(text)
        entities = [
            {
                'text': ent.text,
                'label': ent.label_,
                'start': ent.start_char,
                'end': ent.end_char
            }
            for ent in doc.ents
        ]
        return entities
    
    def classify_text(self, text):
        """Classify text into category"""
        X = self.vectorizer.transform([text])
        category = self.classifier.predict(X)[0]
        probabilities = self.classifier.predict_proba(X)[0]
        confidence = max(probabilities)
        return category, confidence
    
    def analyze_sentiment(self, text):
        """Analyze sentiment"""
        blob = TextBlob(text)
        polarity = blob.sentiment.polarity
        
        if polarity > 0.1:
            sentiment = "positive"
        elif polarity < -0.1:
            sentiment = "negative"
        else:
            sentiment = "neutral"
        
        return sentiment, polarity
    
    def extract_keywords(self, text, top_n=5):
        """Extract key terms"""
        doc = self.nlp(text)
        
        # Get noun phrases and important words
        keywords = []
        for chunk in doc.noun_chunks:
            keywords.append(chunk.text)
        
        # Get named entities as keywords
        for ent in doc.ents:
            keywords.append(ent.text)
        
        # Remove duplicates and return top N
        keywords = list(set(keywords))
        return keywords[:top_n]
    
    def analyze(self, text):
        """Complete analysis pipeline"""
        # Preprocess
        clean_text = self.preprocess(text)
        
        # Run all analyses
        entities = self.extract_entities(clean_text)
        category, confidence = self.classify_text(clean_text)
        sentiment, polarity = self.analyze_sentiment(clean_text)
        keywords = self.extract_keywords(clean_text)
        
        return {
            'original_text': text,
            'preprocessed_text': clean_text,
            'category': category,
            'category_confidence': round(confidence, 3),
            'sentiment': sentiment,
            'sentiment_polarity': round(polarity, 3),
            'entities': entities,
            'keywords': keywords
        }

# Test comprehensive pipeline
pipeline = ComprehensiveNLPPipeline()

test_text = """
Apple Inc. announced the new iPhone 15 today in Cupertino, California.
CEO Tim Cook praised the innovative features and excellent performance.
The product received overwhelmingly positive reviews from tech journalists.
"""

result = pipeline.analyze(test_text)

print("=== Comprehensive NLP Analysis ===\n")
print(f"Original Text: {result['original_text']}\n")
print(f"Category: {result['category']} (confidence: {result['category_confidence']})")
print(f"Sentiment: {result['sentiment']} (polarity: {result['sentiment_polarity']})")
print(f"\nEntities:")
for ent in result['entities']:
    print(f"  - {ent['text']} ({ent['label']})")
print(f"\nKeywords: {', '.join(result['keywords'])}")
```

## Lab 2: RESTful NLP API with Error Handling

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse
from pydantic import BaseModel, validator
from datetime import datetime
import logging
import time

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="Comprehensive NLP API",
    description="Multi-component NLP analysis service",
    version="1.0.0"
)

# Initialize pipeline
nlp_pipeline = ComprehensiveNLPPipeline()

class AnalysisRequest(BaseModel):
    text: str
    include_entities: bool = True
    include_sentiment: bool = True
    include_classification: bool = True
    include_keywords: bool = True
    
    @validator('text')
    def validate_text(cls, v):
        if not v or len(v.strip()) == 0:
            raise ValueError('Text cannot be empty')
        if len(v) > 10000:
            raise ValueError('Text too long (max 10000 characters)')
        return v

class AnalysisResponse(BaseModel):
    text: str
    category: str = None
    category_confidence: float = None
    sentiment: str = None
    sentiment_polarity: float = None
    entities: list = []
    keywords: list = []
    processing_time: float
    timestamp: str

# Middleware for logging
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    # Log request
    logger.info(f"Request: {request.method} {request.url.path}")
    
    # Process request
    response = await call_next(request)
    
    # Log response
    process_time = time.time() - start_time
    logger.info(f"Completed in {process_time:.2f}s with status {response.status_code}")
    
    return response

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unhandled exception: {exc}")
    return JSONResponse(
        status_code=500,
        content={
            "error": "Internal server error",
            "message": str(exc),
            "timestamp": datetime.utcnow().isoformat()
        }
    )

@app.post("/analyze", response_model=AnalysisResponse)
async def analyze_text(request: AnalysisRequest):
    """Comprehensive text analysis"""
    try:
        start_time = time.time()
        
        # Run analysis
        result = nlp_pipeline.analyze(request.text)
        
        # Build response based on requested components
        response_data = {
            'text': request.text,
            'processing_time': time.time() - start_time,
            'timestamp': datetime.utcnow().isoformat() + "Z"
        }
        
        if request.include_classification:
            response_data['category'] = result['category']
            response_data['category_confidence'] = result['category_confidence']
        
        if request.include_sentiment:
            response_data['sentiment'] = result['sentiment']
            response_data['sentiment_polarity'] = result['sentiment_polarity']
        
        if request.include_entities:
            response_data['entities'] = result['entities']
        
        if request.include_keywords:
            response_data['keywords'] = result['keywords']
        
        return response_data
    
    except Exception as e:
        logger.error(f"Analysis failed: {e}")
        raise HTTPException(status_code=500, detail=f"Analysis failed: {str(e)}")

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "components": {
            "spacy": "loaded",
            "classifier": "ready",
            "sentiment": "ready"
        }
    }

@app.get("/metrics")
async def get_metrics():
    """System metrics"""
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "uptime": "available",
        "requests_processed": "tracked in production"
    }

# To run: uvicorn script_name:app --reload
```

## Lab 3: Batch Processing System

```python
import json
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from pathlib import Path

class BatchNLPProcessor:
    """Process multiple texts efficiently"""
    
    def __init__(self, pipeline, max_workers=4):
        self.pipeline = pipeline
        self.max_workers = max_workers
    
    def process_single(self, text_id, text):
        """Process single text"""
        try:
            result = self.pipeline.analyze(text)
            return {
                'id': text_id,
                'status': 'success',
                'result': result
            }
        except Exception as e:
            return {
                'id': text_id,
                'status': 'error',
                'error': str(e)
            }
    
    def process_batch(self, texts, show_progress=True):
        """Process multiple texts in parallel"""
        results = []
        total = len(texts)
        
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            # Submit all tasks
            future_to_id = {
                executor.submit(self.process_single, i, text): i 
                for i, text in enumerate(texts)
            }
            
            # Collect results
            completed = 0
            for future in as_completed(future_to_id):
                result = future.result()
                results.append(result)
                completed += 1
                
                if show_progress:
                    print(f"Progress: {completed}/{total} ({completed/total*100:.1f}%)")
        
        return results
    
    def process_file(self, input_file, output_file):
        """Process texts from file"""
        # Read input
        with open(input_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        texts = data if isinstance(data, list) else [data]
        
        # Process
        results = self.process_batch(texts)
        
        # Write output
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(results, f, indent=2, ensure_ascii=False)
        
        return len(results)

# Test batch processing
batch_processor = BatchNLPProcessor(pipeline, max_workers=4)

test_texts = [
    "Apple released new products today.",
    "The election results surprised many analysts.",
    "The team won the championship game.",
    "Stock markets reached new highs.",
    "Scientists discovered a new species."
] * 5  # 25 texts total

print("Starting batch processing...")
start_time = time.time()

results = batch_processor.process_batch(test_texts)

elapsed = time.time() - start_time
print(f"\nCompleted {len(results)} texts in {elapsed:.2f}s")
print(f"Average: {elapsed/len(results):.3f}s per text")

# Show sample results
print("\nSample Results:")
for result in results[:3]:
    if result['status'] == 'success':
        print(f"\nText {result['id']}:")
        print(f"  Category: {result['result']['category']}")
        print(f"  Sentiment: {result['result']['sentiment']}")
```

## Lab 4: Complete NLP Application with Web Interface

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.responses import HTMLResponse
import uvicorn

app = FastAPI()

# HTML template for web interface
HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>NLP Analysis Tool</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        textarea {
            width: 100%;
            height: 150px;
            padding: 10px;
            font-size: 14px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            cursor: pointer;
            font-size: 16px;
            margin-top: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
        #results {
            margin-top: 20px;
            padding: 20px;
            background-color: #f5f5f5;
            border-radius: 5px;
        }
        .result-item {
            margin: 10px 0;
        }
        .label {
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>📝 NLP Analysis Tool</h1>
    <p>Enter text below to analyze:</p>
    
    <textarea id="textInput" placeholder="Enter your text here..."></textarea>
    
    <div>
        <label><input type="checkbox" id="entities" checked> Named Entities</label>
        <label><input type="checkbox" id="sentiment" checked> Sentiment</label>
        <label><input type="checkbox" id="classification" checked> Classification</label>
        <label><input type="checkbox" id="keywords" checked> Keywords</label>
    </div>
    
    <button onclick="analyzeText()">Analyze</button>
    
    <div id="results"></div>
    
    <script>
        async function analyzeText() {
            const text = document.getElementById('textInput').value;
            
            if (!text.trim()) {
                alert('Please enter some text');
                return;
            }
            
            const request = {
                text: text,
                include_entities: document.getElementById('entities').checked,
                include_sentiment: document.getElementById('sentiment').checked,
                include_classification: document.getElementById('classification').checked,
                include_keywords: document.getElementById('keywords').checked
            };
            
            try {
                const response = await fetch('/analyze', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(request)
                });
                
                const data = await response.json();
                displayResults(data);
            } catch (error) {
                document.getElementById('results').innerHTML = 
                    '<p style="color: red;">Error: ' + error.message + '</p>';
            }
        }
        
        function displayResults(data) {
            let html = '<h2>Analysis Results</h2>';
            
            if (data.category) {
                html += `<div class="result-item">
                    <span class="label">Category:</span> ${data.category} 
                    (${(data.category_confidence * 100).toFixed(1)}% confidence)
                </div>`;
            }
            
            if (data.sentiment) {
                html += `<div class="result-item">
                    <span class="label">Sentiment:</span> ${data.sentiment} 
                    (polarity: ${data.sentiment_polarity.toFixed(2)})
                </div>`;
            }
            
            if (data.entities && data.entities.length > 0) {
                html += '<div class="result-item"><span class="label">Entities:</span><ul>';
                data.entities.forEach(ent => {
                    html += `<li>${ent.text} (${ent.label})</li>`;
                });
                html += '</ul></div>';
            }
            
            if (data.keywords && data.keywords.length > 0) {
                html += `<div class="result-item">
                    <span class="label">Keywords:</span> ${data.keywords.join(', ')}
                </div>`;
            }
            
            html += `<div class="result-item">
                <span class="label">Processing Time:</span> ${(data.processing_time * 1000).toFixed(2)}ms
            </div>`;
            
            document.getElementById('results').innerHTML = html;
        }
    </script>
</body>
</html>
"""

@app.get("/", response_class=HTMLResponse)
async def get_interface():
    """Serve web interface"""
    return HTML_TEMPLATE

# Include analysis endpoint from Lab 2
# ... (paste the analyze endpoint here)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Lab 5: Week 9 Final Project - Complete NLP System

```python
import json
from datetime import datetime
from pathlib import Path

class NLPSystemManager:
    """Manage complete NLP system with all components"""
    
    def __init__(self, config_file=None):
        self.pipeline = ComprehensiveNLPPipeline()
        self.batch_processor = BatchNLPProcessor(self.pipeline)
        self.stats = {
            'total_processed': 0,
            'successful': 0,
            'failed': 0,
            'start_time': datetime.now().isoformat()
        }
        
        if config_file:
            self.load_config(config_file)
    
    def load_config(self, config_file):
        """Load configuration"""
        with open(config_file, 'r') as f:
            self.config = json.load(f)
    
    def process_request(self, text, options=None):
        """Process single request"""
        try:
            result = self.pipeline.analyze(text)
            self.stats['total_processed'] += 1
            self.stats['successful'] += 1
            return {'status': 'success', 'data': result}
        except Exception as e:
            self.stats['total_processed'] += 1
            self.stats['failed'] += 1
            return {'status': 'error', 'error': str(e)}
    
    def get_statistics(self):
        """Get system statistics"""
        success_rate = (self.stats['successful'] / self.stats['total_processed'] * 100 
                       if self.stats['total_processed'] > 0 else 0)
        
        return {
            'total_processed': self.stats['total_processed'],
            'successful': self.stats['successful'],
            'failed': self.stats['failed'],
            'success_rate': f"{success_rate:.2f}%",
            'uptime_since': self.stats['start_time']
        }
    
    def export_report(self, output_file):
        """Export system report"""
        report = {
            'timestamp': datetime.now().isoformat(),
            'statistics': self.get_statistics(),
            'configuration': getattr(self, 'config', {})
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2)

# Initialize complete system
system = NLPSystemManager()

# Process some texts
test_texts = [
    "Apple announced new products",
    "Political debate heats up",
    "Team wins championship"
]

print("=== NLP System Final Project ===\n")
print("Processing texts...")

for i, text in enumerate(test_texts, 1):
    result = system.process_request(text)
    print(f"\n{i}. {text}")
    if result['status'] == 'success':
        data = result['data']
        print(f"   Category: {data['category']}")
        print(f"   Sentiment: {data['sentiment']}")
        print(f"   Entities: {len(data['entities'])}")
    else:
        print(f"   Error: {result['error']}")

# Show statistics
print("\n" + "="*50)
print("System Statistics:")
stats = system.get_statistics()
for key, value in stats.items():
    print(f"  {key}: {value}")

print("\nWeek 9 Complete - NLP Revision & Integration Successful!")
```

---

# **Practice Exercises**

1. Build a complete NLP service with Docker deployment
2. Create a multi-language support system
3. Implement caching for improved performance
4. Build a monitoring dashboard for NLP metrics
5. Create comprehensive documentation and user guide

---

# **Week 9 Summary**

## Topics Covered

- **Day 41**: NLP fundamentals revision and advanced tokenization
- **Day 42**: Advanced text classification with ensemble methods
- **Day 43**: Production sentiment analysis systems
- **Day 44**: Custom NER training and entity linking
- **Day 45**: Complete NLP system integration

## Skills Acquired

Deep understanding of NLP preprocessing techniques  
Building production-ready classification systems  
Implementing multi-component sentiment analysis  
Training custom NER models for specific domains  
Integrating NLP components into cohesive systems  
API design and deployment best practices  
Performance optimization and monitoring  

## Key Achievements

- Mastered complete NLP development lifecycle
- Built production-ready NLP applications
- Integrated multiple NLP components effectively
- Implemented robust error handling and monitoring
- Created comprehensive testing strategies

