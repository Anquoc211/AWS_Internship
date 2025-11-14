---
title: "Blog 2"
date: "2025-09-09T15:44:00+07:00"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Xây dựng Agentic Workflows với OpenAI GPT OSS trên Amazon SageMaker AI và Amazon Bedrock AgentCore

**bởi:** Vivek Gangasani and Surya Kari | **NXB:** 17 TH9 2025 |
**trong:** Amazon Bedrock, Amazon SageMaker AI, Amazon SageMaker Unified Studio, Artificial Intelligence, Customer Solutions

---

OpenAI đã phát hành hai mô hình open-weight, `gpt-oss-120b` (117 tỷ tham số) và `gpt-oss-20b` (21 tỷ tham số), cả hai được xây dựng theo kiến trúc Mixture of Experts (MoE) và sử dụng cửa sổ ngữ cảnh 128K. [cite: 169] Những mô hình này dẫn đầu các mô hình mã nguồn mở, theo Artificial Analysis benchmark, và excel trong khả năng suy luận và workflow agentic. [cite: 170]

Với Amazon SageMaker AI, bạn có thể fine-tune hoặc tùy chỉnh mô hình và triển khai bằng framework bạn chọn thông qua dịch vụ được quản lý toàn diện. [cite: 171] Amazon SageMaker Inference cho bạn linh hoạt trong việc mang mã và framework inference riêng mà không cần tự xây dựng và duy trì cụm máy chủ. [cite: 172]

Mặc dù các large language model (LLMs) xuất sắc trong việc hiểu ngôn ngữ và sinh nội dung, việc xây dựng các ứng dụng agentic thực tế đòi hỏi quản lý workflow phức tạp, khả năng gọi công cụ (tool calling), và quản lý ngữ cảnh. [cite: 173] Kiến trúc đa agent (multi-agent) giải quyết những thách thức này bằng cách phân chia hệ thống phức tạp thành các thành phần chuyên biệt, nhưng điều này cũng mang lại các phức tạp mới trong phối hợp agent, quản lý bộ nhớ, và điều phối workflow. [cite: 174]

Trong bài viết này, chúng tôi sẽ chỉ ra cách triển khai mô hình `gpt-oss-20b` lên các endpoint được quản lý của SageMaker và minh họa ví dụ thực tế về trợ lý agent phân tích cổ phiếu dùng LangGraph — một framework dựa trên đồ thị mạnh mẽ xử lý quản lý trạng thái, workflow phối hợp, và hệ thống bộ nhớ bền vững. [cite: 175] Sau đó, chúng tôi sẽ triển khai các agent lên Amazon Bedrock AgentCore, một lớp điều phối hợp nhất trừu tượng hóa hạ tầng, cho phép bạn triển khai và vận hành các agent AI ở quy mô một cách an toàn. [cite: 176]

## Tổng quan giải pháp

Trong giải pháp này, chúng tôi xây dựng một agent phân tích cổ phiếu (agentic stock analyzer) với các thành phần chính sau: [cite: 178]
* Mô hình GPT OSS 20B được triển khai lên endpoint SageMaker sử dụng vLLM, một framework serving mã nguồn mở cho LLMs [cite: 179]
* LangGraph để xây dựng framework điều phối đa agent [cite: 180]
* Amazon Bedrock AgentCore để triển khai các agent [cite: 181]

Sơ đồ dưới đây minh họa kiến trúc giải pháp: [cite: 182]

