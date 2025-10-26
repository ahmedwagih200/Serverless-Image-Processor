🏗️ Serverless Image Processor – Architecture Documentation
📘 Overview
<img width="1042" height="561" alt="Diagram" src="https://github.com/user-attachments/assets/085ebe47-2999-44ff-b5df-4b160c91276c" />

The Serverless Image Processor is a lightweight, scalable solution that allows users to upload images via a web interface or API endpoint.
Once an image is uploaded, a Lambda function automatically resizes it and stores the processed image in another S3 bucket.
This architecture eliminates the need for managing servers, leveraging AWS’s fully managed services.

🧩 Core Components
1. Amazon S3 (Simple Storage Service)

Buckets:

Source Bucket – Stores the original uploaded images.

Destination Bucket – Stores the resized images.

Event Trigger:
The Source Bucket is configured to trigger an AWS Lambda function upon every ObjectCreated event (i.e., when a new image is uploaded).

2. AWS Lambda

Function Role:
Performs the core image processing logic (e.g., resizing, compressing, converting formats).

Flow:

Triggered by an S3 event.

Downloads the image from the source bucket.

Uses the Pillow (Python Imaging Library) layer to resize the image.

Uploads the processed image to the destination bucket.

Optionally, returns a pre-signed URL for accessing the processed image.

Layers:

Pillow library (for image manipulation).

IAM Permissions Required:

s3:GetObject and s3:PutObject for both buckets.

CloudWatch Logs permissions (logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents).

3. Amazon API Gateway

Purpose:
Provides an HTTP interface for users or web apps to upload images and retrieve processed URLs.

Endpoints:

POST /upload – Accepts image uploads and stores them in the source S3 bucket.

GET /image/{filename} – Optionally retrieves the resized image URL or metadata.

Integration:

The API Gateway invokes Lambda functions using Lambda Proxy Integration.

Requests and responses are handled via JSON payloads.

⚙️ Workflow Diagram (Text Representation)
[User / Web App]
       |
       v
[API Gateway] ---> [Lambda (Upload Handler)] ---> [Source S3 Bucket]
                                                  |
                                                  v (Event Trigger)
                                            [Lambda (Image Processor)]
                                                  |
                                                  v
                                            [Destination S3 Bucket]
                                                  |
                                                  v
                                      [Returns processed image URL]

🚀 Processing Flow in Detail

User Upload:

A user uploads an image via the frontend (static S3 website) or through an API call to API Gateway.

The image is stored in the source S3 bucket.

Triggering Lambda:

S3 sends an ObjectCreated event to the Lambda Image Processor function.

Processing:

The Lambda downloads the image to /tmp/ (temporary storage).

Resizes it using Pillow.

Uploads the processed image to the destination S3 bucket.

Result Delivery:

Optionally, the Lambda returns a pre-signed URL for the resized image via API Gateway.

The frontend displays the processed image.

🛡️ Security & Permissions
Component	Permission	Description
Lambda	s3:GetObject, s3:PutObject	Access to source & destination buckets
API Gateway	lambda:InvokeFunction	To invoke Lambda from API
S3	Event Notification	To trigger Lambda on upload
CloudWatch	Logging permissions	For Lambda monitoring and debugging
📈 Scalability & Cost

Scalability: Fully serverless; scales automatically with uploads.

Cost: Pay only for Lambda execution time, S3 storage, and minimal API Gateway requests.

Performance: Each Lambda function runs independently, allowing parallel image processing.

🧰 Example AWS Resources (Names)
Resource	Example Name
Source S3 Bucket	image-upload-source-bucket
Destination S3 Bucket	image-resized-output-bucket
Lambda Function	image-resize-processor
API Gateway	image-processor-api
🧪 Example Lambda Code (Python)
import boto3
from PIL import Image
import os
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    source_key = event['Records'][0]['s3']['object']['key']
    destination_bucket = 'image-resized-output-bucket'
    size = (800, 800)

    # Download image
    tmp_path = f"/tmp/{os.path.basename(source_key)}"
    s3.download_file(source_bucket, source_key, tmp_path)

    # Resize
    with Image.open(tmp_path) as img:
        img.thumbnail(size)
        buffer = io.BytesIO()
        img.save(buffer, 'JPEG')
        buffer.seek(0)

    # Upload resized image
    resized_key = f"resized-{os.path.basename(source_key)}"
    s3.upload_fileobj(buffer, destination_bucket, resized_key, ExtraArgs={'ContentType': 'image/jpeg'})

    return {
        'statusCode': 200,
        'body': f"Image processed successfully: s3://{destination_bucket}/{resized_key}"
    }<img width="1042" height="561" alt="Diagram" src="https://github.com/user-attachments/assets/9a86f466-3ab8-40e4-8e1e-883e160c7176" />


🧭 Deployment Steps Summary

Create Source and Destination S3 buckets.

Deploy Lambda function (with Pillow layer).

Grant S3 trigger permission to the Lambda.

Create API Gateway endpoints for upload and retrieval.

Deploy a static web app (HTML + JS) to allow user uploads.

Test the flow end-to-end.
