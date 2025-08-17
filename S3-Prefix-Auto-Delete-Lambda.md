# S3 Prefix Auto-Delete Lambda

This AWS Lambda function automatically deletes objects under a given **S3 prefix** that are older than **24 hours (1 day)**.
The function is triggered by **Amazon EventBridge (CloudWatch Events)** on a scheduled basis (e.g., once per day).

---

## Features

- Deletes objects in an S3 bucket under a specified prefix.
- Skips folder-like keys (ending with `/`).
- Uses a **cutoff time of 24 hours**.
- Can be triggered daily/hourly via **EventBridge schedule rule**.

---

## Prerequisites

- AWS Account with **S3** and **Lambda** enabled.
- Python 3.9+ Lambda runtime.
- `boto3` (preinstalled in AWS Lambda environment).
- IAM Role with the following minimum permissions:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "s3:ListBucket",
          "s3:DeleteObject"
        ],
        "Resource": [
          "arn:aws:s3:::your-bucket-name",
          "arn:aws:s3:::your-bucket-name/your/prefix/path/*"
        ]
      }
    ]
  }

  ```
---
## Lambda Code

```py
import boto3
from datetime import datetime, timedelta

s3 = boto3.client('s3')

# Replace with your bucket and prefix
bucket_name = 'your-bucket-name'
prefix = 'your/prefix/path/'

def lambda_handler(event, context):
    cutoff_date = datetime.now() - timedelta(days=1)
    paginator = s3.get_paginator('list_objects_v2')
    deleted_count = 0

    for page in paginator.paginate(Bucket=bucket_name, Prefix=prefix):
        for obj in page.get('Contents', []):
            key = obj['Key']
            last_modified = obj['LastModified'].replace(tzinfo=None)

            if key.endswith('/'):
                continue

            if last_modified < cutoff_date:
                s3.delete_object(Bucket=bucket_name, Key=key)
                print(f"Deleted: {key}")
                deleted_count += 1

    return {
        'statusCode': 200,
        'body': f"Deleted {deleted_count} objects older than {cutoff_date}"
    }


```
---
## Setup Instructions

1. Create Lambda Function
   - Go to AWS Lambda Console.
   - Create a new function (Python 3.9+ runtime recommended).
   - Copy and paste the above code into the function editor.
2. Configure IAM Role
   - Attach the IAM policy shown above to your Lambda execution role.
3. Create EventBridge Rule (Scheduler)
   - Go to Amazon EventBridge → Rules.
   - Create a new rule with a schedule expression.
   - Set the target as your Lambda function.
   - Common Cron Expressions
   - Every day at midnight UTC → cron(0 0 * * ? *)
   - Every 6 hours → cron(0 */6 * * ? *)
   - Every Monday at 5 AM UTC → cron(0 5 ? * MON *)
---

## Limitations

Lambda may timeout if too many objects exist under the prefix.

For large datasets, consider using S3 Lifecycle Rules instead of Lambda.

---
