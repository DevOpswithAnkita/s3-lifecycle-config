# AWS S3 Lifecycle Configuration

This repository provides examples and templates for configuring AWS S3 lifecycle rules to automatically delete objects after a specified time period.

## Overview

AWS S3 Lifecycle configuration allows you to automatically manage objects in your S3 buckets by defining rules that transition or delete objects based on their age or other criteria. This is particularly useful for:

- Cleaning up temporary files
- Managing log files
- Removing draft or work-in-progress data
- Cost optimization by automatically deleting old data

## Quick Start

### Prerequisites

- AWS CLI installed and configured with appropriate permissions
- S3 bucket with objects you want to manage
- IAM permissions for `s3:PutLifecycleConfiguration` and `s3:GetLifecycleConfiguration`

### Basic Example

**Scenario:** Delete all objects under a specific prefix after 24 hours

**Configuration:**
- Bucket name: `your-bucket-name`
- Prefix: `your-prefix/`
- Expiration: 1 day

## Configuration Methods

### Method 1: Direct AWS CLI Command

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket your-bucket-name \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "DeleteObjectsAfter24Hours",
        "Prefix": "your-prefix/",
        "Status": "Enabled",
        "Expiration": {
          "Days": 1
        }
      }
    ]
  }'
```

### Method 2: Using JSON File

Create a `lifecycle.json` file:

```json
{
  "Rules": [
    {
      "ID": "DeleteObjectsAfter24Hours",
      "Prefix": "your-prefix/",
      "Status": "Enabled",
      "Expiration": {
        "Days": 1
      }
    }
  ]
}
```

Apply the configuration:

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket your-bucket-name \
  --lifecycle-configuration file://lifecycle.json
```
## Validate the JSON first:

  cat lifecycle.json | jq .
## Verify the policy was applied:

  aws s3api get-bucket-lifecycle-configuration --bucket bucket-name

### Check if there are actual files inside these folders:

aws s3 ls s3://bucketname/object/ --recursive --human-readable

### Method 3: AWS Management Console

1. Navigate to S3 in AWS Console
2. Select your bucket
3. Go to **Management** tab
4. Click **Create lifecycle rule**
5. Configure:
   - Rule name: `DeleteObjectsAfter24Hours`
   - Rule scope: **Limit the scope with filters**
   - Prefix: `your-prefix/`
   - Lifecycle rule actions: **Expire current versions of objects**
   - Days after object creation: `1`
6. Review and create

## Advanced Examples

### Multiple Prefixes with Different Retention Periods

```json
{
  "Rules": [
    {
      "ID": "DeleteTempFiles",
      "Prefix": "temp/",
      "Status": "Enabled",
      "Expiration": {
        "Days": 1
      }
    },
    {
      "ID": "DeleteLogs",
      "Prefix": "logs/",
      "Status": "Enabled",
      "Expiration": {
        "Days": 7
      }
    },
    {
      "ID": "DeleteDrafts",
      "Prefix": "drafts/",
      "Status": "Enabled",
      "Expiration": {
        "Days": 30
      }
    }
  ]
}
```

### With Versioning Support

```json
{
  "Rules": [
    {
      "ID": "DeleteObjectsWithVersioning",
      "Prefix": "your-prefix/",
      "Status": "Enabled",
      "Expiration": {
        "Days": 1
      },
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 1
      }
    }
  ]
}
```

### Using Tags for Selective Deletion

```json
{
  "Rules": [
    {
      "ID": "DeleteTaggedObjects",
      "Status": "Enabled",
      "Filter": {
        "And": {
          "Prefix": "your-prefix/",
          "Tags": [
            {
              "Key": "Environment",
              "Value": "temp"
            }
          ]
        }
      },
      "Expiration": {
        "Days": 1
      }
    }
  ]
}
```

## Verification Commands

### Check Current Lifecycle Configuration

```bash
aws s3api get-bucket-lifecycle-configuration --bucket your-bucket-name
```

### List Objects with Prefix

```bash
aws s3 ls s3://your-bucket-name/your-prefix/ --recursive
```

### Remove Lifecycle Configuration

```bash
aws s3api delete-bucket-lifecycle --bucket your-bucket-name
```

## Important Notes

###  **Data Loss Warning**
- Objects deleted by lifecycle rules are **permanently deleted**
- There is no recycle bin or recovery option
- Test your configuration thoroughly before applying to production data

### Timing Considerations
- AWS processes lifecycle rules **once per day**
- Deletion may occur 24-48 hours after object creation
- Exact timing is not guaranteed and depends on AWS processing schedules

### Cost Implications
- You stop being charged for storage once objects are deleted
- Early deletion of objects in Glacier storage classes may incur additional charges
- Monitor your usage and costs after implementing lifecycle rules

### Permissions Required

Your IAM user/role needs these permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutLifecycleConfiguration",
        "s3:GetLifecycleConfiguration",
        "s3:DeleteBucket"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name"
    }
  ]
}
```

## Real-World Use Cases

### Temporary File Cleanup
Perfect for applications that generate temporary files, caches, or processing artifacts.

### Log Management
Automatically clean up application logs, access logs, or debugging files.

### Development/Testing
Remove build artifacts, test data, or development files automatically.

### Data Processing Pipelines
Clean up intermediate files in ETL processes or data transformation workflows.

## Troubleshooting

### Common Issues

1. **Rule not taking effect**
   - Check that the rule status is "Enabled"
   - Verify the prefix matches your object keys exactly
   - Remember rules process once daily

2. **Permission denied errors**
   - Ensure your AWS credentials have the required S3 permissions
   - Check bucket policies don't block lifecycle operations

3. **Objects not being deleted**
   - Verify object keys match the specified prefix
   - Check if objects have legal holds or retention policies
   - Confirm the rule has been active for at least 24-48 hours

### Testing Your Configuration

Before applying to production:

1. Create a test bucket with sample objects
2. Apply the lifecycle rule
3. Monitor for 2-3 days to confirm behavior
4. Check AWS CloudTrail logs for lifecycle events

## Contributing

Feel free to submit issues, fork the repository, and create pull requests for any improvements.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Resources

- [AWS S3 Lifecycle Configuration Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [AWS CLI S3API Reference](https://docs.aws.amazon.com/cli/latest/reference/s3api/)
- [S3 Lifecycle Configuration Examples](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-configuration-examples.html)
