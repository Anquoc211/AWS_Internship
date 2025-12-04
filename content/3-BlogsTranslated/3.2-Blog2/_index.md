---
title: "Blog 2"
date: "2025-09-09T15:44:00+07:00"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Building Agentic Workflows with OpenAI GPT OSS on Amazon SageMaker AI and Amazon Bedrock AgentCore

**by:** Vivek Gangasani and Surya Kari | **Published:** Sep 17, 2025 | **in:** Amazon Bedrock, Amazon SageMaker AI, Amazon SageMaker Unified Studio, Artificial Intelligence, Customer Solutions

---

OpenAI has released two open-weight models, `gpt-oss-120b` (117 billion parameters) and `gpt-oss-20b` (21 billion parameters), both built on Mixture of Experts (MoE) architecture and using a 128K context window. These models lead open-source models according to the Artificial Analysis benchmark, and excel in reasoning capabilities and agentic workflows.

With Amazon SageMaker AI, you can fine-tune or customize models and deploy using your chosen framework through a fully managed service. Amazon SageMaker Inference gives you flexibility in bringing your own inference code and framework without needing to build and maintain server clusters yourself.

While large language models (LLMs) excel at understanding language and generating content, building practical agentic applications requires managing complex workflows, tool calling capabilities, and context management. Multi-agent architecture addresses these challenges by breaking down complex systems into specialized components, but this also introduces new complexities in agent coordination, memory management, and workflow orchestration.

In this post, we'll demonstrate how to deploy the `gpt-oss-20b` model to SageMaker managed endpoints and illustrate a practical example of a stock analysis agent assistant using LangGraph — a powerful graph-based framework that handles state management, workflow coordination, and persistent memory systems. We'll then deploy the agents to Amazon Bedrock AgentCore, a unified orchestration layer that abstracts infrastructure, allowing you to deploy and operate AI agents at scale securely.

## Solution Overview

In this solution, we build an agentic stock analyzer with the following main components:
* GPT OSS 20B model deployed to SageMaker endpoint using vLLM, an open-source serving framework for LLMs
* LangGraph to build the multi-agent orchestration framework
* Amazon Bedrock AgentCore to deploy the agents

The diagram below illustrates the solution architecture:

