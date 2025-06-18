# Web Scraper with File Download - Setup Instructions

## Overview
This enhanced web scraper automatically detects whether a URL points to a webpage or downloadable file:
- **Webpages**: Scrapes HTML content using crawl4ai
- **Files** (PDFs, documents, images, etc.): Downloads and saves to AWS S3

## Prerequisites

### Required Python Packages
```bash
pip install crawl4ai aiohttp boto3
```

### AWS Setup
1. **Create an S3 bucket** for storing downloaded files
2. **Set up AWS credentials** using one of these methods:
   - AWS CLI: `aws configure`
   - Environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
   - IAM roles (if running on EC2/Lambda)
   - AWS credentials file

### Required AWS Permissions
Your AWS user/role needs these S3 permissions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```

## Configuration

### Update S3 Settings
In the Python file, modify these variables:
```python
S3_BUCKET_NAME = "your-actual-bucket-name"  # Replace with your bucket
S3_REGION = "us-east-1"                     # Replace with your region
```

### Supported File Types
The scraper automatically detects these file types for download:
- **Documents**: .pdf, .doc, .docx, .xls, .xlsx, .ppt, .pptx
- **Archives**: .zip, .rar, .7z, .tar, .gz, .bz2
- **Images**: .jpg, .jpeg, .png, .gif, .bmp, .tiff
- **Videos**: .mp4, .avi, .mov, .wmv, .flv, .webm
- **Audio**: .mp3, .wav, .flac, .aac, .ogg
- **Data**: .txt, .csv, .json, .xml, .sql

## Usage

### Basic Usage
```python
import asyncio
from your_scraper_file import process_url

async def main():
    # This will automatically detect and handle the URL appropriately
    result = await process_url("https://example.com/document.pdf")
    
    if result['success']:
        if result['file_type'] == 'download':
            print(f"File saved to S3: {result['s3_url']}")
        elif result['file_type'] == 'html':
            print(f"Scraped HTML: {len(result['html'])} characters")
    else:
        print(f"Error: {result['error']}")

asyncio.run(main())
```

### Return Values

#### For Downloaded Files:
```python
{
    'success': True,
    'file_type': 'download',
    'original_url': 'https://example.com/file.pdf',
    's3_key': 'downloads/file.pdf',
    's3_url': 'https://bucket.s3.region.amazonaws.com/downloads/file.pdf',
    'file_size': 1234567,
    'content_type': 'application/pdf'
}
```

#### For Scraped Webpages:
```python
{
    'success': True,
    'file_type': 'html',
    'original_url': 'https://example.com',
    'final_url': 'https://example.com',
    'status_code': 200,
    'html': '<html>...</html>',
    'html_length': 12345
}
```

#### For Errors:
```python
{
    'success': False,
    'file_type': 'download' or 'html',
    'original_url': 'https://example.com',
    'error': 'Error message here'
}
```

### Advanced Usage

#### Custom S3 Path
```python
# Download with custom S3 key
result = await download_file_to_s3(
    "https://example.com/report.pdf",
    s3_key="reports/2024/quarterly-report.pdf"
)
```

#### Using Original Functions
The original scraping functions still work independently:
```python
# Original workflow still available
crawler, result = await navigate_to_page("https://example.com")
await scroll_to_bottom(crawler, result.url)
html = await get_outer_html(crawler, result.url)
await crawler.__aexit__(None, None, None)
```

## Troubleshooting

### Common Issues

1. **AWS Credentials Error**
   ```
   botocore.exceptions.NoCredentialsError: Unable to locate credentials
   ```
   **Solution**: Set up AWS credentials using `aws configure` or environment variables

2. **S3 Permission Denied**
   ```
   botocore.exceptions.ClientError: Access Denied
   ```
   **Solution**: Ensure your AWS user/role has S3 PutObject permissions

3. **Bucket Not Found**
   ```
   botocore.exceptions.ClientError: The specified bucket does not exist
   ```
   **Solution**: Create the S3 bucket or update `S3_BUCKET_NAME` in the code

4. **crawl4ai Installation Issues**
   ```bash
   # If you get playwright browser errors:
   playwright install
   ```

### Testing Your Setup

Run this test to verify everything works:
```python
import asyncio
from your_scraper_file import process_url

async def test_setup():
    # Test webpage scraping
    result = await process_url("https://httpbin.org/html")
    print("Webpage test:", "✓" if result['success'] else "✗")
    
    # Test file download (if you have a test PDF URL)
    # result = await process_url("https://example.com/test.pdf")
    # print("Download test:", "✓" if result['success'] else "✗")

asyncio.run(test_setup())
```

## Environment Variables (Optional)

For production, consider using environment variables:
```bash
export S3_BUCKET_NAME="your-bucket-name"
export S3_REGION="us-east-1"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
```

Then update the code to use them:
```python
import os
S3_BUCKET_NAME = os.getenv('S3_BUCKET_NAME', 'default-bucket-name')
S3_REGION = os.getenv('S3_REGION', 'us-east-1')
```
