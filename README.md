# AWS Translate Automation with S3, Lambda, and IaC
 
[![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)

[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

[![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)](https://www.python.org/)
 
A serverless, event-driven architecture on AWS that automatically translates text files using Amazon Translate. This solution demonstrates Infrastructure as Code (IaC) principles using Terraform to create a scalable, cost-effective translation pipeline.     

## Architecture
 
The architecture consists of the following AWS services:
 
*   **Amazon S3 (Simple Storage Service):** Two S3 buckets are used:
    *   **Input Bucket:** Stores the input text files. When a new file is uploaded to this bucket, it triggers the Lambda function.
    *   **Output Bucket:** Stores the translated text files.
*   **AWS Lambda:** A Lambda function written in Python is triggered by S3 object creation events in the input bucket. The function reads the content of the uploaded file, uses Amazon Translate to perform the translation, and then saves the result as a new file in the output S3 bucket.
*   **Amazon Translate:** The translation service used by the Lambda function to translate the text.
*   **Amazon Comprehend:** Used to detect the dominant language of the text.
*   **AWS IAM (Identity and Access Management):** An IAM role is created for the Lambda function to grant it the necessary permissions to access S3, CloudWatch Logs, and the Translate API.
*   **Amazon CloudWatch:** Used for logging the Lambda function's execution.
 
The entire infrastructure is managed as code using Terraform.
 
## Prerequisites
 
Before you begin, ensure you have the following installed and configured:
 
*   **AWS Account:** An active AWS account with the necessary permissions to create the resources defined in the Terraform configuration.
*   **AWS CLI:** The AWS Command Line Interface installed and configured with your credentials.
*   **Terraform:** Terraform installed on your local machine.
*   **Python:** Python 3.9 or later.
 
## Installation and Setup
 
1.  **Clone the repository:**
 
    ```bash
    git clone https://github.com/your-username/your-repository.git
    cd your-repository
    ```
 
2.  **Configure AWS Credentials:**
 
    Ensure your AWS credentials are configured correctly. You can do this by running `aws configure` or by setting the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.
 
## Deployment
 
1.  **Initialize Terraform:**
 
    Navigate to the root of the project directory and run the following command to initialize Terraform and download the required providers:
 
    ```bash
    terraform init
    ```
 
2.  **Review the execution plan:**
 
    Run the following command to see the resources that Terraform will create:
 
    ```bash
    terraform plan
    ```
 
3.  **Apply the Terraform configuration:**
 
    Run the following command to create the AWS resources:
 
    ```bash
    terraform apply
    ```
 
    You will be prompted to confirm the creation of the resources. Type `yes` to proceed.
 
## Usage
 
1.  **Upload a file to the input S3 bucket:**
 
    Once the infrastructure is deployed, you can trigger the Lambda function by uploading a JSON file to the input S3 bucket. The JSON file should have the following format:
 
    ```json
    {
      "text": "Hello, world!",
      "target_language": "de"
    }
    ```
 
    You can use the AWS Management Console or the AWS CLI to upload the file.
 
2.  **Check the output S3 bucket:**
 
    After the Lambda function has been executed, a new file will be created in the output S3 bucket. The file will be named `translated-<original-file-name>.json` and will contain the original text, the translated text, and the target language.
 
## Testing
 
A sample `test.json` file is provided in the root of the project. You can use this file to test the Lambda function.
 
1.  **Upload the test file:**
 
    ```bash
    aws s3 cp test.json s3://<your-input-bucket-name>/uploads/test.json
    ```
 
2.  **Check the output:**
 
    After a few seconds, you should see a new file in the output S3 bucket.
 
## Terraform File Descriptions
 
*   `main.tf`: The main Terraform file that defines the provider and the `archive_file` data source to zip the Lambda function code.
*   `variables.tf`: Defines the variables used in the Terraform configuration, such as the AWS region and the S3 bucket names.
*   `lambda.tf`: Defines the Lambda function, the Lambda permission to allow S3 to invoke it, and the S3 bucket notification to trigger the Lambda function.
*   `s3.tf`: Defines the input and output S3 buckets and their lifecycle configurations.
*   `iam.tf`: Defines the IAM role and policy for the Lambda function.
*   `outputs.tf`: Defines the outputs of the Terraform configuration, such as the S3 bucket names.
*   `terraform.tfstate`: Stores the state of the managed infrastructure.
 
## Cleanup
 
To destroy all the resources created by Terraform, run the following command:
 
```bash
terraform destroy
```
 
You will be prompted to confirm the deletion of the resources. Type `yes` to proceed.
 
