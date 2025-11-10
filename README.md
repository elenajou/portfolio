# 🚀 CI/CD Pipeline: Static Portfolio (GitHub to AWS S3)

This guide provides steps to set up an automated Continuous Integration/Continuous Delivery (CI/CD) pipeline using **AWS CodePipeline** to deploy a static website portfolio directly from a **GitHub repository** to an **Amazon S3 bucket** configured for static hosting.

Since a static portfolio (HTML, CSS, JS) requires no compilation, the standard **CodeBuild (CI) step is skipped**

## 1. 🎯 Phase 1: AWS S3 Setup (The Target Environment)
This phase prepares your destination bucket for public web hosting.

### 1.1. Create and Configure the S3 Bucket
1.  In the AWS Management Console, create a new **S3 bucket** with a **globally unique name** (ideally matching your domain name).
2.  **Crucial Security Step:** In the bucket settings, navigate to **Permissions** and **Disable "Block all public access"** (and acknowledge the setting). This is necessary for the website to be publicly viewable.

### 1.2. Enable Static Website Hosting
1.  Go to the **Properties** tab of your bucket.
2.  Find **Static website hosting**, click **Edit**, and select **Enable**.
3.  Set the **Index document** (e.g., `index.html`) and, optionally, the Error document.
4.  **Note:** The **Endpoint URL** provided here is the website's public address.

### 1.3. Apply a Public Read Bucket Policy
You must grant public read access to the objects in your bucket.
1.  Go to the **Permissions** tab and edit the **Bucket policy**.
2.  Use the policy below, **replacing `your-bucket-name`** with your actual bucket name:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```
## 2. 🏗️ Phase 2: CodePipeline Setup (The Automation)
This phase creates the pipeline that connects your GitHub repository to the S3 bucket.

### 2.1. Create the CodePipeline
1. In the AWS CodePipeline console, choose Create pipeline.
2. Give your pipeline a name and allow AWS to create a new service role for you.

### 2.2. Configure Source Stage (GitHub)
1. Source provider: Select GitHub (Version 2).
2. Set up a CodeStar Connection to link your AWS account to your GitHub account (this manages authentication securely).
3. Select your Repository and the Branch to monitor (e.g., main). A push to this branch will trigger the pipeline.

### 2.3. Skip the Build Stage
1. When prompted for the "Add build stage," choose Skip build stage and confirm you want to skip it.
2. This bypasses CodeBuild, as there is no compilation necessary for a static site.

### 2.4. Configure Deploy Stage (S3)
1. Deploy provider: Select Amazon S3.
2. Region: Select the AWS region where you created your S3 bucket.
3. Bucket: Select the S3 bucket configured in Phase 1.
4. Crucial Step: Check the box for "Extract file before deploy". This is essential because CodePipeline receives the code as a compressed artifact, and checking this box ensures it extracts the HTML, CSS, and JS files directly into your S3 bucket.

## 🎉 Deployment is Complete!
Your CI/CD pipeline is now active. Every time you push changes to the specified branch (e.g., main) on GitHub, CodePipeline will automatically fetch the code and deploy the updated files to your S3 static website hosting endpoint.