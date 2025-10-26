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

