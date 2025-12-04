---
title: "Ngày 45 - Tích hợp NLP & Dự án cuối cùng"
weight: 5
chapter: false
pre: "<b> 1.9.5. </b>"
---

**Ngày:** 2025-11-07 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Tóm tắt Tuần 9

### Các khái niệm được đề cập trong tuần này

- **Ngày 41**: Ôn tập kiến thức cơ bản NLP - tokenization, quy trình tiền xử lý
- **Ngày 42**: Phân loại văn bản nâng cao - phương pháp ensemble, dữ liệu mất cân bằng
- **Ngày 43**: Phân tích cảm xúc production - dựa trên khía cạnh, phát hiện cảm xúc, APIs
- **Ngày 44**: NER nâng cao - huấn luyện tùy chỉnh, entity linking, trích xuất quan hệ
- **Ngày 45**: Tích hợp & hệ thống NLP hoàn chỉnh

### Những điều học được chính

- Xây dựng hệ thống NLP sẵn sàng production đòi hỏi nhiều hơn mô hình tốt
- Tích hợp nhiều thành phần NLP tạo ra ứng dụng mạnh mẽ
- Giám sát, đánh giá và cải tiến liên tục là thiết yếu
- Thích ứng lĩnh vực quan trọng cho hiệu suất thực tế

## Tích hợp các thành phần NLP

### Quy trình đa thành phần

```
Đầu vào văn bản → Tiền xử lý → Phân loại → NER → Cảm xúc → Đầu ra
```

**Lợi ích của tích hợp:**
- API đơn cho nhiều tác vụ NLP
- Tiền xử lý chung giảm tính toán
- Xử lý lỗi nhất quán trên các thành phần
- Bảo trì và cập nhật dễ dàng hơn

### Các mẫu thiết kế

**Mẫu Pipeline**
- Xử lý văn bản tuần tự qua nhiều giai đoạn
- Mỗi giai đoạn có thể được phát triển và kiểm tra độc lập
- Dễ dàng thêm/xóa giai đoạn

**Mẫu Microservices**
- Mỗi tác vụ NLP là dịch vụ riêng biệt
- Có thể mở rộng và triển khai độc lập
- Giao tiếp qua REST API hoặc message queues

**Mẫu Monolithic**
- Tất cả tác vụ NLP trong một ứng dụng
- Triển khai đơn giản hơn
- Phù hợp cho ứng dụng nhỏ hơn

## Xây dựng ứng dụng NLP hoàn chỉnh

### Phân tích yêu cầu

**Yêu cầu chức năng:**
- Cần những tác vụ NLP nào?
- Độ chính xác chấp nhận được là bao nhiêu?
- Xử lý thời gian thực hay hàng loạt?
- Định dạng đầu vào/đầu ra?

**Yêu cầu phi chức năng:**
- Hiệu suất (độ trễ, thông lượng)
- Khả năng mở rộng (người dùng đồng thời, khối lượng dữ liệu)
- Độ tin cậy (thời gian hoạt động, tỷ lệ lỗi)
- Bảo mật (quyền riêng tư dữ liệu, xác thực)

### Kiến trúc hệ thống

**Các lớp:**
1. **Lớp API**: REST/GraphQL endpoints, xác thực yêu cầu
2. **Lớp Business Logic**: Điều phối quy trình NLP
3. **Lớp Processing**: Các thành phần NLP riêng lẻ
4. **Lớp Data**: Lưu trữ mô hình, caching, logging

## Thực hành tốt nhất

### Tổ chức code

- Tách biệt mối quan tâm: tiền xử lý, mô hình, API, utilities
- Quản lý cấu hình (biến môi trường, config files)
- Dependency injection cho tính linh hoạt
- Quy ước đặt tên rõ ràng

### Chiến lược kiểm thử

- Unit tests cho các thành phần riêng lẻ
- Integration tests cho pipelines
- End-to-end tests cho quy trình hoàn chỉnh
- Performance/load testing

### Triển khai

- Containerization (Docker) cho tính nhất quán
- CI/CD pipelines cho triển khai tự động
- Blue-green hoặc canary deployments cho an toàn
- Health checks và monitoring

## Những hiểu biết quan trọng

- Bắt đầu đơn giản, lặp lại dựa trên sử dụng thực tế
- Log mọi thứ để debug và cải thiện
- Lên kế hoạch cho lỗi - xử lý graceful
- Vòng lặp phản hồi người dùng thiết yếu cho cải thiện
- Tài liệu quan trọng không kém code

---

# **Thực hành Lab**

## Lab 1: Quy trình NLP đa thành phần

