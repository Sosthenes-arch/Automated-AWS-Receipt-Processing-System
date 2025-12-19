# Automated AWS Receipt Processing System

This project implements a serverless workflow to automatically process receipts uploaded to an Amazon S3 bucket. It extracts key information such as vendor, date, total amount, and line items using Amazon Textract, stores the structured data in Amazon DynamoDB, and sends an email summary via Amazon SES.

## Architecture

![Architecture Diagram](Automated%20AWS%20Receipt%20Processing%20System.drawio.svg)

### Data Flow
1.  **Upload**: A receipt image (JPG, PNG, PDF) is uploaded to an **Amazon S3** bucket.
2.  **Trigger**: The upload event triggers an **AWS Lambda** function (`Reciptproccessor.py`).
3.  **Analyze**: The Lambda function sends the document to **Amazon Textract** (`AnalyzeExpense` API) to extract structured data.
4.  **Store**: Extracted details (Receipt ID, Vendor, Date, Total, Items) are stored in an **Amazon DynamoDB** table.
5.  **Notify**: An email notification containing the receipt summary is sent to a specified recipient using **Amazon SES**.

## Features

-   **Serverless Architecture**: Fully managed and scalable using AWS Lambda.
-   **Intelligent Extraction**: Uses machine learning (Amazon Textract) to understand receipt layouts and extract specific fields without templates.
-   **Structured Storage**: Stores parsed receipt data in DynamoDB for easy querying and analytics.
-   **Instant Notifications**: Sends email alerts immediately after processing.
-   **Error Handling**: Basic error logging and exception handling features.

## Prerequisites

To deploy and run this system, you need:

-   **AWS Account**: Access to the AWS Console.
-   **AWS CLI**: Installed and configured with appropriate credentials.
-   **Python 3.x**: For local development or examining the code.
-   **Boto3**: AWS SDK for Python.

## Setup & Configuration

### 1. Resources
Ensure the following AWS resources are created:
-   **S3 Bucket**: To store the receipt images.
-   **DynamoDB Table**: Create a table (default name: `Receipts`) with a Primary Key (e.g., `receipt_id`).
-   **SES Identity**: Verify the sender and recipient email addresses in Amazon SES console.

### 2. Lambda Function
Create a Lambda function and deploy the code in `Reciptproccessor.py`.
-   **Runtime**: Python 3.x
-   **Timeout**: Increase timeout (e.g., 30s-60s) as Textract can take time.
-   **Permissions**: The Lambda execution role must have permissions for:
    -   `s3:GetObject` on the source bucket.
    -   `textract:AnalyzeExpense`.
    -   `dynamodb:PutItem` on the target table.
    -   `ses:SendEmail`.

### 3. Environment Variables
Configure the following environment variables in your Lambda function:

| Variable | Description | Default |
| :--- | :--- | :--- |
| `DYNAMODB_TABLE` | Name of the DynamoDB table | `Receipts` |
| `SES_SENDER_EMAIL` | Verified SES email address to send from | `your-email@example.com` |
| `SES_RECIPIENT_EMAIL` | Email address to receive notifications | `recipient@example.com` |

### 4. Event Trigger
Configure an S3 Event Notification on your bucket to trigger the Lambda function on `PUT` object events.

## Usage

1.  **Upload a Receipt**:
    Upload a receipt image file to your configured S3 bucket.
    ```bash
    aws s3 cp my-receipt.jpg s3://your-bucket-name/
    ```

2.  **Processing**:
    The system automatically picks up the file, processes it, and saves the data.

3.  **Check Results**:
    -   **DynamoDB**: Check your table for a new item.
    -   **Email**: Check your inbox for a summary of the receipt.

## Troubleshooting

-   **Logs**: Check CloudWatch Logs for the Lambda function if the email is not received.
-   **Textract Limits**: Ensure the image format and size are supported by Amazon Textract.
-   **Permissions**: Verify IAM roles if you see "Access Denied" errors in logs.

## Future Improvements

Potential enhancements for the system include:

1.  **Web Interface**: Create a simple S3-hosted HTML/JS page for uploading receipts directly to the bucket.
2.  **Expense Categorization**: Add logic to classify expenses (e.g., "Food", "Travel", "Office Supplies") and store them in DynamoDB.
3.  **Monthly Reporting**: Implement a Lambda function to aggregate expenses and send a monthly summary via email.
4.  **Image Optimization**: Compress receipt images before storage to save on S3 costs.
5.  **Error Notifications**: key failure metrics to SNS for real-time error alerts.
6.  **Status Tracking**: Add status fields (Processed, Pending, Rejected) to DynamoDB for better receipt lifecycle management.
7.  **Authentication**: Secure the upload process using Amazon Cognito to control access.