![Solution Architecture](https://i.imgur.com/placeholder-blog2-img1.png)

This architecture illustrates a multi-agent workflow hosted on Amazon Bedrock AgentCore Runtime running on AWS. Users submit queries, processed by a pipeline of specialized agents — Data Gathering Agent, Stock Performance Analyzer Agent, and Stock Report Generation Agent — each responsible for a distinct part of the stock assessment process. These agents coordinate within Amazon Bedrock AgentCore Runtime, and when they need to understand or generate language, they call a GPT OSS model hosted on SageMaker AI. The model processes inputs and returns structured results that help determine agent behavior, enabling a fully serverless, modular, and scalable agentic system using open-source models.

## Prerequisites

* Ensure you have the necessary quota for G6e instances to deploy the model.
* Request quota here if you don't have it yet.
* If this is your first time working with Amazon SageMaker Studio, you need to create a SageMaker domain first.
* Ensure your IAM role has the necessary permissions to deploy SageMaker Models and Endpoints.
* For more information, see How Amazon SageMaker AI works with IAM in the SageMaker Developer Guide.

## Deploying GPT-OSS Model to SageMaker Inference

Customers wanting to customize models and frameworks can deploy in a serverful manner, but this requires GPU access, serving frameworks, load balancers, and infrastructure configuration. SageMaker AI provides a fully managed hosting platform, handling infrastructure provisioning, necessary drivers, model loading, and deployment.

OpenAI's GPT-OSS models launch with 4-bit quantization scheme (MXFP4), enabling fast inference while keeping resources low. These models can run on `P5(H100)`, `P6(H200)`, `P4(A100)` and `G6e(L40)` instances. GPT-OSS models are sparse MoE architecture with 128 experts (120B) or 32 experts (20B), where each token is routed to 4 experts without sharing experts. Using MXFP4 for MoE weights shrinks model size to 63 GB (120B) and 14 GB (20B), making them runnable on a single H100 GPU.

For efficient deployment, you need a strong serving framework like vLLM. To deploy the model, we build a vLLM container with the latest version supporting GPT OSS models on SageMaker AI. You can use the Docker file and following script to build the container and push to Amazon Elastic Container Registry (Amazon ECR). The recommended approach is to execute directly from SageMaker Studio, a managed JupyterLab environment with AWS CLI access, where you can build and push the image to ECR as part of the SageMaker workflow. Alternatively, you can perform the same steps on an Amazon Elastic Compute Cloud (Amazon EC2) instance with Docker installed.

After you've built and pushed the container to Amazon ECR, you can open SageMaker Studio via the SageMaker AI console, as illustrated in the screenshot below:

![Open SageMaker Studio](https://i.imgur.com/placeholder-blog2-img2.png)

You can then create a Jupyter space or use an existing space to open JupyterLab and run the notebook.

![Create JupyterLab space](https://i.imgur.com/placeholder-blog2-img3.png)

Clone the following notebook and run "Option 3: Deploying from HF using BYOC." Update necessary parameters, such as inference image in the notebook with the container image. We also provide the necessary environment variables, as follows:

```python
# 
inference_image = f"{account_id}.dkr.ecr.{region}.amazonaws.com/vllm:v0.10.0-gpt-oss"
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
```

After you set up the deployment configuration, you can deploy to SageMaker AI with the following code:

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

You can run an inference example:

```python
# 
payload={
    "messages": [
        {"role": "user", "content": "Name popular places to visit in London?"}
    ],
}
res = llm.predict(payload)
print("-----\n" + res["choices"][0]["message"]["content"] + "\n-----\n")
print(res["usage"])

# Output:
# -----
# Here are some of the must-see spots in London -- a mix of iconic landmarks, world-class museums, and vibrant neighborhoods:
# | # | Place | Why It's Popular |
# |---|-------|------------------|
# | 1 | **Buckingham Palace** | The Queen's official London residence - watch the Changing of the Guard. |
# | 2 | **The Tower of London & Tower Bridge** | Historic castle, Crown Jewels, and the iconic bridge with glass floors. |
# | 3 | **The British Museum** | World-famous collection from the Rosetta Stone to Egyptian mummies (free entry). |
# ... (Rest of table) ...
# |15 | **Oxford Street & Regent Street** | Prime shopping streets for fashion, flagship stores, and historic architecture. |
# These spots cover history, culture, shopping, and leisure--perfect for a first visit or a weekend escape in London!
# -----
```

## Using LangGraph to Build Stock Analysis Agent

For our multi-agent stock analysis system, we use LangGraph to orchestrate the workflow. The Jupyter notebook for the code is in this Github repository. The system consists of three specialized tools working together for comprehensive stock analysis:

  * **gather_stock_data tool** collects comprehensive stock data for a given stock ticker, including current price, historical performance, financial metrics and market data. It returns formatted information including price history, company fundamentals, trading metrics, and recent news headlines.
  * **analyze_stock_performance tool** performs detailed technical and fundamental analysis of stock data, calculating metrics like price trends, volatility, and overall investment score. It evaluates multiple factors including P/E ratio, profit margins, and dividend yield to provide comprehensive performance analysis.
  * **generate_stock_report tool** creates professional PDF reports from collected and analyzed stock data, automatically uploading to Amazon S3 with date-organized folders.

To test locally, you can use a simplified version of the system by importing necessary functions from your local script. For example:

```python
# 
from langgraph_stock_local import langgraph_stock_sagemaker 
# Test the agent locally 
result = langgraph_stock_sagemaker({
    "prompt": "Analyze SIM_STOCK Stock for Investment purposes."
})
print(result)
```

This approach helps you iterate agent logic quickly before deploying to a scalable platform, ensuring each component works correctly and the overall workflow produces expected results for various stock types.

## Deploying to Amazon Bedrock AgentCore

After you develop and test the LangGraph framework locally, you can deploy it to Amazon Bedrock AgentCore Runtime. Amazon Bedrock AgentCore handles most container orchestration, session management, scalability and infrastructure management abstraction. It provides a persistent execution environment that can maintain agent state across multiple invocations.

Before deploying the stock analysis agent to AgentCore Runtime, we need to create an AWS Identity and Access Management (IAM) role with appropriate permissions. This role allows AgentCore to call your SageMaker endpoint for GPT-OSS model inference, manage ECR container repository, write Amazon CloudWatch logs for monitoring and debugging, access AgentCore's workload service for runtime, and send telemetry data to AWS X-Ray and CloudWatch for observability.

The code is as follows:

```python
# 
from create_agentcore_role import create_bedrock_agentcore_role
role_arn = create_bedrock_agentcore_role(
    role_name="MyStockAnalyzerRole",
    sagemaker_endpoint_name="your-endpoint-name",
    region="us-west-2"
)
```

After creating the role, you use the Amazon Bedrock AgentCore Starter Toolkit to deploy the agent. The toolkit simplifies the deployment process by packaging code, creating container images, and configuring the runtime environment:

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

When you use `BedrockAgentCoreApp`, it automatically creates an HTTP server listening on port 8080, implements the `/invocations` endpoint to handle agent requests, the `/ping` endpoint for health checks (crucial for asynchronous agents), handles content types and response formatting correctly, and manages errors according to AWS standards.

After you deploy to AgentCore Runtime, the status will show `Ready` on the Amazon Bedrock AgentCore console.

## Invoking the Agent

After you create the agent, you must set up the entry point to invoke the agent. With Amazon AgentCore Runtime, we use the `@app.entrypoint` decorator for the agent invocation part and use that as the runtime entry point.

After deploying the agent to AgentCore Runtime, you invoke it using the AWS SDK:

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

After invoking the stock analysis agent via AgentCore Runtime, you must parse and format the response for clear display. Response handling includes these steps:

  * Decode the byte stream from AgentCore into readable text.
  * Parse the JSON response containing complete stock analysis.
  * Extract three main sections using regex pattern matching:
      * **Stock Data Gathering Section**: Extracts core information like stock ticker, company info, current price, market metrics, financial ratios, trading data, and recent news headlines.
      * **Performance Analysis section**: Analyzes technical indicators, fundamental indicators, and volatility to create comprehensive stock analysis.
      * **Stock Report Generation Section**: Creates detailed PDF report with all stock technical analysis.
  * The system also includes graceful error handling for JSON parsing errors, fallback to text display if structured parsing fails, and provides debugging information for handling stock analysis parsing errors.

```python
# 
stock_analysis = parse_bedrock_agentcore_stock_response(invoke_response)
```

The formatted output makes it easy to view the agent's decision-making process and present professional stock analysis results to stakeholders, completing the workflow from model deployment to meaningful business outcomes:

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

## Cleanup

You can delete the SageMaker endpoint to avoid incurring charges after testing by running the following cells in the same notebook:

```python
# 
sess.delete_inference_component(inference_component_name)
sess.delete_endpoint(endpoint_name)
sess.delete_endpoint_config(endpoint_name)
sess.delete_model(model_name)
```

You can also delete Amazon Bedrock AgentCore resources with the following commands:

```python
# 
runtime_delete_response = agentcore_control_client.delete_agent_runtime(agentRuntimeId=launch_result.agent_id)
response = ecr_client.delete_repository(repositoryName=launch_result.ecr_uri.split('/')[1],force=True)
```

## Conclusion

In this post, we built an end-to-end solution to deploy OpenAI open-weight models on a single G6e(L40s) GPU, created a multi-agent stock analysis system with LangGraph and deployed seamlessly with Amazon Bedrock AgentCore. This implementation shows organizations can now use powerful open-source LLMs cost-effectively through serving frameworks like vLLM.

Beyond technical implementation, upgrading this workflow can deliver significant business value, such as reducing stock analysis processing time, increasing analyst productivity by automating regular stock assessments. Furthermore, by freeing analysts from repetitive tasks, organizations can shift skilled experts toward handling complex cases and building relationships — activities that can drive business growth. We invite you to try our code samples and iterate on agentic workflows to meet your use cases.

---

## About the Authors

**Vivek Gangasani** is Worldwide Lead GenAI Specialist Solutions Architect for SageMaker Inference. He is responsible for Go-to-Market (GTM) strategy and Outbound Product strategy for SageMaker Inference. Vivek also supports enterprises and startups in deploying, managing and scaling their GenAI models using SageMaker and GPUs. Currently, he focuses on developing strategy and solutions for optimizing inference performance and GPU efficiency for hosting Large Language Models (LLMs). In his free time, Vivek enjoys rock climbing, watching movies, and trying different cuisines.

**Surya Kari** is a Senior Generative AI Data Scientist at AWS, specializing in developing solutions leveraging advanced foundation models. He has extensive experience working with advanced language models like DeepSeek-R1, Llama family, and Qwen, focusing on fine-tuning and optimizing them for specialized scientific applications. His experience includes implementing efficient training pipelines and model deployment strategies using AWS SageMaker, helping scale foundation models from development to production. Surya collaborates with customers to design and implement Generative AI solutions, helping them navigate model selection, fine-tuning methods, and deployment strategies to achieve optimal performance for specific use cases.