```python
import spacy
from textblob import TextBlob
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
import pickle

class ComprehensiveNLPPipeline:
    """Quy trình NLP tích hợp với nhiều thành phần"""
    
    def __init__(self):
        # Tải mô hình
        self.nlp = spacy.load("en_core_web_sm")
        
        # Khởi tạo thành phần
        self.vectorizer = TfidfVectorizer(max_features=100)
        self.classifier = MultinomialNB()
        
        # Huấn luyện classifier đơn giản (trong production, tải đã huấn luyện trước)
        sample_texts = ["đánh giá sản phẩm công nghệ", "bài báo tin tức chính trị", "tóm tắt trận đấu thể thao"] * 10
        sample_labels = ["công nghệ", "chính trị", "thể thao"] * 10
        X = self.vectorizer.fit_transform(sample_texts)
        self.classifier.fit(X, sample_labels)
    
    def preprocess(self, text):
        """Làm sạch và chuẩn hóa văn bản"""
        # Chữ thường
        text = text.lower()
        # Loại bỏ khoảng trắng thừa
        text = ' '.join(text.split())
        return text
    
    def extract_entities(self, text):
        """Trích xuất thực thể được đặt tên"""
        doc = self.nlp(text)
        entities = [
            {
                'văn_bản': ent.text,
                'nhãn': ent.label_,
                'bắt_đầu': ent.start_char,
                'kết_thúc': ent.end_char
            }
            for ent in doc.ents
        ]
        return entities
    
    def classify_text(self, text):
        """Phân loại văn bản vào danh mục"""
        X = self.vectorizer.transform([text])
        category = self.classifier.predict(X)[0]
        probabilities = self.classifier.predict_proba(X)[0]
        confidence = max(probabilities)
        return category, confidence
    
    def analyze_sentiment(self, text):
        """Phân tích cảm xúc"""
        blob = TextBlob(text)
        polarity = blob.sentiment.polarity
        
        if polarity > 0.1:
            sentiment = "tích_cực"
        elif polarity < -0.1:
            sentiment = "tiêu_cực"
        else:
            sentiment = "trung_tính"
        
        return sentiment, polarity
    
    def extract_keywords(self, text, top_n=5):
        """Trích xuất các từ khóa"""
        doc = self.nlp(text)
        
        # Lấy noun phrases và từ quan trọng
        keywords = []
        for chunk in doc.noun_chunks:
            keywords.append(chunk.text)
        
        # Lấy named entities làm keywords
        for ent in doc.ents:
            keywords.append(ent.text)
        
        # Loại bỏ trùng lặp và trả về top N
        keywords = list(set(keywords))
        return keywords[:top_n]
    
    def analyze(self, text):
        """Quy trình phân tích hoàn chỉnh"""
        # Tiền xử lý
        clean_text = self.preprocess(text)
        
        # Chạy tất cả phân tích
        entities = self.extract_entities(clean_text)
        category, confidence = self.classify_text(clean_text)
        sentiment, polarity = self.analyze_sentiment(clean_text)
        keywords = self.extract_keywords(clean_text)
        
        return {
            'văn_bản_gốc': text,
            'văn_bản_đã_xử_lý': clean_text,
            'danh_mục': category,
            'độ_tin_cậy_danh_mục': round(confidence, 3),
            'cảm_xúc': sentiment,
            'polarity_cảm_xúc': round(polarity, 3),
            'thực_thể': entities,
            'từ_khóa': keywords
        }

# Kiểm tra quy trình toàn diện
pipeline = ComprehensiveNLPPipeline()

test_text = """
Apple Inc. công bố iPhone 15 mới hôm nay tại Cupertino, California.
CEO Tim Cook ca ngợi các tính năng đổi mới và hiệu suất xuất sắc.
Sản phẩm nhận được đánh giá tích cực áp đảo từ các nhà báo công nghệ.
"""

result = pipeline.analyze(test_text)

print("=== Phân tích NLP toàn diện ===\n")
print(f"Văn bản gốc: {result['văn_bản_gốc']}\n")
print(f"Danh mục: {result['danh_mục']} (tin cậy: {result['độ_tin_cậy_danh_mục']})")
print(f"Cảm xúc: {result['cảm_xúc']} (polarity: {result['polarity_cảm_xúc']})")
print(f"\nThực thể:")
for ent in result['thực_thể']:
    print(f"  - {ent['văn_bản']} ({ent['nhãn']})")
print(f"\nTừ khóa: {', '.join(result['từ_khóa'])}")
```

## Lab 2: RESTful NLP API với xử lý lỗi

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse
from pydantic import BaseModel, validator
from datetime import datetime
import logging
import time

