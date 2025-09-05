# AWS Translate Automation with S3, Lambda, and IaC

[![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)](https://www.python.org/)

A serverless, event-driven architecture on AWS that automatically translates text files using Amazon Translate. This solution demonstrates Infrastructure as Code (IaC) principles using Terraform to create a scalable, cost-effective translation pipeline.

## 📋 Table of Contents

- [Architecture Overview](#-architecture-overview)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Installation & Deployment](#-installation--deployment)
- [Usage](#-usage)
- [Testing](#-testing)
- [Project Structure](#-project-structure)
- [Cost Optimization](#-cost-optimization)
- [Monitoring & Logging](#-monitoring--logging)
- [Cleanup](#-cleanup)
- [Troubleshooting](#-troubleshooting)
- [Future Enhancements](#-future-enhancements)

## 🏗️ Architecture Overview

The solution implements a fully serverless translation pipeline:

```
User Uploads File → S3 Input Bucket → S3 Event Notification → AWS Lambda → 
Amazon Translate → S3 Output Bucket → (Optional) SNS Notification
```

### Core AWS Services

| Service | Purpose |
|---------|---------|
| **Amazon S3** | Two buckets for input (source files) and output (translated files) |
| **AWS Lambda** | Python function that orchestrates the translation process |
| **Amazon Translate** | Neural machine translation service |
| **Amazon Comprehend** | Optional language detection for source text |
| **AWS IAM** | Secure roles and policies with least privilege access |
| **Amazon CloudWatch** | Logging, monitoring, and observability |
| **Terraform** | Infrastructure as Code management |

## ✨ Features

- **Fully Automated**: Translation triggers automatically on file upload
- **Multi-language Support**: Supports 50+ language pairs via Amazon Translate
- **Cost Effective**: Serverless architecture with pay-per-use pricing
- **Secure**: Encryption at rest and in transit, IAM least privilege principles
- **Scalable**: Automatically handles increased translation workloads
- **Infrastructure as Code**: Complete environment defined in Terraform

## 📋 Prerequisites

Before deployment, ensure you have:

1. **AWS Account** with appropriate permissions
2. **AWS CLI** installed and configured:
   ```bash
   aws configure
   ```
3. **Terraform** (v1.0+) installed
4. **Python 3.9+** (for local testing/development)

## 🚀 Installation & Deployment

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/aws-translate-automation.git
cd aws-translate-automation
```

### 2. Initialize Terraform

```bash
terraform init
```

### 3. Review Deployment Plan

```bash
terraform plan
```

### 4. Deploy Infrastructure

```bash
terraform apply -auto-approve
```

**Expected Output:**
```
Apply complete! Resources: 9 added, 0 changed, 0 destroyed.

Outputs:

input_bucket_name = "translate-request-bucket-12345"
output_bucket_name = "translate-response-bucket-12345"
lambda_function_name = "translate-processor"
```

## 📊 Usage

### File Format

Upload JSON files to the input bucket with the following structure:

```json
{
  "text": "Hello, world! I'm using AWS Translate.",
  "source_language": "en",
  "target_language": "de"
}
```

**Parameters:**
- `text`: The content to translate (required)
- `source_language`: Source language code or "auto" for detection (optional)
- `target_language`: Target language code (required, e.g., "es", "fr", "de")

### Uploading Files

**Using AWS CLI:**
```bash
aws s3 cp your-file.json s3://$(terraform output -raw input_bucket_name)/uploads/
```

**Using AWS Console:**
1. Navigate to S3 service
2. Find your input bucket (name from terraform output)
3. Upload JSON file to the `uploads/` folder

### Retrieving Results

Translated files will appear in the output bucket with the naming convention `translated-<original-filename>.json`:

```bash
# List files in output bucket
aws s3 ls s3://$(terraform output -raw output_bucket_name)/

# Download a specific translated file
aws s3 cp s3://$(terraform output -raw output_bucket_name)/translated-your-file.json .
```

## 🧪 Testing

### Sample Test File

A sample test file `sample-request.json` is provided:

```json
{
  "text": "Hello World, I'm Seli. This is a test of the translation service.",
  "source_language": "auto", 
  "target_language": "fr"
}
```

### Run Test

```bash
# Upload test file
aws s3 cp sample-request.json s3://$(terraform output -raw input_bucket_name)/uploads/

# Check for results (after 30-60 seconds)
aws s3 ls s3://$(terraform output -raw output_bucket_name)/

# Download and view translated result
aws s3 cp s3://$(terraform output -raw output_bucket_name)/translated-sample-request.json .
cat translated-sample-request.json
```

### Expected Result

```json
{
  "original_text": "Hello World, I'm Seli. This is a test of the translation service.",
  "translated_text": "Bonjour le monde, je suis Seli. C'est un test du service de traduction.",
  "source_language": "en",
  "target_language": "fr",
  "translation_timestamp": "2023-11-05T14:30:45.123Z"
}
```

## 📁 Project Structure

```
aws-translate-automation/
├── main.tf                 # Primary Terraform configuration
├── variables.tf            # Variable definitions
├── outputs.tf              # Output values
├── lambda/                 # Lambda function directory
│   ├── main.py            # Lambda function code
│   ├── requirements.txt   # Python dependencies
│   └── build.sh           # Lambda packaging script
├── samples/               # Sample files
│   ├── sample-request.json
│   └── test-suite.json
├── scripts/               # Utility scripts
│   ├── deploy.sh
│   └── test-translation.sh
└── README.md
```

## 💰 Cost Optimization

This architecture is designed for cost efficiency:

- **S3**: Costs only for storage and requests (5GB Free Tier)
- **Lambda**: Pay per execution (1M free requests/month)
- **Translate**: 2M characters free monthly for first 12 months
- **Lifecycle Policies**: Automatic deletion of old files after 7 days

**Estimated Cost:** <$5/month for moderate usage (within Free Tier limits)

## 📈 Monitoring & Logging

### CloudWatch Logs

View Lambda execution logs:
```bash
aws logs describe-log-groups --query 'logGroups[?contains(logGroupName, `translate-processor`)].logGroupName' --output text | xargs -I {} aws logs filter-log-events --log-group-name {} --limit 10
```

### CloudWatch Metrics

Monitor key metrics:
- Lambda invocations and errors
- Translate character count
- S3 bucket operations

### Adding Custom Monitoring

Uncomment the CloudWatch alarm section in `main.tf` to enable:
- Error rate monitoring
- Translation volume alerts
- S3 bucket size alerts

## 🧹 Cleanup

To avoid ongoing charges, destroy all resources:

```bash
terraform destroy
```

You will be prompted to confirm deletion of all AWS resources.

## 🐛 Troubleshooting

### Common Issues

1. **Lambda not triggering**
   - Check S3 bucket event configuration
   - Verify Lambda permissions policy

2. **Translation failures**
   - Check CloudWatch logs for error details
   - Verify language codes are supported by Amazon Translate

3. **Access denied errors**
   - Verify IAM role has necessary permissions
   - Check bucket policies

### Debugging Steps

1. Check Lambda execution status:
   ```bash
   aws lambda get-function --function-name $(terraform output -raw lambda_function_name)
   ```

2. View recent CloudWatch logs:
   ```bash
   aws logs describe-log-streams --log-group-name /aws/lambda/$(terraform output -raw lambda_function_name) --query 'logStreams[0].logStreamName' --output text | xargs -I {} aws logs get-log-events --log-group-name /aws/lambda/$(terraform output -raw lambda_function_name) --log-stream-name {}
   ```

## 🔮 Future Enhancements

Potential improvements to this architecture:

1. **API Gateway Frontend**: REST API for direct translation requests
2. **SNS Notifications**: Email alerts when translations complete
3. **Step Functions**: Workflow orchestration for complex translation pipelines
4. **DynamoDB**: Tracking translation requests and status
5. **CloudFront**: CDN distribution for translated content
6. **Multi-file Support**: Batch processing of multiple documents
7. **Format Support**: Expanded beyond JSON to text files, documents

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check [issues page](https://github.com/your-username/aws-translate-automation/issues).

## 🙏 Acknowledgments

- **Amazon Web Services** for comprehensive cloud services
- **Hashicorp** for Terraform IaC tooling
- **Azubi Africa** and **Generation Ghana** for training and support
- **AWS Documentation** for excellent reference materials

---

