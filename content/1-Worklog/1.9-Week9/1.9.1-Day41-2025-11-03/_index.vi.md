---
title: "Ngày 41 - Ôn tập kiến thức cơ bản NLP"
weight: 1
chapter: false
pre: "<b> 1.9.1. </b>"
---

**Ngày:** 2025-11-03
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Tóm tắt Tuần 8

### Các khái niệm NLP cốt lõi được xem lại

**Quy trình tiền xử lý văn bản**
- Tokenization: cấp độ từ và câu
- Chuẩn hóa: chữ thường, loại bỏ ký tự đặc biệt
- Loại bỏ stop word: quyết định phụ thuộc ngữ cảnh
- Stemming vs Lemmatization: sự đánh đổi và trường hợp sử dụng

**Những điểm chính từ Tuần 8**
- Chất lượng tiền xử lý ảnh hưởng trực tiếp đến hiệu suất mô hình
- Các tác vụ khác nhau yêu cầu chiến lược tiền xử lý khác nhau
- Lemmatization thường được ưu tiên cho hệ thống production
- Luôn xác thực lựa chọn tiền xử lý với trường hợp sử dụng cụ thể

## Đi sâu: Kỹ thuật Tokenization

### Tokenization nâng cao

**Subword Tokenization**
- Byte Pair Encoding (BPE): xử lý các từ ngoài từ vựng
- WordPiece: được BERT và các transformer khác sử dụng
- SentencePiece: tokenization không phụ thuộc ngôn ngữ
- Trường hợp sử dụng: mô hình đa ngôn ngữ và xử lý từ hiếm

**So sánh các cách tiếp cận**

| Phương pháp | Ưu điểm | Nhược điểm | Tốt nhất cho |
|-------------|---------|------------|--------------|
| Word | Đơn giản, nhanh | Từ vựng lớn | Tác vụ đơn giản |
| Character | Không có vấn đề OOV | Chuỗi dài | Tác vụ chính tả |
| Subword | Cách tiếp cận cân bằng | Phức tạp hơn | NLP hiện đại |

### Tokenization biểu thức chính quy

- Mẫu tùy chỉnh cho văn bản đặc thù lĩnh vực
- Xử lý URLs, emails, hashtags
- Bảo toàn dấu câu quan trọng
- Tokenization văn bản y tế/kỹ thuật

## Thực hành tốt nhất về chuẩn hóa văn bản

### Khi nào nên chuẩn hóa

**Nên chuẩn hóa:**
- Độ nhạy chữ hoa/thường không quan trọng
- Cần định dạng nhất quán
- Ràng buộc về bộ nhớ/tính toán

**Tránh chuẩn hóa quá mức:**
- Thực thể có tên quan trọng
- Phân tích cảm xúc (biểu tượng cảm xúc quan trọng)
- Code hoặc tài liệu kỹ thuật

### Xử lý Unicode

- Mã hóa/giải mã đúng cách
- Các dạng chuẩn hóa (NFC, NFD, NFKC, NFKD)
- Xử lý văn bản đa ngôn ngữ
- Bảo toàn emoji và ký tự đặc biệt

## Những hiểu biết quan trọng

- Xem lại kiến thức cơ bản tiết lộ sự hiểu biết sâu sắc hơn
- Các trường hợp biên thường xác định chiến lược tiền xử lý
- NLP hiện đại ngày càng sử dụng subword tokenization
- Sự cân bằng giữa đơn giản và hiệu quả là rất quan trọng

---

# **Thực hành Lab**

## Lab 1: So sánh tiền xử lý toàn diện

```python
import nltk
from nltk.tokenize import word_tokenize, sent_tokenize
from nltk.stem import PorterStemmer, WordNetLemmatizer
from nltk.corpus import stopwords
import string

text = """
The AI-powered system's performance improved significantly! 
Running multiple tests @ 99.9% accuracy. #MachineLearning
"""

def compare_preprocessing(text):
    """So sánh các cách tiếp cận tiền xử lý khác nhau"""
    
    # Gốc
    print("VĂN BẢN GỐC:")
    print(text)
    print("\n" + "="*60 + "\n")
    
    # Tokenization cơ bản
    words_basic = word_tokenize(text)
    print("TOKENIZATION CƠ BẢN:")
    print(words_basic)
    print(f"Số lượng token: {len(words_basic)}\n")
    
    # Chữ thường + loại bỏ dấu câu
    words_clean = [w.lower() for w in words_basic if w not in string.punctuation]
    print("CHỮ THƯỜNG + KHÔNG DẤU CÂU:")
    print(words_clean)
    print(f"Số lượng token: {len(words_clean)}\n")
    
    # Loại bỏ stop word
    stop_words = set(stopwords.words('english'))
    words_no_stop = [w for w in words_clean if w not in stop_words]
    print("KHÔNG STOP WORDS:")
    print(words_no_stop)
    print(f"Số lượng token: {len(words_no_stop)}\n")
    
    # Stemming
    stemmer = PorterStemmer()
    words_stemmed = [stemmer.stem(w) for w in words_no_stop]
    print("STEMMED:")
    print(words_stemmed)
    print(f"Số lượng token: {len(words_stemmed)}\n")
    
    # Lemmatization
    lemmatizer = WordNetLemmatizer()
    words_lemmatized = [lemmatizer.lemmatize(w, pos='v') for w in words_no_stop]
    print("LEMMATIZED:")
    print(words_lemmatized)
    print(f"Số lượng token: {len(words_lemmatized)}\n")

compare_preprocessing(text)
```

## Lab 2: Tokenization tùy chỉnh với Regex

