# Hosting-an-static-website

This guide will help you host a secure, scalable static website on AWS using S3, ACM, CloudFront, and Route 53. Follow these steps to get your website up and running with a custom domain and SSL/TLS encryption.

---

## Prerequisites

- An AWS account
- A registered domain (e.g., `abc.com`)
- Basic understanding of AWS services

## Steps

### 1. Set Up S3 Bucket for Static Website Hosting

1. **Create an S3 Bucket**:
   - Go to the [S3 Console](https://s3.console.aws.amazon.com/s3/home).
   - Click on **Create bucket**.
   - Name your bucket (e.g., `abc.com`) and choose the same region as your other AWS services.
   - check **Block all public access** (we will configure access through CloudFront).
   - Click **Create bucket**.

2. **Enable Static Website Hosting**:
   - Open your S3 bucket.
   - Go to the **Properties** tab.
   - Scroll down to **Static website hosting** and click **Edit**.
   - Choose **Enable**.
   - Set the **Index document** to `index.html` and **Error document** to `error.html`.
   - Save changes.

3. **Upload Website Files**:
   - Go to the **Objects** tab of your bucket.
   - Click **Upload** and select your website files (e.g., `index.html`, `error.html`, CSS, JavaScript, etc.).

### 2. Request SSL/TLS Certificate with ACM

1. **Go to the ACM Console**:
   - Navigate to the [ACM Console](https://console.aws.amazon.com/acm/home).

2. **Request a Public Certificate**:
   - Click on **Request a certificate**.
   - Choose **Request a public certificate**.
   - Enter your domain names:
     - `abc.com`
     - `*.abc.com`
     - `www.abc.com`
   - Click **Next**.

3. **Validate the Domain**:
   - Choose **DNS validation** and click **Next**.
   - Click **Review** and then **Confirm and request**.
   - In the **Validation** section, click on **Create record in Route 53**.
   - ACM will automatically add the necessary DNS records to your Route 53 hosted zone.

4. **Wait for Validation**:
   - The certificate will be issued once the DNS records propagate. This may take a few minutes.

### 3. Set Up CloudFront Distribution

1. **Create a CloudFront Distribution**:
   - Go to the [CloudFront Console](https://console.aws.amazon.com/cloudfront/home).
   - Click **Create Distribution**.

2. **Configure Origin Settings**:
   - In the **Origin** settings:
     - **Origin Domain**: Select your S3 bucket (e.g., `abc.com.s3.amazonaws.com`).
     - **Origin Path**: Leave this blank.
     - **Origin Access**: Choose **Origin Access Control (OAC)**.
     - **Create an OAC** if you haven’t already, and attach it to the origin.
     - **S3 Bucket Access**: Set to "Yes, update the bucket policy" to allow CloudFront to access your S3 bucket.

3. **Configure Default Cache Behavior**:
   - **Viewer Protocol Policy**: Choose **Redirect HTTP to HTTPS**.
   - **Allowed HTTP Methods**: Select **GET, HEAD**.

4. **Configure Distribution Settings**:
   - **Alternate Domain Names (CNAMEs)**: Add `abc.com`, `*.abc.com`, and `www.abc.com`.
   - **SSL Certificate**: Choose **Custom SSL Certificate** and select the certificate you created with ACM.
   - **Default Root Object**: Set to `index.html`.

### 4. Configure Bucket Policy for Origin Access Control (OAC)
   - To restrict access to your S3 bucket only to CloudFront, you need to configure a bucket policy.
   - Go to the **Permissions** tab of your S3 bucket and click **Edit** under **Bucket policy**.
   - Add the following policy, replacing `YOUR-OAC-ARN` with your Origin Access Control ARN and `abc.com` with your bucket name:

   ```json
   {
     "Version": "2008-10-17",
     "Id": "PolicyForCloudFrontPrivateContent",
     "Statement": [
       {
         "Sid": "AllowCloudFrontServicePrincipal",
         "Effect": "Allow",
         "Principal": {
           "Service": "cloudfront.amazonaws.com"
         },
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::abc.com/*",
         "Condition": {
           "StringEquals": {
             "AWS:SourceArn": "arn:aws:cloudfront::YOUR-OAC-ARN:distribution/*"
           }
         }
       }
     ]
   }
   ```

6. **Custom Error Pages**:
   - Go to the **Error Pages** tab.
   - Add a custom error response for 403 errors:
     - **HTTP Error Code**: 403
     - **Customize Error Response**: Yes
     - **Response Page Path**: `/error.html`
     - **Response Code**: 200

6. **Create the Distribution**:
   - Click **Create Distribution**.
   - Wait for the distribution status to change to **Deployed**.

### 5. Configure Route 53 DNS

1. **Go to Route 53 Console**:
   - Navigate to the [Route 53 Console](https://console.aws.amazon.com/route53/home).

2. **Update DNS Records**:
   - Go to your hosted zone for `abc.com`.
   - Add a new record set:
     - **Type**: `A` or `AAAA`
     - **Alias**: Yes
     - **Alias Target**: Select your CloudFront distribution from the dropdown.
     - **Name**: `abc.com`

   - Add a wildcard CNAME record set:
     - **Type**: `CNAME`
     - **Name**: `*.abc.com`
     - **Value**: Your CloudFront distribution domain name (e.g., `dxxxxxx.cloudfront.net`).

### 6. Test Your Setup

1. **Access Your Website**:
   - Visit `https://abc.com` and `https://www.abc.com` to verify that your website is accessible over HTTPS.

---

### Conclusion

Your static website is now securely hosted on AWS using S3, CloudFront, ACM, and Route 53. This setup ensures your website is fast, secure, and scalable, with global content delivery and SSL/TLS encryption.

---

### Troubleshooting

If you encounter issues, consider the following:
- Check the S3 bucket policy to ensure CloudFront has access.
- Verify that DNS records in Route 53 are correctly configured.
- Make sure that the SSL certificate is properly issued and associated with your CloudFront distribution.

---

This README provides a detailed, step-by-step process for setting up a static website on AWS, covering all necessary AWS services, including the bucket policy configuration for OAC.