# Cấu hình logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="API NLP toàn diện",
    description="Dịch vụ phân tích NLP đa thành phần",
    version="1.0.0"
)

# Khởi tạo pipeline
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
            raise ValueError('Văn bản không thể trống')
        if len(v) > 10000:
            raise ValueError('Văn bản quá dài (tối đa 10000 ký tự)')
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

# Middleware cho logging
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    # Log yêu cầu
    logger.info(f"Yêu cầu: {request.method} {request.url.path}")
    
    # Xử lý yêu cầu
    response = await call_next(request)
    
    # Log phản hồi
    process_time = time.time() - start_time
    logger.info(f"Hoàn thành trong {process_time:.2f}s với trạng thái {response.status_code}")
    
    return response

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Ngoại lệ không xử lý: {exc}")
    return JSONResponse(
        status_code=500,
        content={
            "error": "Lỗi máy chủ nội bộ",
            "message": str(exc),
            "timestamp": datetime.utcnow().isoformat()
        }
    )

@app.post("/analyze", response_model=AnalysisResponse)
async def analyze_text(request: AnalysisRequest):
    """Phân tích văn bản toàn diện"""
    try:
        start_time = time.time()
        
        # Chạy phân tích
        result = nlp_pipeline.analyze(request.text)
        
        # Xây dựng phản hồi dựa trên các thành phần được yêu cầu
        response_data = {
            'text': request.text,
            'processing_time': time.time() - start_time,
            'timestamp': datetime.utcnow().isoformat() + "Z"
        }
        
        if request.include_classification:
            response_data['category'] = result['danh_mục']
            response_data['category_confidence'] = result['độ_tin_cậy_danh_mục']
        
        if request.include_sentiment:
            response_data['sentiment'] = result['cảm_xúc']
            response_data['sentiment_polarity'] = result['polarity_cảm_xúc']
        
        if request.include_entities:
            response_data['entities'] = result['thực_thể']
        
        if request.include_keywords:
            response_data['keywords'] = result['từ_khóa']
        
        return response_data
    
    except Exception as e:
        logger.error(f"Phân tích thất bại: {e}")
        raise HTTPException(status_code=500, detail=f"Phân tích thất bại: {str(e)}")

@app.get("/health")
async def health_check():
    """Endpoint kiểm tra sức khỏe"""
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "components": {
            "spacy": "đã tải",
            "classifier": "sẵn sàng",
            "sentiment": "sẵn sàng"
        }
    }

@app.get("/metrics")
async def get_metrics():
    """Số liệu hệ thống"""
    return {
        "timestamp": datetime.utcnow().isoformat(),
        "uptime": "có sẵn",
        "requests_processed": "được theo dõi trong production"
    }

# Chạy: uvicorn script_name:app --reload
```

## Lab 3-5: [Giống như phiên bản tiếng Anh với các thông báo và nhận xét bằng tiếng Việt]

---

# **Bài tập thực hành**

1. Xây dựng dịch vụ NLP hoàn chỉnh với triển khai Docker
2. Tạo hệ thống hỗ trợ đa ngôn ngữ
3. Triển khai caching để cải thiện hiệu suất
4. Xây dựng dashboard giám sát cho số liệu NLP
5. Tạo tài liệu toàn diện và hướng dẫn người dùng

---

# **Tóm tắt Tuần 9**

## Các chủ đề được đề cập

- **Ngày 41**: Ôn tập kiến thức cơ bản NLP và tokenization nâng cao
- **Ngày 42**: Phân loại văn bản nâng cao với phương pháp ensemble
- **Ngày 43**: Hệ thống phân tích cảm xúc production
- **Ngày 44**: Huấn luyện NER tùy chỉnh và entity linking
- **Ngày 45**: Tích hợp hệ thống NLP hoàn chỉnh

## Kỹ năng đã đạt được

Hiểu sâu về kỹ thuật tiền xử lý NLP  
Xây dựng hệ thống phân loại sẵn sàng production  
Triển khai phân tích cảm xúc đa thành phần  
Huấn luyện mô hình NER tùy chỉnh cho các lĩnh vực cụ thể  
Tích hợp các thành phần NLP thành hệ thống liền mạch  
Thiết kế API và thực hành triển khai tốt nhất  
Tối ưu hóa hiệu suất và giám sát  

## Thành tựu chính

- Nắm vững vòng đời phát triển NLP hoàn chỉnh
- Xây dựng ứng dụng NLP sẵn sàng production
- Tích hợp nhiều thành phần NLP hiệu quả
- Triển khai xử lý lỗi và giám sát mạnh mẽ
- Tạo chiến lược kiểm thử toàn diện