```python
import re

def custom_tokenize(text):
    """Tokenization tùy chỉnh bảo toàn các mẫu đặc biệt"""
    
    # Mẫu cho các phần tử văn bản khác nhau
    patterns = [
        r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+',  # URLs
        r'[\w.+-]+@[\w-]+\.[\w.-]+',  # Emails
        r'#\w+',  # Hashtags
        r'@\w+',  # Mentions
        r'\d+\.?\d*%',  # Phần trăm
        r'\$\d+\.?\d*',  # Tiền
        r'\d{4}-\d{2}-\d{2}',  # Ngày tháng
        r'\w+',  # Từ
        r'[^\w\s]',  # Dấu câu
    ]
    
    pattern = '|'.join(patterns)
    tokens = re.findall(pattern, text)
    
    return tokens

# Kiểm tra tokenization tùy chỉnh
sample = """
Xem https://example.com để biết cập nhật AI!
Liên hệ: info@ai-company.com
Sự kiện: 2025-11-03 @10:00AM
Ngân sách: $50,000 (99.5% tài trợ) #TechEvent
"""

tokens = custom_tokenize(sample)
print("Tokenization tùy chỉnh:")
for i, token in enumerate(tokens, 1):
    print(f"{i}. {token}")
```

## Lab 3: Xử lý văn bản đa ngôn ngữ

```python
def process_multilingual_text(text):
    """Xử lý văn bản với nhiều ngôn ngữ và ký tự đặc biệt"""
    
    # Chuẩn hóa Unicode
    import unicodedata
    
    # NFC (Canonical Decomposition, followed by Canonical Composition)
    normalized = unicodedata.normalize('NFC', text)
    
    # Xác định các mẫu đặc thù ngôn ngữ
    has_emoji = bool(re.search(r'[\U0001F600-\U0001F64F]', text))
    has_cjk = bool(re.search(r'[\u4e00-\u9fff]', text))  # Trung/Nhật/Hàn
    has_arabic = bool(re.search(r'[\u0600-\u06ff]', text))
    
    info = {
        'gốc': text,
        'đã_chuẩn_hóa': normalized,
        'có_emoji': has_emoji,
        'có_cjk': has_cjk,
        'có_arabic': has_arabic,
        'độ_dài': len(text),
        'độ_dài_chuẩn_hóa': len(normalized)
    }
    
    return info

# Kiểm tra xử lý đa ngôn ngữ
multilingual_samples = [
    "Hello 世界! 🌍",
    "Natural Language Processing",
    "café résumé naïve",
    "مرحبا العالم"
]

for sample in multilingual_samples:
    result = process_multilingual_text(sample)
    print(f"\nVăn bản: {result['gốc']}")
    print(f"Đã chuẩn hóa: {result['đã_chuẩn_hóa']}")
    print(f"Có Emoji: {result['có_emoji']}")
    print(f"Có CJK: {result['có_cjk']}")
    print(f"Có Arabic: {result['có_arabic']}")
```

## Lab 4: Xây dựng quy trình tiền xử lý

```python
class PreprocessingPipeline:
    """Quy trình tiền xử lý linh hoạt"""
    
    def __init__(self):
        self.steps = []
        self.stemmer = PorterStemmer()
        self.lemmatizer = WordNetLemmatizer()
        self.stop_words = set(stopwords.words('english'))
    
    def add_lowercase(self):
        self.steps.append(('chữ_thường', lambda x: x.lower()))
        return self
    
    def add_tokenization(self):
        self.steps.append(('tokenize', word_tokenize))
        return self
    
    def add_remove_punctuation(self):
        def remove_punct(tokens):
            return [t for t in tokens if t not in string.punctuation]
        self.steps.append(('loại_dấu_câu', remove_punct))
        return self
    
    def add_remove_stopwords(self):
        def remove_stop(tokens):
            return [t for t in tokens if t not in self.stop_words]
        self.steps.append(('loại_stop_word', remove_stop))
        return self
    
    def add_stemming(self):
        def stem(tokens):
            return [self.stemmer.stem(t) for t in tokens]
        self.steps.append(('stem', stem))
        return self
    
    def add_lemmatization(self, pos='v'):
        def lemmatize(tokens):
            return [self.lemmatizer.lemmatize(t, pos=pos) for t in tokens]
        self.steps.append(('lemmatize', lemmatize))
        return self
    
    def process(self, text):
        result = text
        for step_name, step_func in self.steps:
            result = step_func(result)
            print(f"Sau {step_name}: {result[:100]}...")  # Hiển thị 100 ký tự đầu
        return result

# Xây dựng quy trình tùy chỉnh
pipeline = (PreprocessingPipeline()
            .add_lowercase()
            .add_tokenization()
            .add_remove_punctuation()
            .add_remove_stopwords()
            .add_lemmatization())

text = "The students are studying NLP concepts and building amazing applications!"
result = pipeline.process(text)
print(f"\nKết quả cuối cùng: {result}")
```

---

# **Bài tập thực hành**

1. So sánh các cách tiếp cận tiền xử lý trên các loại văn bản khác nhau (tweets, bài báo, code)
2. Xây dựng quy trình tiền xử lý cho một lĩnh vực cụ thể (y tế, pháp lý, mạng xã hội)
3. Triển khai tokenization tùy chỉnh để xử lý các định dạng đặc biệt (mã sản phẩm, IDs)
4. Phân tích tác động của các lựa chọn tiền xử lý khác nhau đến tác vụ phân loại
5. Tạo benchmark tiền xử lý so sánh sự đánh đổi giữa tốc độ và chất lượng