![Kiến trúc giải pháp](https://i.imgur.com/placeholder-blog2-img1.png)

Kiến trúc này minh họa một workflow đa agent được host trên Amazon Bedrock AgentCore Runtime chạy trên AWS. [cite: 183] Người dùng gửi truy vấn, được xử lý bởi pipeline của các agent chuyên biệt — Data Gathering Agent, Stock Performance Analyzer Agent, và Stock Report Generation Agent — mỗi agent chịu trách nhiệm một phần riêng trong quá trình đánh giá cổ phiếu. [cite: 184] Các agent này phối hợp trong Amazon Bedrock AgentCore Runtime, và khi cần hiểu hoặc sinh ngôn ngữ, chúng gọi tới một mô hình GPT OSS được host trên SageMaker AI. [cite: 185] Mô hình xử lý đầu vào và trả về kết quả cấu trúc giúp quyết định hành vi agent, cho phép hệ thống agentic hoàn toàn không máy chủ (serverless), mô-đun và có thể mở rộng sử dụng các mô hình mã nguồn mở. [cite: 186]

## Yêu cầu trước khi thực hiện

* Đảm bảo bạn có quota cần thiết cho các instance G6e để triển khai mô hình. [cite: 188]
* Yêu cầu quota tại đây nếu bạn chưa có. [cite: 189]
* Nếu đây là lần đầu bạn làm việc với Amazon SageMaker Studio, bạn cần tạo một domain SageMaker trước. [cite: 190]
* Đảm bảo role IAM của bạn có các quyền cần thiết để triển khai SageMaker Models và Endpoints. [cite: 191]
* Để biết thêm, xem How Amazon SageMaker AI works with IAM trong SageMaker Developer Guide. [cite: 192]

## Triển khai mô hình GPT-OSS lên SageMaker Inference

Khách hàng muốn tùy chỉnh mô hình và framework có thể triển khai theo cách serverful, nhưng điều này đòi hỏi truy cập GPU, các framework phục vụ, load balancer, và cấu hình hạ tầng. [cite: 194] SageMaker AI cung cấp nền tảng hosting được quản lý hoàn toàn, lo phần cấp phát hạ tầng, driver cần thiết, tải mô hình, và triển khai. [cite: 195]

Mô hình GPT-OSS của OpenAI được khởi chạy với scheme quantization 4-bit (MXFP4), cho phép inference nhanh trong khi giữ tài nguyên thấp. [cite: 196] Những mô hình này có thể chạy trên các instance `P5(H100)`, `P6(H200)`, `P4(A100)` và `G6e(L40)`. [cite: 197] Các mô hình GPT-OSS là kiến trúc MoE thưa với 128 expert (120B) hoặc 32 expert (20B), trong đó mỗi token được định tuyến tới 4 experts mà không chia sẻ expert. [cite: 198] Sử dụng MXFP4 cho trọng số MoE giúp thu nhỏ kích thước mô hình xuống còn 63 GB (120B) và 14 GB (20B), khiến chúng có thể chạy trên một GPU H100 đơn. [cite: 199]

Để triển khai hiệu quả, bạn cần một framework serving mạnh như vLLM. [cite: 200] Để triển khai mô hình, chúng tôi xây dựng một container vLLM với phiên bản mới nhất hỗ trợ các mô hình GPT OSS trên SageMaker AI. [cite: 201] Bạn có thể dùng Docker file và script sau để build container và push lên Amazon Elastic Container Registry (Amazon ECR). [cite: 202] Cách được khuyến nghị là thực hiện trực tiếp từ SageMaker Studio, môi trường JupyterLab được quản lý có truy cập AWS CLI, nơi bạn có thể build và push image lên ECR như một phần của workflow SageMaker. [cite: 203] Ngoài ra, bạn cũng có thể thực hiện các bước tương tự trên một instance Amazon Elastic Compute Cloud (Amazon EC2) có cài Docker. [cite: 204]

Sau khi bạn đã build và push container lên Amazon ECR, bạn có thể mở SageMaker Studio qua console SageMaker AI, như minh họa trong screenshot dưới đây: [cite: 205]

![Mở SageMaker Studio](https://i.imgur.com/placeholder-blog2-img2.png)

Sau đó bạn có thể tạo không gian Jupyter hoặc dùng không gian có sẵn để mở JupyterLab và chạy notebook. [cite: 206]

![Tạo không gian JupyterLab](https://i.imgur.com/placeholder-blog2-img3.png)

Clone notebook sau và chạy “Option 3: Deploying from HF using BYOC.” [cite: 207] Cập nhật các tham số cần thiết, như image inference trong notebook với image container. [cite: 208] Chúng tôi cũng cung cấp các biến môi trường cần thiết, như sau: [cite: 209]

```python
# 
inference_image = f"{account_id}.dkr.ecr.{region}[.amazonaws.com/vllm:v0.10.0-gpt-oss](https://.amazonaws.com/vllm:v0.10.0-gpt-oss)"
instance_type = "ml.g6e.4xlarge"
num_gpu = 1
model_name = sagemaker.utils.name_from_base("model-byoc")
endpoint_name = model_name
inference_component_name = f"ic-{model_name}"
config = {
    "OPTION_MODEL": "openai/gpt-oss-20b",
    "OPTION_SERVED_MODEL_NAME": "model",
    "OPTION_TENSOR_PARALLEL_SIZE": json.dumps(num_gpu),
    "OPTION_ASYNC_SCHEDULING": "true",
}
````

[cite\_start]Sau khi bạn thiết lập cấu hình triển khai, bạn có thể deploy lên SageMaker AI bằng mã sau: [cite: 211]

```python
# 
from sagemaker.compute_resource_requirements.resource_requirements import ResourceRequirements
lmi_model = sagemaker.Model(
    image_uri=inference_image,
    env=config,
    role=role,
    name=model_name,
)
lmi_model.deploy(
    initial_instance_count=1,
    instance_type=instance_type,
    container_startup_health_check_timeout=600,
    endpoint_name=endpoint_name,
    endpoint_type=sagemaker.enums.EndpointType.INFERENCE_COMPONENT_BASED,
    inference_component_name=inference_component_name,
    resources=ResourceRequirements(requests={"num_accelerators": num_gpu, "memory": 1024*5, "copies": 1,}),
)
```

[cite\_start]Bạn có thể chạy một ví dụ inference: [cite: 213]

```python
# 
payload={
    "messages": [
        {"role": "user", "content": "Name popular places to visit in London?""}
    ],
}
res = llm.predict(payload)
print("-----\n" + res["choices"][0]["message"]["content"] + "\n-----\n")
print(res["usage"])

# Output:
# -----
# Here are some of the must-see spots in London -- a mix of iconic landmarks, world-class museums, and vibrant neighborhoods:
# | # | Place | Why It's Popular ||---|-------|------------------|
# | 1 | **Buckingham Palace** | The Queen's official London residence - watch the Changing of the Guard. |
# | 2 | **The Tower of London & Tower Bridge** | Historic castle, Crown Jewels, and the iconic bridge with glass floors. |
# | 3 | **The British Museum** | World-famous collection from the Rosetta Stone to Egyptian mummies (free entry). |
# ... (Phần còn lại của bảng) ...
# |15 | **Oxford Street & Regent Street** | Prime shopping streets for fashion, flagship stores, and historic architecture. |
# These spots cover history, culture, shopping, and leisure--perfect for a first visit or a weekend escape in London!
# -----
```

## Dùng LangGraph để xây dựng agent phân tích cổ phiếu

[cite\_start]Đối với hệ thống đa agent phân tích cổ phiếu của chúng tôi, chúng tôi sử dụng LangGraph để điều phối workflow. [cite: 218] Jupyter notebook cho mã nằm trong Github repository này. [cite\_start]Hệ thống bao gồm ba công cụ chuyên biệt làm việc cùng nhau để phân tích cổ phiếu toàn diện: [cite: 219]

  * [cite\_start]**gather\_stock\_data tool** thu thập dữ liệu cổ phiếu toàn diện cho mã cổ phiếu cho trước, bao gồm giá hiện tại, hiệu suất lịch sử, các chỉ số tài chính và dữ liệu thị trường. [cite: 220] [cite\_start]Nó trả về thông tin định dạng bao gồm lịch sử giá, cơ bản công ty, các chỉ số giao dịch, và tiêu đề tin tức gần đây. [cite: 221]
  * [cite\_start]**analyze\_stock\_performance tool** thực hiện phân tích kỹ thuật và cơ bản chi tiết của dữ liệu cổ phiếu, tính các chỉ số như xu hướng giá, độ biến động, và điểm đầu tư tổng thể. [cite: 222] [cite\_start]Nó đánh giá nhiều yếu tố bao gồm tỉ lệ P/E, biên lợi nhuận, và lợi suất cổ tức để cung cấp phân tích hiệu suất toàn diện. [cite: 223]
  * [cite\_start]**generate\_stock\_report tool** tạo báo cáo PDF chuyên nghiệp từ dữ liệu cổ phiếu thu thập và phân tích, tự động upload lên Amazon S3 với thư mục theo ngày. [cite: 224]

[cite\_start]Để test cục bộ, bạn có thể dùng phiên bản đơn giản của hệ thống bằng cách import các hàm cần thiết từ script cục bộ của bạn. [cite: 225] [cite\_start]Ví dụ: [cite: 226]

```python
# 
from langgraph_stock_local import langgraph_stock_sagemaker 
# Test the agent locally 
result = langgraph_stock_sagemaker({
    "prompt": "Analyze SIM_STOCK Stock for Investment purposes."
})
print(result)
```

[cite\_start]Cách này giúp bạn lặp logic của agent nhanh chóng trước khi deploy lên nền tảng có thể mở rộng, đảm bảo mỗi thành phần hoạt động đúng và workflow tổng thể cho kết quả mong đợi cho nhiều loại cổ phiếu khác nhau. [cite: 228]

## Triển khai lên Amazon Bedrock AgentCore

[cite\_start]Sau khi bạn phát triển và test LangGraph framework cục bộ, bạn có thể triển khai nó lên Amazon Bedrock AgentCore Runtime. [cite: 230] [cite\_start]Amazon Bedrock AgentCore xử lý phần lớn việc điều phối container, quản lý phiên (session), khả năng mở rộng và trừu tượng hóa quản lý hạ tầng. [cite: 231] [cite\_start]Nó cung cấp môi trường thực thi bền vững có thể giữ trạng thái agent qua nhiều lần gọi. [cite: 232]

[cite\_start]Trước khi triển khai agent phân tích cổ phiếu lên AgentCore Runtime, chúng ta cần tạo AWS Identity and Access Management role IAM với quyền phù hợp. [cite: 233] [cite\_start]Role này cho phép AgentCore gọi endpoint SageMaker của bạn để inference mô hình GPT-OSS, quản lý kho container ECR, ghi log Amazon CloudWatch để giám sát và debugging, truy cập dịch vụ workload của AgentCore cho runtime, và gửi telemetry data tới AWS X-Ray và CloudWatch để quan sát. [cite: 234]

[cite\_start]Mã như sau: [cite: 235]

```python
# 
from create_agentcore_role import create_bedrock_agentcore_role
role_arn = create_bedrock_agentcore_role(
    role_name="MyStockAnalyzerRole",
    sagemaker_endpoint_name="your-endpoint-name",
    region="us-west-2"
)
```

[cite\_start]Sau khi tạo role, bạn dùng Amazon Bedrock AgentCore Starter Toolkit để triển khai agent. [cite: 237] [cite\_start]Toolkit đơn giản hóa quy trình triển khai bằng cách đóng gói mã, tạo image container, và cấu hình môi trường runtime: [cite: 238]

```python
# 
from bedrock_agentcore_starter_toolkit import Runtime
agentcore_runtime = Runtime() 
# Configure the agent 
response = agentcore_runtime.configure(
    entrypoint="langgraph_stock_sagemaker_gpt_oss.py",
    execution_role=role_arn,
    auto_create_ecr=True,
    requirements_file="requirements.txt",
    region="us-west-2",
    agent_name="stock_analyzer_agent"
) 
# Deploy to the cloud 
launch_result = agentcore_runtime.launch(local=False, local_build=False)
```

[cite\_start]Khi bạn dùng `BedrockAgentCoreApp`, nó tự động tạo HTTP server lắng nghe tại cổng 8080, triển khai endpoint `/invocations` để xử lý yêu cầu agent, endpoint `/ping` để kiểm tra sức khỏe (rất quan trọng cho agent bất đồng bộ), xử lý đúng content types và định dạng phản hồi, và quản lý error theo tiêu chuẩn AWS. [cite: 240]

[cite\_start]Sau khi bạn deploy lên AgentCore Runtime, trạng thái sẽ hiển thị `Ready` trên console Amazon Bedrock AgentCore. [cite: 241]

## Gọi agent

[cite\_start]Sau khi bạn tạo agent, bạn phải thiết lập điểm entry để gọi agent. [cite: 243] [cite\_start]Với Amazon AgentCore Runtime, chúng ta dùng decorator `@app.entrypoint` cho phần gọi agent và sử dụng đó làm điểm entry runtime. [cite: 244]

[cite\_start]Sau khi deploy agent lên AgentCore Runtime, bạn gọi nó bằng AWS SDK: [cite: 245]

```python
# 
import boto3
import json
agentcore_client = boto3.client('bedrock-agentcore', region_name='us-west-2')
response = agentcore_client.invoke_agent_runtime(
    agentRuntimeArn=launch_result.agent_arn,
    qualifier="DEFAULT",
    payload=json.dumps({
        "prompt": "Analyze SIM_STOCK for investment purposes"
    })
)
```

[cite\_start]Sau khi gọi agent phân tích cổ phiếu qua AgentCore Runtime, bạn phải parse và format phản hồi để hiển thị rõ ràng. [cite: 247] [cite\_start]Xử lý phản hồi gồm các bước: [cite: 248]

  * [cite\_start]Giải mã byte stream từ AgentCore thành văn bản đọc được. [cite: 249]
  * [cite\_start]Parse JSON phản hồi chứa phân tích cổ phiếu hoàn chỉnh. [cite: 250]
  * [cite\_start]Trích ba phần chính bằng regex pattern matching: [cite: 251]
      * [cite\_start]**Stock Data Gathering Section**: Trích các thông tin cốt lõi như mã cổ phiếu, thông tin công ty, giá hiện tại, các chỉ số thị trường, các tỉ số tài chính, dữ liệu giao dịch, và tiêu đề tin tức gần đây. [cite: 252]
      * [cite\_start]**Performance Analysis section**: Phân tích chỉ số kỹ thuật, chỉ số cơ bản, và độ biến động để tạo ra phân tích toàn diện về cổ phiếu. [cite: 253]
      * [cite\_start]**Stock Report Generation Section**: Tạo báo cáo PDF chi tiết với tất cả các phân tích kỹ thuật cổ phiếu. [cite: 254]
  * [cite\_start]Hệ thống cũng bao gồm xử lý lỗi graceful nếu có lỗi parse JSON, fallback sang hiển thị văn bản nếu parse cấu trúc thất bại, và cung cấp thông tin debugging để xử lý lỗi parsing phân tích cổ phiếu. [cite: 255]

<!-- end list -->

```python
# 
stock_analysis = parse_bedrock_agentcore_stock_response(invoke_response)
```

[cite\_start]Đầu ra đã format giúp dễ dàng xem quy trình ra quyết định của agent và trình bày kết quả phân tích cổ phiếu chuyên nghiệp cho các bên liên quan, hoàn thiện workflow từ triển khai mô hình đến kết quả kinh doanh có ý nghĩa: [cite: 257]

```text
# 
STOCK DATA GATHERING REPORT:
================================
Stock Symbol: SIM_STOCK
Company Name: Simulated Stock Inc.
Sector: SIM_SECTOR
Industry: SIM INDUSTRY
CURRENT MARKET DATA:
- Current Price: $29.31
- Market Cap: $3,958
- 52-Week High: $29.18
- 52-Week Low: $16.80
- YTD Return: 1.30%
- Volatility (Annualized): 32.22%
FINANCIAL METRICS:
- P/E Ratio: 44.80
- Forward P/E: 47.59
- Price-to-Book: 11.75
- Dividend Yield: 0.46%
- Revenue (TTM): $4,988
- Profit Margin: 24.30% 

STOCK PERFORMANCE ANALYSIS:
===============================
Stock: SIM_STOCK | Current Price: $29.31
TECHNICAL ANALYSIS:
- Price Trend: SLIGHT UPTREND
- YTD Performance: 1.03%
- Technical Score: 3/5
FUNDAMENTAL ANALYSIS:
- P/E Ratio: 34.80
- Profit Margin: 24.30%
- Dividend Yield: 0.46%
- Beta: 1.165
- Fundamental Score: 3/5

STOCK REPORT GENERATION:
===============================
Stock: SIM_STOCK Sector: SIM_INDUSTRY
Current Price: $29.78
REPORT SUMMARY:
- Technical Analysis: 8.33% YTD performance
- Report Type: Comprehensive stock analysis for informational purposes
- Generated: 2025-09-04 23:11:55
PDF report uploaded to S3: s3://amzn-s3-demo-bucket/2025/09/04/SIM_STOCK_Stock_Report_20250904_231155.pdf
REPORT CONTENTS:
• Executive Summary with key metrics
• Detailed market data and financial metrics
• Technical and fundamental analysis
• Professional formatting for documentation
```

## Dọn dẹp

[cite\_start]Bạn có thể xóa endpoint SageMaker để tránh phát sinh chi phí sau khi test bằng cách chạy các ô sau trong cùng notebook: [cite: 262]

```python
# 
sess.delete_inference_component(inference_component_name)
sess.delete_endpoint(endpoint_name)
sess.delete_endpoint_config(endpoint_name)
sess.delete_model(model_name)
```

[cite\_start]Bạn cũng có thể xóa các tài nguyên của Amazon Bedrock AgentCore bằng các lệnh sau: [cite: 264]

```python
# 
runtime_delete_response = agentcore_control_client.delete_agent_runtime(agentRuntimeId=launch_result.agent_id)
response = ecr_client.delete_repository(repositoryName=launch_result.ecr_uri.split('/')[1],force=True)
```

## Kết luận

[cite\_start]Trong bài viết này, chúng tôi đã xây dựng giải pháp end-to-end để triển khai các mô hình OpenAI open-weight trên một GPU G6e(L40s) duy nhất, tạo hệ thống phân tích cổ phiếu đa agent với LangGraph và triển khai liền mạch với Amazon Bedrock AgentCore. [cite: 267] [cite\_start]Triển khai này cho thấy tổ chức giờ đây có thể sử dụng các LLM mã nguồn mở mạnh mẽ với hiệu quả chi phí thông qua các framework serving như vLLM. [cite: 268]

[cite\_start]Bên cạnh phần triển khai kỹ thuật, việc nâng cấp workflow này có thể đem lại giá trị kinh doanh đáng kể, chẳng hạn giảm thời gian xử lý phân tích cổ phiếu, tăng năng suất của nhà phân tích bằng cách tự động hóa các đánh giá cổ phiếu định kỳ. [cite: 269] [cite\_start]Hơn nữa, bằng cách giải phóng nhà phân tích khỏi các tác vụ lặp đi lặp lại, tổ chức có thể chuyển các chuyên gia có kỹ năng sang xử lý các trường hợp phức tạp và xây dựng mối quan hệ — những hoạt động có thể thúc đẩy tăng trưởng kinh doanh. [cite: 270] [cite\_start]Chúng tôi mời bạn thử các code sample của chúng tôi và lặp lại các workflow agentic để đáp ứng các trường hợp sử dụng của bạn. [cite: 271]

-----

## Về các tác giả

[cite\_start]**Vivek Gangasani** là Worldwide Lead GenAI Specialist Solutions Architect cho SageMaker Inference. [cite: 273] [cite\_start]Anh phụ trách chiến lược Go-to-Market (GTM) và Outbound Product strategy cho SageMaker Inference. [cite: 274] [cite\_start]Vivek cũng hỗ trợ các doanh nghiệp và startup trong việc triển khai, quản lý và mở rộng mô hình GenAI của họ bằng SageMaker và GPU. [cite: 275] [cite\_start]Hiện tại, anh tập trung vào việc phát triển chiến lược và giải pháp tối ưu hiệu năng inference cũng như hiệu suất GPU cho việc lưu trữ các Large Language Models (LLMs). [cite: 276] [cite\_start]Trong thời gian rảnh, Vivek thích leo núi, xem phim, và thử các món ăn khác nhau. [cite: 277]

[cite\_start]**Surya Kari** là Senior Generative AI Data Scientist tại AWS, chuyên phát triển các giải pháp tận dụng foundation models tiên tiến. [cite: 278] [cite\_start]Anh có nhiều kinh nghiệm làm việc với các advanced language models như DeepSeek-R1, Llama family, và Qwen, tập trung vào fine-tuning và tối ưu hóa chúng cho các ứng dụng khoa học chuyên biệt. [cite: 279] [cite\_start]Kinh nghiệm của anh bao gồm việc triển khai pipeline huấn luyện hiệu quả và chiến lược triển khai mô hình bằng AWS SageMaker, giúp mở rộng foundation models từ giai đoạn phát triển đến sản xuất. [cite: 280] [cite\_start]Surya hợp tác với khách hàng để thiết kế và triển khai giải pháp Generative AI, giúp họ định hướng trong việc lựa chọn mô hình, phương pháp fine-tuning, và chiến lược triển khai nhằm đạt được hiệu năng tối ưu cho từng trường hợp cụ thể. [cite: 281]

```