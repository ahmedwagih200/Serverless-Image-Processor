🏗️ Serverless Image Processor – Architecture Documentation
📘 Overview
<img width="1042" height="561" alt="Diagram" src="https://github.com/user-attachments/assets/085ebe47-2999-44ff-b5df-4b160c91276c" />

# 🏗️ Serverless Image Processor – Architecture Documentation

## 📘 Overview
The **Serverless Image Processor** is a lightweight, scalable solution that allows users to upload images via a web interface or API endpoint.  
Once an image is uploaded, a **Lambda function** automatically resizes it and stores the processed image in another **S3 bucket**.  
This architecture is fully **serverless**, eliminating the need to manage any servers.

---

## 🧩 Core Components

### 1. Amazon S3
- **Buckets**
  - **Source Bucket** – Stores the original uploaded images.
  - **Destination Bucket** – Stores the resized images.
- **Event Trigger**
  - The Source Bucket triggers an AWS Lambda function upon every `ObjectCreated` event (when a new image is uploaded).

---

### 2. AWS Lambda
- **Function Role**
  - Performs the image processing logic (resize, compress, format conversion).
- **Flow**
  1. Triggered by an S3 event.
  2. Downloads the uploaded image from the Source Bucket.
  3. Uses the **Pillow** library to resize the image.
  4. Uploads the processed image to the Destination Bucket.
  5. Returns a pre-signed URL (optional).

- **Layers**
  - Pillow library (Python Imaging Library).

- **Required IAM Permissions**
  - `s3:GetObject` and `s3:PutObject` for both buckets.
  - `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` for CloudWatch logging.

---

### 3. Amazon API Gateway
- **Purpose**
  - Provides an HTTP interface for image upload and retrieval.
- **Endpoints**
  - `POST /upload` → Upload image to S3.
  - `GET /image/{filename}` → Retrieve processed image URL.
- **Integration**
  - Uses **Lambda Proxy Integration** for request handling.

---

## ⚙️ Architecture Workflow


---

## 🚀 Processing Flow

1. User uploads an image via frontend or API Gateway.  
2. The image is stored in the **Source S3 bucket**.  
3. S3 triggers the **Image Processor Lambda**.  
4. Lambda resizes the image and uploads it to the **Destination Bucket**.  
5. The API or frontend fetches the processed image link.  

---

## 🛡️ Security & Permissions

| Component | Permission | Description |
|------------|-------------|-------------|
| Lambda | `s3:GetObject`, `s3:PutObject` | Access to source & destination buckets |
| API Gateway | `lambda:InvokeFunction` | To invoke Lambda functions |
| S3 | Event Notification | Trigger Lambda on image upload |
| CloudWatch | Logging permissions | For Lambda debugging and monitoring |

---

## 📈 Scalability & Cost
- **Scalability:** Fully serverless and scales automatically with workload.  
- **Cost:** Pay only for Lambda execution time, S3 storage, and API Gateway requests.  
- **Performance:** Each Lambda runs independently, allowing parallel processing.  

---

## 🧰 Example AWS Resources

| Resource | Example Name |
|-----------|---------------|
| Source Bucket | `image-upload-source-bucket` |
| Destination Bucket | `image-resized-output-bucket` |
| Lambda Function | `image-resize-processor` |
| API Gateway | `image-processor-api` |

---

## 🧪 Example Lambda Code (Python)

```python
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
    }


🧭 Deployment Steps

Create Source and Destination S3 buckets.

Create and deploy Lambda function (with Pillow layer).

Configure S3 trigger for ObjectCreated → Lambda.

Create API Gateway endpoints (/upload, /image/{filename}).

Deploy a static web app (HTML + JS) for user uploads.

Test end-to-end flow.
